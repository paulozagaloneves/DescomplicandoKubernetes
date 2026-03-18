# Day 16 - Descomplicando Taints, Labels, Toleration e Affinity no K8s

## Taints

### O que são Taints?

Taints (manchas ou marcações) são propriedades aplicadas aos **nodes** do Kubernetes que permitem que um node repila (rejeite) um conjunto de Pods. Eles funcionam como uma forma de garantir que Pods não sejam agendados em nodes inapropriados.

Um Taint é composto por três elementos:
- **Key (chave)**: Identifica o tipo de restrição
- **Value (valor)**: Informação adicional sobre a restrição (opcional)
- **Effect (efeito)**: Define o que acontece com os Pods que não toleram o Taint

### Para que servem os Taints?

Os Taints são utilizados para diversos cenários, incluindo:

1. **Dedicação de Nodes**: Reservar nodes específicos para cargas de trabalho especiais (ex: nodes com GPU para workloads de machine learning)
2. **Isolamento de Workloads**: Separar ambientes diferentes ou equipes, garantindo que determinados Pods só executem em nodes específicos
3. **Manutenção de Nodes**: Marcar nodes que estão em manutenção ou com problemas para evitar que novos Pods sejam agendados
4. **Nodes com Hardware Especial**: Garantir que apenas Pods que necessitam de hardware específico sejam agendados em nodes com esse hardware
5. **Proteção de Control Plane**: Por padrão, os nodes master possuem Taints para evitar que Pods de aplicação sejam executados neles

### Efeitos dos Taints

Existem três tipos de efeitos que um Taint pode ter:

- **NoSchedule**: Novos Pods que não toleram o Taint não serão agendados no node. Pods já em execução não são afetados.
- **PreferNoSchedule**: O Kubernetes tentará evitar agendar Pods que não toleram o Taint, mas não é uma regra rígida. Se não houver outra opção, o Pod pode ser agendado mesmo assim.
- **NoExecute**: Pods que não toleram o Taint não serão agendados no node, e Pods já em execução que não toleram o Taint serão removidos (evicted) do node.

### Sintaxe de um Taint

```bash
kubectl taint nodes <node-name> <key>=<value>:<effect>
```

### Exemplos de Uso

**Exemplo 1: Adicionar um Taint NoSchedule**
```bash
kubectl taint nodes node01 ambiente=producao:NoSchedule
```
Este comando adiciona um Taint chamado "ambiente" com valor "producao" e efeito NoSchedule ao node01.

**Exemplo 2: Adicionar um Taint NoExecute**
```bash
kubectl taint nodes node02 manutencao=true:NoExecute
```
Este comando marca o node02 como em manutenção e remove todos os Pods que não toleram esse Taint.

**Exemplo 3: Adicionar um Taint PreferNoSchedule**
```bash
kubectl taint nodes node03 hardware=gpu:PreferNoSchedule
```
Este comando sugere que apenas Pods que necessitam GPU sejam agendados no node03.

**Exemplo 4: Remover um Taint**
```bash
kubectl taint nodes node01 ambiente=producao:NoSchedule-
```
O sinal de menos (-) no final remove o Taint do node.

**Exemplo 5: Listar Taints de um Node**
```bash
kubectl describe node <node-name> | grep Taints
```

### Importante

Para que um Pod possa ser agendado em um node com Taint, ele precisa ter uma **Toleration** que corresponda ao Taint. As Tolerations são definidas no manifest do Pod e serão abordadas em uma seção específica.

### Exemplo da aula

**Antes de aplicar o Taint**

