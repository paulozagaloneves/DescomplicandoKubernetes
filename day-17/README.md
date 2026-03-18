# Day 17 - Descomplicando Network Policy

## Índice

- [O que são Network Policy](#o-que-são-network-policy)
  - [Como funciona?](#como-funciona)
  - [Principais características](#principais-características)
  - [Casos de uso comuns](#casos-de-uso-comuns)
  - [⚠️ Importante](#-importante)
- [Meu primeiro Network Policy](#meu-primeiro-network-policy)
  - [Teste antes de aplicar o Network Policy](#teste-antes-de-aplicar-o-network-policy)
  - [Exemplo Prático](#exemplo-prático)
  - [Teste após aplicar o Network Policy](#teste-após-aplicar-o-network-policy)

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