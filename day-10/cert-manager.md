# cert-manager

## O que é cert-manager

## Instalando cert-manager

**Referência:** https://cert-manager.io/

```bash
$ helm install cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.20.2 \
  --set crds.enabled=true
```

## trust-manager

**Referência:** https://cert-manager.io/docs/trust/trust-manager/#quick-start-example

### O que é trust-manager

trust-manager is the easiest way to manage trust bundles in Kubernetes and OpenShift clusters.

It orchestrates bundles of trusted X.509 certificates which are primarily used for validating certificates during a TLS handshake but can be used in other situations, too.

**Overview**

trust-manager is a small Kubernetes operator which reduces the overhead of managing TLS trust bundles in your clusters, providing a much quicker way to update trust stores when they need to change.

It adds the Bundle custom Kubernetes resource (CRD) which can read input from various CA sources and combine the resultant certificates into a bundle ready to be used by your applications.

trust-manager ensures that it's both quick and easy to keep your trusted certificates up-to-date and enables cluster administrators to easily automate providing a secure bundle without having to worry about rebuilding containers to update trust stores.

It's designed to complement cert-manager and works well when consuming CA certificates used by a cert-manager Issuer or ClusterIssuer - but trust-manager can be used entirely independently of cert-manager, too.

### Instalando trust-manager

#### Instalando cert-manager

```bash
$ helm install cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.20.2 \
  --set crds.enabled=true
```

#### Instalando trust-manager

```bash
$ helm upgrade trust-manager oci://quay.io/jetstack/charts/trust-manager \
  --install \
  --namespace cert-manager \
  --wait
  Release "trust-manager" does not exist. Installing it now.
Pulled: quay.io/jetstack/charts/trust-manager:0.22.0
Digest: sha256:16bd36a86be474a42b8250cc00d715f09b0543a29dace58d932d2bf74e4b075a
NAME: trust-manager
LAST DEPLOYED: Tue Apr 14 16:39:55 2026
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
⚠  WARNING: Consider increasing the Helm value `replicaCount` to 2 if you require high availability.
⚠  WARNING: Consider setting the Helm value `podDisruptionBudget.enabled` to true if you require high availability.

trust-manager v0.22.0 has been deployed successfully!
Your installation includes a default CA package, using the following
default CA package image:

:

It's imperative that you keep the default CA package image up to date.
To find out more about securely running trust-manager and to get started
with creating your first bundle, check out the documentation on the
cert-manager website:

https://cert-manager.io/docs/projects/trust-manager/
```

#### Criar CA Bundle

Cria um `Secret` com o teu CA root no namespace `cert-manager`:

O nome **internal-ca-root** pode ser trocado por outro nome que melhor identifique o certificado (ex: domain-com-ca-root)

**Filename:** cert_internal_secrets.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: internal-ca-root
  namespace: cert-manager
type: Opaque
stringData:
  ca.crt: |
    -----BEGIN CERTIFICATE-----
    <ca-root PEM>
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    <ca-intermediate PEM>
    -----END CERTIFICATE-----
```

```bash
$ kubectl apply -f cert_internal_secrets.yaml
```

**Filename:** cert_internal_ca_bundle.yaml

```yaml
apiVersion: trust.cert-manager.io/v1alpha1
kind: Bundle
metadata:
  name: internal-ca-bundle
spec:
  sources:
    - secret:
        name: internal-ca-root
        key: ca.crt
    - useDefaultCAs: true          # inclui os CAs do sistema (opcional)
  target:
    configMap:
      key: ca-bundle.crt
    namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: <namespace-da-tua-app>
```

```bash
$ kubectl apply -f cert_internal_ca_bundle.yaml
```

#### Validar

**Filename:** cert-debug-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cert-debug
  namespace: <namespace-da-tua-app>
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cert-debug
  template:
    metadata:
      labels:
        app: cert-debug
    spec:
      containers:
        - name: cert-debug
          image: debian:bookworm-slim
          command: ["sleep", "infinity"]
          env:
            - name: SSL_CERT_FILE
              value: /etc/ssl/trust-manager/ca-bundle.crt
          volumeMounts:
            - name: ca-bundle
              mountPath: /etc/ssl/trust-manager
              readOnly: true
      volumes:
        - name: ca-bundle
          configMap:
            name: internal-ca-bundle
            items:
              - key: ca-bundle.crt
                path: ca-bundle.crt
```

```bash
$ kubectl apply -f cert-debug-deployment.yaml
```

**Validar no Pod**

```bash
# Entra no pod
kubectl exec -it deploy/cert-debug -n <namespace> -- bash

# Instala as ferramentas
apt-get update -qq && apt-get install -y -qq openssl curl ca-certificates
```

**Verificar se o bundle foi montado**

```bash
$ ls -lh /etc/ssl/trust-manager/ca-bundle.crt
# Contar quantos certs estão no bundle:
$ grep -c "BEGIN CERTIFICATE" /etc/ssl/trust-manager/ca-bundle.crt
# Verificar os subjects de todos certificados no bundle
$ openssl crl2pkcs7 -nocrl \
  -certfile /etc/ssl/trust-manager/ca-bundle.crt \
  | openssl pkcs7 -print_certs -noout \
  | grep -E "subject|issuer"
# Verificar se o teu CA root/intermediate estão presentes
$ openssl crl2pkcs7 -nocrl \
  -certfile /etc/ssl/trust-manager/ca-bundle.crt \
  | openssl pkcs7 -print_certs -noout \
  | grep -i "<nome-da-tua-org>"
```

Testar o handshake TLS directamente contra a API

```bash
# Usando o bundle como trust anchor:
openssl s_client \
  -connect <host-da-api>:<porto> \
  -CAfile /etc/ssl/trust-manager/ca-bundle.crt \
  -verify_return_error \
  -brief

# Resultado esperado: "Verification: OK"
```

Testar com curl

```bash
curl -v \
  --cacert /etc/ssl/trust-manager/ca-bundle.crt \
  https://<host-da-api>/health
```

 Verificar o chain completo (root → intermediate → certificado final)

 ```bash
 $ openssl s_client \
  -connect <host-da-api>:<porto> \
  -CAfile /etc/ssl/trust-manager/ca-bundle.crt \
  -showcerts 2>/dev/null \
  | openssl x509 -noout -text \
  | grep -A2 "Issuer\|Subject\|Validity"
```