```bash
$ kubectl get pods -o wide
NAME                                          READY   STATUS             RESTARTS          AGE   IP           NODE            NOMINATED NODE   READINESS GATES
giropops-senhas-deployment-69576c64cd-787dh   1/1     Running            9 (27d ago)       61d   10.0.2.229   dk8s-worker-3   <none>           <none>
giropops-senhas-deployment-69576c64cd-dzft2   1/1     Running            9 (27d ago)       61d   10.0.1.108   dk8s-worker-2   <none>           <none>
giropops-senhas-deployment-69576c64cd-h7fg8   1/1     Running            9 (27d ago)       61d   10.0.0.130   dk8s-worker-1   <none>           <none>
locust-giropops-76cc56c95b-j5fjx              1/1     Running            0                 9d    10.0.1.91    dk8s-worker-2   <none>           <none>
minion-deployment-68df78c4df-9rpxm            1/1     Running            9 (27d ago)       52d   10.0.1.124   dk8s-worker-2   <none>           <none>
nginx-hpa-58f69dfffc-k2597                    1/1     Running            0                 15d   10.0.1.224   dk8s-worker-2   <none>           <none>
nginx-hpa-58f69dfffc-sw42m                    1/1     Running            0                 15d   10.0.2.215   dk8s-worker-3   <none>           <none>
nginx-podmetrics                              2/2     Running            0                 27d   10.0.1.139   dk8s-worker-2   <none>           <none>
nginx-server-7d9cbf7d4f-7nkdn                 2/2     Running            2 (27d ago)       30d   10.0.1.120   dk8s-worker-2   <none>           <none>
nginx-server-7d9cbf7d4f-dssgw                 2/2     Running            2 (27d ago)       30d   10.0.0.200   dk8s-worker-1   <none>           <none>
nginx-server-7d9cbf7d4f-wppbw                 2/2     Running            2 (27d ago)       30d   10.0.2.183   dk8s-worker-3   <none>           <none>
nginx-ssl-deployment-688dcb8fd-d47dj          1/1     Running            8 (27d ago)       51d   10.0.0.49    dk8s-worker-1   <none>           <none>
nginx-statefulset-0                           1/1     Running            5 (27d ago)       51d   10.0.1.208   dk8s-worker-2   <none>           <none>
nginx-statefulset-1                           1/1     Running            5 (27d ago)       51d   10.0.0.121   dk8s-worker-1   <none>           <none>
nginx-statefulset-2                           1/1     Running            5 (27d ago)       51d   10.0.2.208   dk8s-worker-3   <none>           <none>
ops-deck-api-84ddd69678-2278d                 0/1     Running            11182 (9s ago)    30d   10.0.0.107   dk8s-worker-1   <none>           <none>
ops-deck-api-84ddd69678-4tmks                 0/1     CrashLoopBackOff   11287 (19s ago)   30d   10.0.2.90    dk8s-worker-3   <none>           <none>
redis-deployment-d74599fc4-bt7vf              1/1     Running            9 (27d ago)       61d   10.0.2.60    dk8s-worker-3   <none>           <none>
```

Aplicar o taint "desativando" o node **dk8s-worker-2**

```bash
$ kubectl taint node dk8s-worker-2 manutencao=true:NoExecute
node/dk8s-worker-2 tainted
```

**Obter taints de um Node**

```bash
$ kubectl describe node dk8s-worker-2 | grep Taints
Taints:             manutencao=true:NoExecute
```

**Validar pods**

```bash
$ kubectl get pods -o wide
NAME                                          READY   STATUS             RESTARTS            AGE   IP           NODE            NOMINATED NODE   READINESS GATES
giropops-senhas-deployment-69576c64cd-787dh   1/1     Running            9 (27d ago)         61d   10.0.2.229   dk8s-worker-3   <none>           <none>
giropops-senhas-deployment-69576c64cd-dzft2   1/1     Running            9 (27d ago)         61d   10.0.1.108   dk8s-worker-2   <none>           <none>
giropops-senhas-deployment-69576c64cd-h7fg8   1/1     Running            9 (27d ago)         61d   10.0.0.130   dk8s-worker-1   <none>           <none>
locust-giropops-76cc56c95b-j5fjx              1/1     Terminating        0                   9d    10.0.1.91    dk8s-worker-2   <none>           <none>
minion-deployment-68df78c4df-9rpxm            0/1     Error              9 (27d ago)         52d   10.0.1.124   dk8s-worker-2   <none>           <none>
nginx-hpa-58f69dfffc-k2597                    0/1     Completed          0                   15d   10.0.1.224   dk8s-worker-2   <none>           <none>
nginx-hpa-58f69dfffc-sw42m                    1/1     Running            0                   15d   10.0.2.215   dk8s-worker-3   <none>           <none>
nginx-podmetrics                              2/2     Terminating        0                   27d   10.0.1.139   dk8s-worker-2   <none>           <none>
nginx-server-7d9cbf7d4f-7nkdn                 0/2     Completed          2                   30d   10.0.1.120   dk8s-worker-2   <none>           <none>
nginx-server-7d9cbf7d4f-dssgw                 2/2     Running            2 (27d ago)         30d   10.0.0.200   dk8s-worker-1   <none>           <none>
nginx-server-7d9cbf7d4f-wppbw                 2/2     Running            2 (27d ago)         30d   10.0.2.183   dk8s-worker-3   <none>           <none>
nginx-ssl-deployment-688dcb8fd-d47dj          1/1     Running            8 (27d ago)         51d   10.0.0.49    dk8s-worker-1   <none>           <none>
nginx-statefulset-0                           0/1     Completed          5                   51d   10.0.1.208   dk8s-worker-2   <none>           <none>
nginx-statefulset-1                           1/1     Running            5 (27d ago)         51d   10.0.0.121   dk8s-worker-1   <none>           <none>
nginx-statefulset-2                           1/1     Running            5 (27d ago)         51d   10.0.2.208   dk8s-worker-3   <none>           <none>
ops-deck-api-84ddd69678-2278d                 0/1     CrashLoopBackOff   11182 (82s ago)     30d   10.0.0.107   dk8s-worker-1   <none>           <none>
ops-deck-api-84ddd69678-4tmks                 0/1     CrashLoopBackOff   11287 (2m32s ago)   30d   10.0.2.90    dk8s-worker-3   <none>           <none>
redis-deployment-d74599fc4-bt7vf              1/1     Running            9 (27d ago)         61d   10.0.2.60    dk8s-worker-3   <none>           <none>
```

