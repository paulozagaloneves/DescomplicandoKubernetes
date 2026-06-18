# Tutorial: Kubernetes Secrets e ConfigMaps

Este tutorial explica como utilizar **Secrets** e **ConfigMaps** no Kubernetes, utilizando os arquivos de exemplo presentes neste diretório.

## O que são Secrets e ConfigMaps?

*   **ConfigMap**: Utilizado para armazenar dados de configuração não confidenciais em pares chave-valor. Permite desacoplar a configuração da imagem do container.
*   **Secret**: Semelhante ao ConfigMap, mas destinado a armazenar dados sensíveis (senhas, tokens, chaves SSH/TLS). Os dados são armazenados em base64 (embora isso não seja encriptação, apenas codificação).

---

## 1. Criando um Secret Simples

O arquivo `primeiro-secret.yaml` demonstra um Secret básico do tipo `Opaque` (o padrão).

**Arquivo:** `primeiro-secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  # Valores codificados em base64:
  # paulo.zagalo.neves -> cGF1bG8uemFnYWxvLm5ldmVzCg==
  # password123        -> cGFzc3dvcmQxMjM=
  username: cGF1bG8uemFnYWxvLm5ldmVzCg==
  password: cGFzc3dvcmQxMjM=
```

### Como aplicar:

```bash
kubectl apply -f primeiro-secret.yaml
```

### Como verificar:

```bash
kubectl get secret my-secret
kubectl get secret my-secret -o yaml
```

O Kubernetes decodifica os valores automaticamente quando montados como variáveis de ambiente ou volumes num Pod.

---

## 2. Usando Secrets para Docker Registry Privado

Para puxar imagens de um registro privado (como o Docker Hub privado), usamos um Secret do tipo `kubernetes.io/dockerconfigjson`.

**Arquivo:** `dockerhub-secrets.yaml`
Este arquivo contém as credenciais de autenticação do Docker.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dockerhub-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <BASE64_DO_CONFIG_JSON>
```

Podemos utilizar este secret num Deployment através do campo `imagePullSecrets`.

**Arquivo:** `minion-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minion-deployment
spec:
  # ...
  template:
    # ...
    spec:
      containers:
      - name: minion
        image: pauloneves/minion:latest  # Imagem privada
        # ...
      imagePullSecrets:     # Referência ao Secret
      - name: dockerhub-secret
```

### Como aplicar (apenas exemplo, requer credenciais reais):

```bash
kubectl apply -f dockerhub-secrets.yaml
kubectl apply -f minion-deployment.yaml
```

---

## 3. ConfigMaps e Secrets em Volumes (Exemplo com SSL)

Este exemplo mostra como configurar uma aplicação (Nginx) usando um **ConfigMap** para o arquivo de configuração `nginx.conf` e um **Secret** para os certificados SSL/TLS.

### Componentes:

1.  **ConfigMap (`nginx-configmap.yaml`)**: Contém o arquivo `nginx.conf`.
2.  **Secret (`tls-secret.yaml`)**: Contém o certificado (`tls.crt`) e a chave privada (`tls.key`).
3.  **Deployment (`my-app-with-ssl.yaml`)**: Monta ambos como volumes no container.

**Arquivo do ConfigMap:** `nginx-configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events { }
    http {
      server {
          listen 443 ssl;
          ssl_certificate     /etc/nginx/tls/tls.crt;
          ssl_certificate_key /etc/nginx/tls/tls.key;
          # ...
      }
    }
```

**Arquivo do Deployment:** `my-app-with-ssl.yaml`
Este arquivo conecta tudo. Observe a seção `volumes` e `volumeMounts`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ssl-deployment
spec:
  # ...
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 443
        volumeMounts:
        # Monta o arquivo nginx.conf do ConfigMap
        - name: nginx-config-volume
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        # Monta os certificados do Secret
        - name: tls-secret-volume
          mountPath: /etc/nginx/tls
      volumes:
      - name: nginx-config-volume
        configMap:
          name: nginx-config
      - name: tls-secret-volume
        secret:
          secretName: tls-secret
```

### Como aplicar:

```bash
# 1. Criar o Secret TLS e o ConfigMap
kubectl apply -f tls-secret.yaml
kubectl apply -f nginx-configmap.yaml

# 2. Criar o Deployment
kubectl apply -f my-app-with-ssl.yaml
```

Isso criará um servidor Nginx configurado via ConfigMap e servindo HTTPS usando certificados armazenados de forma segura num Secret.
