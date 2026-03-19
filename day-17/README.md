# Day 17 - Descomplicando Network Policy

## Índice

- [Day 17 - Descomplicando Network Policy](#day-17---descomplicando-network-policy)
  - [Índice](#índice)
  - [O que são Network Policy](#o-que-são-network-policy)
    - [Como funciona?](#como-funciona)
    - [Principais características](#principais-características)
    - [Casos de uso comuns](#casos-de-uso-comuns)
    - [⚠️ Importante](#️-importante)
  - [Meu primeiro Network Policy](#meu-primeiro-network-policy)
    - [Teste antes de aplicar o Network Policy](#teste-antes-de-aplicar-o-network-policy)
    - [Exemplo Prático](#exemplo-prático)
    - [Teste após aplicar o Network Policy](#teste-após-aplicar-o-network-policy)
  - [Criando NetPol para bloquear Ingress de um Namespace](#criando-netpol-para-bloquear-ingress-de-um-namespace)
    - [Apagar NetworkPolicy](#apagar-networkpolicy)
    - [Criar NetworkPolicy para bloquear todo ingress no Giropops](#criar-networkpolicy-para-bloquear-todo-ingress-no-giropops)
    - [Validar policy](#validar-policy)
  - [Melhorando Network Policy e Operadores Lógicos](#melhorando-network-policy-e-operadores-lógicos)
  - [Criando regras NetPol para blindar a nossa App](#criando-regras-netpol-para-blindar-a-nossa-app)

## O que são Network Policy

**Network Policy** é um recurso nativo do Kubernetes que permite controlar o **acesso à rede entre pods** e para o exterior do cluster. É como um "firewall" no nível dos pods.

### Como funciona?

Por padrão, no Kubernetes:
- ✅ **Todos os pods podem se comunicar entre si** (sem restrições)
- ✅ **Qualquer pod pode alcançar qualquer serviço**

Uma **Network Policy** muda isso, permitindo criar **regras de tráfego explícitas**:
- Apenas pods A podem falar com pods B
- Apenas pods do namespace X podem acessar pods do namespace Y
- Tráfego de entrada (Ingress) ou saída (Egress)

### Principais características

| Aspecto | Descrição |
|---------|-----------|
| **Ingress** | Controla tráfego **de entrada** para os pods |
| **Egress** | Controla tráfego **de saída** dos pods |
| **podSelector** | Seleciona quais pods a policy aplica |
| **namespaceSelector** | Seleciona pods de outros namespaces |
| **Whitelist** | Network Policy funciona por "liberação" (padrão negar) |

### Casos de uso comuns

1. **Isolamento de namespaces** - Frontend não fala com Backend
2. **Segurança em multi-tenant** - Tenants isolados um do outro
3. **Conformidade** - Atender requisitos de segurança/compliance
4. **Microsserviços** - Apenas serviços permitidos se comunicam

### ⚠️ Importante

- Requer CNI plugin que suporte Network Policy (Calico, Flannel, etc.)
- Não funciona sem um plugin apropriado instalado
- Padrão é **deny-all** quando a policy é criada (nega o que não foi explicitamente permitido)


## Meu primeiro Network Policy

### Teste antes de aplicar o Network Policy

```bash
$ kubectl run alpine --image alpine:3 -it sh
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ # apk add redis
(1/3) Installing libgcc (15.2.0-r2)
(2/3) Installing libstdc++ (15.2.0-r2)
(3/3) Installing redis (8.4.2-r0)
  Executing redis-8.4.2-r0.pre-install
  Executing redis-8.4.2-r0.post-install
Executing busybox-1.37.0-r30.trigger
OK: 14.8 MiB in 19 packages
/ # redis-cli -h redis-service.giropops.svc
redis-service.giropops.svc:6379> ping
PONG
redis-service.giropops.svc:6379> exit
/ # redis-cli -h redis-service.giropops.svc.cluster.local ping
PONG
/ # exit
Session ended, resume using 'kubectl attach alpine -c alpine -i -t' command when the pod is running
$
```

### Exemplo Prático

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: limit-redis-giropops
  namespace: giropops
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector: {}  # Qualquer pod do namespace giropops
```

**Resultado:** Apenas pods do namespace `giropops` podem se conectar ao Redis

### Teste após aplicar o Network Policy

```bash
$ kubectl attach alpine -c alpine -i -t
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ # apk add redis
(1/3) Installing libgcc (15.2.0-r2)
(2/3) Installing libstdc++ (15.2.0-r2)
(3/3) Installing redis (8.4.2-r0)
  Executing redis-8.4.2-r0.pre-install
  Executing redis-8.4.2-r0.post-install
Executing busybox-1.37.0-r30.trigger
OK: 14.8 MiB in 19 packages
/ # redis-cli -h redis-service.giropops.svc ping
^C
/ # exit
Session ended, resume using 'kubectl attach alpine -c alpine -i -t' command when the pod is running
$
```

## Criando NetPol para bloquear Ingress de um Namespace

**Mostrar todos namespaces e suas labels**

```bash
$ kubectl get ns --show-labels         
NAME                STATUS   AGE   LABELS
cert-manager        Active   54d   kubernetes.io/metadata.name=cert-manager
cilium-secrets      Active   74d   app.kubernetes.io/managed-by=Helm,app.kubernetes.io/part-of=cilium,kubernetes.io/metadata.name=cilium-secrets
cilium-test-1       Active   74d   app.kubernetes.io/name=cilium-cli,kubernetes.io/metadata.name=cilium-test-1
cilium-test-ccnp1   Active   74d   kubernetes.io/metadata.name=cilium-test-ccnp1
cilium-test-ccnp2   Active   74d   kubernetes.io/metadata.name=cilium-test-ccnp2
default             Active   74d   kubernetes.io/metadata.name=default
development         Active   73d   kubernetes.io/metadata.name=development
giropops            Active   20d   app.kubernetes.io/owner=kode.expert,kubernetes.io/metadata.name=giropops
ingress-nginx       Active   55d   app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx,kubernetes.io/metadata.name=ingress-nginx
integration         Active   20d   app.kubernetes.io/owner=kode.expert,kubernetes.io/metadata.name=integration
kube-node-lease     Active   74d   kubernetes.io/metadata.name=kube-node-lease
kube-public         Active   74d   kubernetes.io/metadata.name=kube-public
kube-system         Active   74d   kubernetes.io/metadata.name=kube-system
kyverno             Active   20d   kubernetes.io/metadata.name=kyverno,name=kyverno
metallb-system      Active   73d   kubernetes.io/metadata.name=metallb-system,pod-security.kubernetes.io/audit=privileged,pod-security.kubernetes.io/enforce=privileged,pod-security.kubernetes.io/warn=privileged
monitoring          Active   49d   kubernetes.io/metadata.name=monitoring,pod-security.kubernetes.io/warn-version=latest,pod-security.kubernetes.io/warn=privileged
traefik             Active   73d   kubernetes.io/metadata.name=traefik
```


### Apagar NetworkPolicy

Apagar regras criadas anteriormente para evitar conflitos.

```bash
$ kubectl delete netpol -n giropops limit-redis-giropops 
networkpolicy.networking.k8s.io "limit-redis-giropops" deleted from giropops namespace
```

### Criar NetworkPolicy para bloquear todo ingress no Giropops

**Objetivo:** bloquear o acesso a qualquer pod dentro do Giropops de requisições vindas do exterior

**Filename:** netpol-block-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-ingress-from-namespace
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchExpressions:
              - key: kubernetes.io/metadata.name
                operator: In
                values:
                  - giropops
```


```bash
$ kubectl apply -f netpol-block-ingress.yaml 
networkpolicy.networking.k8s.io/block-ingress-from-namespace created
$
$ kubectl get netpol -n giropops                        
NAME                           POD-SELECTOR   AGE
block-ingress-from-namespace   <none>         4m40s
```

### Validar policy

**Bloqueo **de pedidos fora do namespace

```bash
$ kubectl exec -ti alpine -n default  -- sh  
/ # apk add curl
OK: 13.2 MiB in 26 packages
/ # curl giropops-service.giropops.svc:5000
^C
/ #
/ # apk add redis
(1/3) Installing libgcc (15.2.0-r2)
(2/3) Installing libstdc++ (15.2.0-r2)
(3/3) Installing redis (8.4.2-r0)
  Executing redis-8.4.2-r0.pre-install
  Executing redis-8.4.2-r0.post-install
Executing busybox-1.37.0-r30.trigger
OK: 20.0 MiB in 29 packages
/ # redis-cli -h redis-service.giropops.svc ping
^C
/ # 

```

**Permitir acesso** dentro do namespace

```bash
$ kubectl exec -ti alpine -n giropops -- sh
/ # redis-cli -h redis-service.giropops.svc ping
PONG
/ #
```

## Melhorando Network Policy e Operadores Lógicos

Dando acesso aos pods dentro do namespace **giropops** e no **ingress-nginx**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-ingress-from-namespace
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchExpressions:
              - key: kubernetes.io/metadata.name
                operator: In
                values:
                  - giropops
                  - ingress-nginx
```

## Criando regras NetPol para blindar a nossa App

Apagar **todos** Network Policy


```bash
$ kubectl get netpol -n giropops               
NAME                           POD-SELECTOR   AGE
block-ingress-from-namespace   <none>         57m
$ kubectl delete netpol -n giropops block-ingress-from-namespace 
networkpolicy.networking.k8s.io "block-ingress-from-namespace" deleted from giropops namespace
```

**Filename:** netpol-block-all-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Aplicar regra

```bash
$ kubectl apply -f netpol-block-all-ingress.yaml                
networkpolicy.networking.k8s.io/default-deny-all created

$ k get netpol -n giropops                      
NAME               POD-SELECTOR   AGE
default-deny-all   <none>         24s
$
```

Validar

```bash
$ curl -k https://giropops.cloud.local
<html>
<head><title>504 Gateway Time-out</title></head>
<body>
<center><h1>504 Gateway Time-out</h1></center>
<hr><center>nginx</center>
</body>
</html>
$
$
$ kubectl exec -ti alpine -n giropops -- sh     
/ # redis-cli -h redis-service.giropops.svc ping
Could not connect to Redis at redis-service.giropops.svc:6379: Try again
/ # curl giropops-service.giropops.svc:5000
curl: (6) Could not resolve host: giropops-service.giropops.svc (Timeout while contacting DNS servers)
/ # 
```