```bash
$ kubectl get pods -o wide
NAME                                          READY   STATUS             RESTARTS            AGE   IP           NODE            NOMINATED NODE   READINESS GATES
giropops-senhas-deployment-69576c64cd-787dh   1/1     Running            9 (27d ago)         61d   10.0.2.229   dk8s-worker-3   <none>           <none>
giropops-senhas-deployment-69576c64cd-dzft2   1/1     Running            9 (27d ago)         61d   10.0.1.108   dk8s-worker-2   <none>           <none>
giropops-senhas-deployment-69576c64cd-h7fg8   1/1     Running            9 (27d ago)         61d   10.0.0.130   dk8s-worker-1   <none>           <none>
nginx-hpa-58f69dfffc-sw42m                    1/1     Running            0                   15d   10.0.2.215   dk8s-worker-3   <none>           <none>
nginx-server-7d9cbf7d4f-dssgw                 2/2     Running            2 (27d ago)         30d   10.0.0.200   dk8s-worker-1   <none>           <none>
nginx-server-7d9cbf7d4f-wppbw                 2/2     Running            2 (27d ago)         30d   10.0.2.183   dk8s-worker-3   <none>           <none>
nginx-ssl-deployment-688dcb8fd-d47dj          1/1     Running            8 (27d ago)         51d   10.0.0.49    dk8s-worker-1   <none>           <none>
nginx-statefulset-1                           1/1     Running            5 (27d ago)         51d   10.0.0.121   dk8s-worker-1   <none>           <none>
nginx-statefulset-2                           1/1     Running            5 (27d ago)         51d   10.0.2.208   dk8s-worker-3   <none>           <none>
ops-deck-api-84ddd69678-2278d                 0/1     CrashLoopBackOff   11182 (4m15s ago)   30d   10.0.0.107   dk8s-worker-1   <none>           <none>
ops-deck-api-84ddd69678-4tmks                 0/1     Running            11288 (5m25s ago)   30d   10.0.2.90    dk8s-worker-3   <none>           <none>
redis-deployment-d74599fc4-bt7vf              1/1     Running            9 (27d ago)         61d   10.0.2.60    dk8s-worker-3   <none>           <none>
```

**Tive que remover regras do kyverno que estavam invalidando os pods de subir**

```bash
OldReplicaSets:    <none>
NewReplicaSet:     locust-giropops-76cc56c95b (0/1 replicas created)
Events:
  Type     Reason           Age   From          Message
  ----     ------           ----  ----          -------
  Warning  PolicyViolation  16m   kyverno-scan  policy require-cpu-memory-limits/autogen-validate-limits fail: validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  Warning  PolicyViolation  16m   kyverno-scan  policy disallow-root-user/autogen-check-runAsNonRoot fail: validation error: Running as root user is not allowed. rule autogen-check-runAsNonRoot failed at path /spec/template/spec/containers/0/securityContext/
  Warning  PolicyViolation  16m   kyverno-scan  policy require-cpu-memory-limits/autogen-validate-limits fail: validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  Warning  PolicyViolation  16m   kyverno-scan  policy disallow-root-user/autogen-check-runAsNonRoot fail: validation error: Running as root user is not allowed. rule autogen-check-runAsNonRoot failed at path /spec/template/spec/containers/0/securityContext/
  Warning  PolicyViolation  46s   kyverno-scan  policy require-cpu-memory-limits/autogen-validate-limits fail: validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
```

```bash
$ kubectl get clusterpolicies.kyverno.io
NAME                               ADMISSION   BACKGROUND   READY   AGE   MESSAGE
add-label-namespace                true        true         True    8d    Ready
disallow-root-user                 true        true         True    8d    Ready
generate-configmap-for-namespace   true        true         True    8d    Ready
require-cpu-memory-limits          true        true         True    8d    Ready
╭─paulo@discovery ~ (⎈|k8s_40:N/A)
╰─$ kubectl delete clusterpolicies.kyverno.io disallow-root-user
clusterpolicy.kyverno.io "disallow-root-user" deleted
╭─paulo@discovery ~ (⎈|k8s_40:N/A)
╰─$ kubectl delete clusterpolicies.kyverno.io require-cpu-memory-limits
clusterpolicy.kyverno.io "require-cpu-memory-limits" deleted
```

**Agora o cluster já consegue realocar os pods**

```bash
$ kubectl get pods -o wide
NAME                                          READY   STATUS              RESTARTS      AGE   IP           NODE            NOMINATED NODE   READINESS GATES
giropops-senhas-deployment-69576c64cd-787dh   1/1     Running             9 (27d ago)   61d   10.0.2.229   dk8s-worker-3   <none>           <none>
giropops-senhas-deployment-69576c64cd-dzft2   1/1     Terminating         9 (27d ago)   61d   10.0.1.108   dk8s-worker-2   <none>           <none>
giropops-senhas-deployment-69576c64cd-h7fg8   1/1     Running             9 (27d ago)   61d   10.0.0.130   dk8s-worker-1   <none>           <none>
giropops-senhas-deployment-69576c64cd-tnhwr   1/1     Running             0             12s   10.0.2.54    dk8s-worker-3   <none>           <none>
locust-giropops-56fd678dbc-8fqd4              0/1     ContainerCreating   0             11s   <none>       dk8s-worker-3   <none>           <none>
locust-giropops-56fd678dbc-tf9dl              1/1     Terminating         0             15m   10.0.1.67    dk8s-worker-2   <none>           <none>
minion-deployment-c84f94b49-q62tz             1/1     Running             0             12s   10.0.2.69    dk8s-worker-3   <none>           <none>
nginx-hpa-58f69dfffc-cdmqj                    1/1     Running             0             11s   10.0.0.229   dk8s-worker-1   <none>           <none>
nginx-hpa-58f69dfffc-sw42m                    1/1     Running             0             15d   10.0.2.215   dk8s-worker-3   <none>           <none>
nginx-server-7d9cbf7d4f-dssgw                 2/2     Running             2 (27d ago)   30d   10.0.0.200   dk8s-worker-1   <none>           <none>
nginx-server-7d9cbf7d4f-s774s                 2/2     Running             0             12s   10.0.2.58    dk8s-worker-3   <none>           <none>
nginx-server-7d9cbf7d4f-wppbw                 2/2     Running             2 (27d ago)   30d   10.0.2.183   dk8s-worker-3   <none>           <none>
nginx-ssl-deployment-688dcb8fd-d47dj          1/1     Running             8 (27d ago)   51d   10.0.0.49    dk8s-worker-1   <none>           <none>
nginx-statefulset-0                           1/1     Running             0             11s   10.0.0.43    dk8s-worker-1   <none>           <none>
nginx-statefulset-1                           1/1     Running             0             12m   10.0.0.80    dk8s-worker-1   <none>           <none>
nginx-statefulset-2                           1/1     Running             0             12m   10.0.2.179   dk8s-worker-3   <none>           <none>
redis-deployment-d74599fc4-bt7vf              1/1     Running             9 (27d ago)   61d   10.0.2.60    dk8s-worker-3   <none>           <none>
```

**Remover Taint**

```bash
$ kubectl taint node dk8s-worker-2 manutencao=true:NoExecute-
node/dk8s-worker-2 untainted
```

**Reorganizar pods**

```bash
$ kubectl rollout restart statefulset nginx-statefulset
statefulset.apps/nginx-statefulset restarted
```

```bash
$ kubectl rollout restart deployment minion-deployment
deployment.apps/minion-deployment restarted
```

```bash
$ kubectl rollout restart deployment locust-giropops
deployment.apps/locust-giropops restarted
```

## Tolerations

### O que são Tolerations?

Tolerations (tolerâncias) são propriedades aplicadas aos **Pods** do Kubernetes que permitem que eles sejam agendados em nodes que possuem Taints correspondentes. Enquanto os Taints repelem Pods, as Tolerations permitem que um Pod "tolere" um Taint e seja executado em nodes que, de outra forma, o rejeitariam.

As Tolerations trabalham em conjunto com os Taints para criar um sistema de controle de agendamento mais granular e flexível no Kubernetes.

### Para que servem as Tolerations?

As Tolerations são utilizadas para:

1. **Permitir Acesso a Nodes Específicos**: Garantir que determinados Pods possam ser executados em nodes com Taints, como nodes com hardware especial (GPUs, SSDs de alta performance, etc.)
2. **Executar Pods Privilegiados**: Permitir que componentes do sistema ou ferramentas de monitoramento executem em todos os nodes, incluindo o control plane
3. **Manter Alta Disponibilidade**: Permitir que Pods críticos continuem executando mesmo quando um node está marcado para manutenção ou drenagem
4. **Gerenciar Problemas de Nodes**: Controlar quanto tempo um Pod deve permanecer em um node que apresentou problemas antes de ser removido
5. **Implementar Isolamento de Workloads**: Garantir que apenas workloads específicos tenham acesso a nodes dedicados

### Quando usar Tolerations?

Use Tolerations quando:

- **Necessidade de Hardware Específico**: Seus Pods precisam executar em nodes com recursos especiais (GPUs, FPGAs, memória específica)
- **Componentes de Sistema**: Você está executando DaemonSets, agentes de monitoramento ou logging que precisam rodar em todos os nodes
- **Ambientes Multi-tenant**: Você precisa garantir isolamento entre diferentes equipes ou aplicações
- **Controle de Manutenção**: Você quer controlar o comportamento de Pods durante operações de manutenção
- **Execução no Control Plane**: Você precisa executar Pods específicos nos nodes master (não recomendado para aplicações regulares)
- **Tratamento de Falhas**: Você quer controlar quanto tempo um Pod permanece em um node com problemas antes de ser movido

### Sintaxe de Tolerations

As Tolerations são definidas no manifest do Pod, dentro da seção `spec.tolerations`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
spec:
  tolerations:
  - key: "chave"
    operator: "Equal"
    value: "valor"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

### Operadores de Tolerations

Existem dois operadores disponíveis:

- **Equal**: A toleration corresponde ao Taint se a chave, valor e efeito forem iguais

  ```yaml
  tolerations:
  - key: "ambiente"
    operator: "Equal"
    value: "producao"
    effect: "NoSchedule"
  ```
- **Exists**: A toleration corresponde ao Taint se a chave existir (valor não é necessário)

  ```yaml
  tolerations:
  - key: "ambiente"
    operator: "Exists"
    effect: "NoSchedule"
  ```

### Exemplos Práticos

**Exemplo 1: Toleration para Node com GPU**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ml-training-pod
spec:
  tolerations:
  - key: "hardware"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
  containers:
  - name: tensorflow
    image: tensorflow/tensorflow:latest-gpu
    resources:
      limits:
        nvidia.com/gpu: 1
```

**Exemplo 2: Toleration para Ambiente de Produção**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-producao
spec:
  replicas: 3
  selector:
    matchLabels:
      app: producao
  template:
    metadata:
      labels:
        app: producao
    spec:
      tolerations:
      - key: "ambiente"
        operator: "Equal"
        value: "producao"
        effect: "NoSchedule"
      containers:
      - name: app
        image: minha-app:v1.0
```

**Exemplo 3: DaemonSet com Toleration para Todos os Nodes**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      tolerations:
      - operator: "Exists"
        effect: "NoSchedule"
      - operator: "Exists"
        effect: "NoExecute"
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
```

**Exemplo 4: Toleration com TolerationSeconds (para NoExecute)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-timeout
spec:
  tolerations:
  - key: "manutencao"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"
    tolerationSeconds: 3600  # Pod será removido após 1 hora
  containers:
  - name: nginx
    image: nginx
```

**Exemplo 5: Toleration para Node com Problemas**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-resiliente
spec:
  tolerations:
  - key: "node.kubernetes.io/not-ready"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300  # Espera 5 minutos antes de ser removido
  - key: "node.kubernetes.io/unreachable"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300
  containers:
  - name: app
    image: minha-app:v1.0
```

### Tolerations Automáticas

O Kubernetes adiciona automaticamente algumas tolerations aos Pods:

- `node.kubernetes.io/not-ready:NoExecute` (tolerationSeconds=300)
- `node.kubernetes.io/unreachable:NoExecute` (tolerationSeconds=300)
- `node.kubernetes.io/memory-pressure:NoSchedule`
- `node.kubernetes.io/disk-pressure:NoSchedule`
- `node.kubernetes.io/unschedulable:NoSchedule`

Essas tolerations automáticas podem ser sobrescritas conforme necessário.

### Diferença entre Taints e Tolerations

| Aspecto         | Taints                      | Tolerations                      |
| --------------- | --------------------------- | -------------------------------- |
| **Aplicado em** | Nodes                       | Pods                             |
| **Função**      | Repele Pods                 | Permite que Pods sejam agendados |
| **Quem define** | Administrador do cluster    | Desenvolvedor da aplicação       |
| **Ação**        | Rejeita Pods sem toleration | Aceita Taints do node            |

### Importante

- Uma Toleration apenas permite que um Pod seja agendado em um node com Taint, mas **não garante** que o Pod será agendado naquele node
- Para garantir que um Pod seja agendado em um node específico, combine Taints/Tolerations com **Node Affinity** ou **Node Selector**
- Uma Toleration sem `tolerationSeconds` significa que o Pod tolerará o Taint indefinidamente (para efeito NoExecute)
- Tolerations não funcionam como uma "lista de permissões" - um Pod pode ser agendado em nodes sem Taints mesmo que tenha Tolerations definidas

### Aula

```bash
$ kubectl taint node dk8s-worker-2 gpu=true:NoSchedule
node/dk8s-worker-2 tainted

╭─paulo@discovery ~ (⎈|k8s_40:N/A)
╰─$ kubectl taint node dk8s-worker-2 manutencao=true:NoExecute-

node/dk8s-worker-2 untainted
╭─paulo@discovery ~ (⎈|k8s_40:N/A)
╰─$ kubectl describe node dk8s-worker-2 | grep Taints
Taints:             gpu=true:NoSchedule
```

**Criar um deployment com definição de Tolerations**

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-gpu
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-gpu
  template:
    metadata:
      labels:
        app: nginx-gpu
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      tolerations:
      - key: "gpu"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
```

```bash
$ kubectl get pods -o wide
NAME                                          READY   STATUS    RESTARTS      AGE   IP           NODE            NOMINATED NODE   READINESS GATES
giropops-senhas-deployment-69576c64cd-787dh   1/1     Running   9 (27d ago)   61d   10.0.2.229   dk8s-worker-3   <none>           <none>
giropops-senhas-deployment-69576c64cd-h7fg8   1/1     Running   9 (27d ago)   61d   10.0.0.130   dk8s-worker-1   <none>           <none>
giropops-senhas-deployment-69576c64cd-tnhwr   1/1     Running   0             12m   10.0.2.54    dk8s-worker-3   <none>           <none>
locust-giropops-56fd678dbc-8fqd4              1/1     Running   0             12m   10.0.2.230   dk8s-worker-3   <none>           <none>
minion-deployment-c84f94b49-q62tz             1/1     Running   0             12m   10.0.2.69    dk8s-worker-3   <none>           <none>
nginx-gpu-6578b8cc5-77m2j                     1/1     Running   0             8s    10.0.0.236   dk8s-worker-1   <none>           <none>
nginx-gpu-6578b8cc5-gh428                     1/1     Running   0             8s    10.0.1.38    dk8s-worker-2   <none>           <none>
nginx-gpu-6578b8cc5-lr2bc                     1/1     Running   0             8s    10.0.2.158   dk8s-worker-3   <none>           <none>
nginx-gpu-6578b8cc5-nc9v5                     1/1     Running   0             8s    10.0.1.241   dk8s-worker-2   <none>           <none>
nginx-hpa-58f69dfffc-cdmqj                    1/1     Running   0             12m   10.0.0.229   dk8s-worker-1   <none>           <none>
nginx-hpa-58f69dfffc-sw42m                    1/1     Running   0             15d   10.0.2.215   dk8s-worker-3   <none>           <none>
nginx-server-7d9cbf7d4f-dssgw                 2/2     Running   2 (27d ago)   30d   10.0.0.200   dk8s-worker-1   <none>           <none>
nginx-server-7d9cbf7d4f-s774s                 2/2     Running   0             12m   10.0.2.58    dk8s-worker-3   <none>           <none>
nginx-server-7d9cbf7d4f-wppbw                 2/2     Running   2 (27d ago)   30d   10.0.2.183   dk8s-worker-3   <none>           <none>
nginx-ssl-deployment-688dcb8fd-d47dj          1/1     Running   8 (27d ago)   51d   10.0.0.49    dk8s-worker-1   <none>           <none>
nginx-statefulset-0                           1/1     Running   0             12m   10.0.0.43    dk8s-worker-1   <none>           <none>
nginx-statefulset-1                           1/1     Running   0             24m   10.0.0.80    dk8s-worker-1   <none>           <none>
nginx-statefulset-2                           1/1     Running   0             24m   10.0.2.179   dk8s-worker-3   <none>           <none>
redis-deployment-d74599fc4-bt7vf              1/1     Running   9 (27d ago)   61d   10.0.2.60    dk8s-worker-3   <none>           <none>
```

## Labels nos Nodes

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes ‹main●› (⎈|k8s_40:N/A)
╰─$ kubectl label nodes dk8s-worker-1 region=sa-east-1
node/dk8s-worker-1 labeled

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes ‹main●› (⎈|k8s_40:N/A)
╰─$ kubectl label nodes dk8s-worker-2 region=us-east-1
node/dk8s-worker-2 labeled

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes ‹main●› (⎈|k8s_40:N/A)
╰─$ kubectl label nodes dk8s-worker-3 region=sa-east-1
node/dk8s-worker-3 labeled

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes ‹main●› (⎈|k8s_40:N/A)
╰─$ k get nodes --show-labels
NAME                   STATUS   ROLES           AGE   VERSION   LABELS
dk8s-control-panel-1   Ready    control-plane   61d   v1.35.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=dk8s-control-panel-1,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
dk8s-worker-1          Ready    <none>          61d   v1.35.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=dk8s-worker-1,kubernetes.io/os=linux,region=sa-east-1
dk8s-worker-2          Ready    <none>          61d   v1.35.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=dk8s-worker-2,kubernetes.io/os=linux,region=us-east-1
dk8s-worker-3          Ready    <none>          61d   v1.35.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=dk8s-worker-3,kubernetes.io/os=linux,region=sa-east-1

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes ‹main●› (⎈|k8s_40:N/A)
╰─$ k get nodes dk8s-worker-2 --show-labels
NAME            STATUS   ROLES    AGE   VERSION   LABELS
dk8s-worker-2   Ready    <none>   61d   v1.35.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=dk8s-worker-2,kubernetes.io/os=linux,region=us-east-1

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes ‹main●› (⎈|k8s_40:N/A)
╰─$
```

```bash
$ kubectl label nodes dk8s-worker-2 gpu=true
node/dk8s-worker-2 labeled

paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes ‹main●› (⎈|k8s_40:N/A)
╰─$ k get nodes dk8s-worker-2 --show-labels
NAME            STATUS   ROLES    AGE   VERSION   LABELS
dk8s-worker-2   Ready    <none>   61d   v1.35.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,gpu=true,kubernetes.io/arch=amd64,kubernetes.io/hostname=dk8s-worker-2,kubernetes.io/os=linux,region=us-east-1

```

### Listar nodes mostrando as Labels

```bash
$ kubectl get nodes -L region,gpu
NAME                   STATUS   ROLES           AGE   VERSION   REGION      GPU
dk8s-control-panel-1   Ready    control-plane   61d   v1.35.0
dk8s-worker-1          Ready    <none>          61d   v1.35.0   sa-east-1
dk8s-worker-2          Ready    <none>          61d   v1.35.0   us-east-1   true
dk8s-worker-3          Ready    <none>          61d   v1.35.0   sa-east-1
```

## Affinity (Afinidade)

### O que é Affinity?

Affinity (afinidade) é um conjunto de regras que permite controlar de forma mais expressiva e flexível em quais nodes os Pods serão agendados. Diferente dos Taints/Tolerations que funcionam como uma "rejeição/permissão", o Affinity trabalha como uma "atração", permitindo que você especifique preferências ou requisitos para o agendamento de Pods.

Existem dois tipos principais de Affinity:

1. **Node Affinity**: Define regras de afinidade entre Pods e Nodes baseadas em labels
2. **Pod Affinity**: Define regras de afinidade entre Pods, permitindo co-localização
3. **Pod Anti-Affinity**: Define regras de separação entre Pods, evitando co-localização

### Para que serve o Affinity?

O Affinity é utilizado para:

1. **Otimização de Performance**: Colocar Pods próximos a recursos específicos (armazenamento local, cache, etc.)
2. **Alta Disponibilidade**: Distribuir réplicas de uma aplicação em diferentes zonas de disponibilidade ou racks
3. **Redução de Latência**: Co-localizar Pods que se comunicam frequentemente para reduzir latência de rede
4. **Considerações de Hardware**: Agendar Pods em nodes com características específicas (SSD, CPU específica, memória)
5. **Conformidade e Licenciamento**: Garantir que aplicações executem apenas em nodes específicos por questões regulatórias ou de licença
6. **Otimização de Custos**: Agendar workloads não-críticas em nodes de menor custo

### Node Affinity

Node Affinity permite especificar regras que limitam em quais nodes um Pod pode ser agendado, baseado nas labels dos nodes.

#### Tipos de Node Affinity

**1. requiredDuringSchedulingIgnoredDuringExecution** (Hard Affinity)
- Regra **obrigatória**: O Pod só será agendado se a regra for satisfeita
- Similar ao `nodeSelector`, mas com sintaxe mais expressiva

**2. preferredDuringSchedulingIgnoredDuringExecution** (Soft Affinity)
- Regra **preferencial**: O scheduler tentará satisfazer a regra, mas agendará o Pod mesmo se não conseguir
- Permite definir pesos (weight) para priorizar preferências

> **Nota**: "IgnoredDuringExecution" significa que se as labels de um node mudarem após o Pod estar em execução, o Pod não será removido.

#### Operadores Disponíveis

- `In`: O valor da label deve estar na lista especificada
- `NotIn`: O valor da label não deve estar na lista especificada
- `Exists`: A label deve existir (valor não importa)
- `DoesNotExist`: A label não deve existir
- `Gt`: Maior que (para valores numéricos)
- `Lt`: Menor que (para valores numéricos)

### Exemplo de Node Affinity

**Exemplo 1: Node Affinity Obrigatória (Hard)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-affinity-required
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: region
            operator: In
            values:
            - sa-east-1
            - sa-east-2
  containers:
  - name: nginx
    image: nginx
```

**Exemplo 2: Node Affinity Preferencial (Soft)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-affinity-preferred
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: gpu
            operator: In
            values:
            - "true"
      - weight: 20
        preference:
          matchExpressions:
          - key: ssd
            operator: Exists
  containers:
  - name: app
    image: minha-app:v1.0
```

**Exemplo 3: Combinando Hard e Soft Affinity**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-com-affinity
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        nodeAffinity:
          # Requisito obrigatório: node deve estar em produção
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: ambiente
                operator: In
                values:
                - producao
          # Preferência: nodes com SSD têm prioridade
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: disk-type
                operator: In
                values:
                - ssd
      containers:
      - name: nginx
        image: nginx:latest
```

### Pod Affinity

Pod Affinity permite agendar Pods baseado nas labels de outros Pods já em execução. Útil para co-localizar Pods que trabalham juntos.

**Exemplo de Pod Affinity:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  labels:
    app: backend
spec:
  containers:
  - name: backend
    image: backend:v1.0
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - backend
        topologyKey: kubernetes.io/hostname  # Mesmo node
  containers:
  - name: frontend
    image: frontend:v1.0
```

### Pod Anti-Affinity

Pod Anti-Affinity permite separar Pods, garantindo alta disponibilidade ao distribuí-los em diferentes nodes ou zonas.

**Exemplo de Pod Anti-Affinity:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web
            topologyKey: kubernetes.io/hostname  # Não colocar no mesmo node
      containers:
      - name: nginx
        image: nginx
```

### Comparação: Node Selector vs Node Affinity vs Taints/Tolerations

| Característica    | Node Selector    | Node Affinity           | Taints/Tolerations |
| ----------------- | ---------------- | ----------------------- | ------------------ |
| **Flexibilidade** | Baixa            | Alta                    | Média              |
| **Operadores**    | Apenas igualdade | In, NotIn, Exists, etc. | Equal, Exists      |
| **Soft Rules**    | Não              | Sim (preferred)         | Não                |
| **Perspectiva**   | Pod escolhe node | Pod escolhe node        | Node rejeita pod   |
| **Uso combinado** | Sim              | Sim                     | Sim                |

### TopologyKey

O `topologyKey` é fundamental em Pod Affinity/Anti-Affinity. Define o escopo da regra:

- `kubernetes.io/hostname`: Mesmo node
- `topology.kubernetes.io/zone`: Mesma zona de disponibilidade
- `topology.kubernetes.io/region`: Mesma região
- Qualquer label customizada nos nodes

### Quando usar cada tipo?

**Use Node Affinity quando:**
- Precisa de regras baseadas em características dos nodes
- Quer definir preferências com pesos
- Precisa de expressões complexas com múltiplos operadores

**Use Pod Affinity quando:**
- Precisa co-localizar Pods relacionados (ex: app e cache)
- Quer reduzir latência entre serviços
- Precisa agrupar Pods por questões de performance

**Use Pod Anti-Affinity quando:**
- Quer alta disponibilidade distribuindo réplicas
- Precisa evitar single point of failure
- Quer separar Pods por segurança ou isolamento

**Use Taints/Tolerations quando:**
- Precisa dedicar nodes para workloads específicas
- Quer impedir que Pods sejam agendados em nodes específicos
- A decisão de rejeição parte do node, não do Pod

### Exemplo Completo: Alta Disponibilidade com Anti-Affinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-ha
spec:
  replicas: 3
  selector:
    matchLabels:
      app: critical-app
  template:
    metadata:
      labels:
        app: critical-app
    spec:
      affinity:
        # Node Affinity: nodes de produção com SSD
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: ambiente
                operator: In
                values:
                - producao
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: disk-type
                operator: In
                values:
                - ssd
        # Pod Anti-Affinity: distribuir em nodes diferentes
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - critical-app
            topologyKey: kubernetes.io/hostname
          # Preferir zonas diferentes
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - critical-app
              topologyKey: topology.kubernetes.io/zone
      containers:
      - name: app
        image: critical-app:v1.0
        resources:
          requests:
            memory: "256Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"
            cpu: "1000m"
```

### Aula

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-gpu
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-gpu
  template:
    metadata:
      labels:
        app: nginx-gpu
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: gpu
                operator: In
                values:
                - "true"
```

## AntiAffinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-region
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-region
  template:
    metadata:
      labels:
        app: nginx-region
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: nginx-region
            topologyKey: "region"
```

## preferredScheduling e requiredScheduling