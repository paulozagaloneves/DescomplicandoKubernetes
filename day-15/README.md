# Day 15 - Descomplicando Kyverno e as policies K8s

## Kyverno

**Kyverno** é um mecanismo de validação, mutação e geração de políticas para Kubernetes. Funciona como um Admission Controller que permite criar e aplicar políticas de segurança, conformidade e governança no cluster Kubernetes.

Principais características:
1. Validação (Validating Policies)

- Bloqueia ou aprova requisições que não atendem aos critérios da política
- Exemplos: bloquear pods sem resource limits, exigir labels específicos, validar assinaturas de imagens
  
2. Mutação (Mutating Policies)

- Modifica automaticamente recursos antes deles serem criados
- Exemplos: adicionar automaticamente labels, injetar sidecars, configurar pull policy em imagens
  
3. Geração (Generating Policies)

- Cria recursos automaticamente baseado em geradores
- Exemplos: gerar NetworkPolicies ou RBACs automaticamente

4. Limpeza (Cleanup Policies)

- Remove recursos que não mais correspondem às políticas
- Exemplos: limpar recursos órfãos ou expirados

Diferenciais do Kyverno:

- ✅ Native Kubernetes: Usa CRDs padrão do Kubernetes, sem necessidade de linguagem externa
- ✅ Linguagem simples: Políticas em YAML (fácil de entender e manter)
- ✅ Não requer containers sidecar: Diferente de OPA/Gatekeeper
- ✅ Múltiplos controladores: Admission, Background, Reports e Cleanup
- ✅ Sem dívida técnica: Mantido pela comunidade Cloud Native Computing Foundation

Casos de uso comuns:

- Enforçar segurança (ex: bloquear containers privilegiados)
- Garantir conformidade (ex: obrigar policy de network)
- Gerenciar imagens (ex: validar registro de origem)
- Aplicar padrões organizacionais (ex: exigir labels de custo, owner)
- Prevenir misconfigurations (ex: exigir resource limits e health probes)


### Instalação

**Adicionar repositorio no helm**

```bash
$ helm repo add kyverno https://kyverno.github.io/kyverno/
"kyverno" has been added to your repositories
```

**Update cache**
```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kyverno" chart repository
...Successfully got an update from the "traefik" chart repository
Update Complete. ⎈Happy Helming!⎈
```


**Instalar kyverno**

```bash
$ helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
NAME: kyverno
LAST DEPLOYED: Thu Feb 26 18:22:49 2026
NAMESPACE: kyverno
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
NOTES:
Chart version: 3.7.1
Kyverno version: v1.17.1

Thank you for installing kyverno! Your release is named kyverno.

The following components have been installed in your cluster:
- CRDs
- Admission controller
- Reports controller
- Cleanup controller
- Background controller


⚠  WARNING: Setting the admission controller replica count below 2 means Kyverno is not running in high availability mode.


⚠  WARNING: PolicyExceptions are disabled by default. To enable them, set '--enablePolicyException' to true.

💡 Note: There is a trade-off when deciding which approach to take regarding Namespace exclusions. Please see the documentation at https://kyverno.io/docs/installation/#security-vs-operability to understand the risks.
```

**Validar**

```bash
$ kubectl get pods -n kyverno              
NAME                                             READY   STATUS    RESTARTS   AGE
kyverno-admission-controller-659d58644b-8n57f    1/1     Running   0          9m32s
kyverno-background-controller-778bffc669-drvqf   1/1     Running   0          9m32s
kyverno-cleanup-controller-8bfc4f578-2g8l7       1/1     Running   0          9m32s
kyverno-reports-controller-6c666d96-z99x9        1/1     Running   0          9m32s
```

```bash
$ kubectl get crd | grep kyverno      
cleanuppolicies.kyverno.io                              2026-02-26T18:22:35Z
clustercleanuppolicies.kyverno.io                       2026-02-26T18:22:35Z
clusterephemeralreports.reports.kyverno.io              2026-02-26T18:22:35Z
clusterpolicies.kyverno.io                              2026-02-26T18:22:35Z
deletingpolicies.policies.kyverno.io                    2026-02-26T18:22:35Z
ephemeralreports.reports.kyverno.io                     2026-02-26T18:22:35Z
generatingpolicies.policies.kyverno.io                  2026-02-26T18:22:35Z
globalcontextentries.kyverno.io                         2026-02-26T18:22:35Z
imagevalidatingpolicies.policies.kyverno.io             2026-02-26T18:22:35Z
mutatingpolicies.policies.kyverno.io                    2026-02-26T18:22:35Z
namespaceddeletingpolicies.policies.kyverno.io          2026-02-26T18:22:35Z
namespacedgeneratingpolicies.policies.kyverno.io        2026-02-26T18:22:35Z
namespacedimagevalidatingpolicies.policies.kyverno.io   2026-02-26T18:22:35Z
namespacedmutatingpolicies.policies.kyverno.io          2026-02-26T18:22:35Z
namespacedvalidatingpolicies.policies.kyverno.io        2026-02-26T18:22:35Z
policies.kyverno.io                                     2026-02-26T18:22:35Z
policyexceptions.kyverno.io                             2026-02-26T18:22:35Z
policyexceptions.policies.kyverno.io                    2026-02-26T18:22:35Z
updaterequests.kyverno.io                               2026-02-26T18:22:35Z
validatingpolicies.policies.kyverno.io                  2026-02-26T18:22:35Z
```

### Criação Policy no Kyverno

```bash
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-cpu-memory-limits
spec:
  validationFailureAction: Enforce
  rules:
  - name: validate-limits
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "CPU and memory limits are required"
      pattern:
        spec:
          containers:
          - name: "*"
            resources:
              limits:
                memory: "?*"
                cpu: "?*"
```


**Obter policy criada**

```bash
$ kubectl get clusterpolicies.kyverno.io         
NAME                        ADMISSION   BACKGROUND   READY   AGE   MESSAGE
require-cpu-memory-limits   true        true         True    14m   Ready
```


**Describe**

```bash
$ kubectl describe clusterpolicies.kyverno.io require-cpu-memory-limits 
Name:         require-cpu-memory-limits
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  kyverno.io/v1
Kind:         ClusterPolicy
Metadata:
  Creation Timestamp:  2026-02-26T19:00:56Z
  Generation:          2
  Resource Version:    12334443
  UID:                 5b52bad9-0d29-4d31-8dd6-1e892cd029df
Spec:
  Admission:     true
  Background:    true
  Emit Warning:  false
  Rules:
    Match:
      Resources:
        Kinds:
          Pod
    Name:                      validate-limits
    Skip Background Requests:  true
    Validate:
      Allow Existing Violations:  true
      Message:                    CPU and memory limits are required
      Pattern:
        Spec:
          Containers:
            Name:  *
            Resources:
              Limits:
                Cpu:          ?*
                Memory:       ?*
  Validation Failure Action:  Enforce
Status:
  Autogen:
    Rules:
      Match:
        Resources:
          Kinds:
            DaemonSet
            Deployment
            Job
            ReplicaSet
            ReplicationController
            StatefulSet
      Name:                      autogen-validate-limits
      Skip Background Requests:  true
      Validate:
        Allow Existing Violations:  true
        Message:                    CPU and memory limits are required
        Pattern:
          Spec:
            Template:
              Spec:
                Containers:
                  Name:  *
                  Resources:
                    Limits:
                      Cpu:     ?*
                      Memory:  ?*
      Match:
        Resources:
          Kinds:
            CronJob
      Name:                      autogen-cronjob-validate-limits
      Skip Background Requests:  true
      Validate:
        Allow Existing Violations:  true
        Message:                    CPU and memory limits are required
        Pattern:
          Spec:
            Job Template:
              Spec:
                Template:
                  Spec:
                    Containers:
                      Name:  *
                      Resources:
                        Limits:
                          Cpu:     ?*
                          Memory:  ?*
  Conditions:
    Last Transition Time:  2026-02-26T19:00:56Z
    Message:               Ready
    Reason:                Succeeded
    Status:                True
    Type:                  Ready
  Rulecount:
    Generate:      0
    Mutate:        0
    Validate:      1
    Verifyimages:  0
  Validatingadmissionpolicy:
    Generated:  false
    Message:    skip generating ValidatingAdmissionPolicy for non CEL rules.
Events:
  Type     Reason           Age   From               Message
  ----     ------           ----  ----               -------
  Warning  PolicyViolation  16m   kyverno-scan       DaemonSet metallb-system/speaker: [autogen-validate-limits] fail; validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  Warning  PolicyViolation  16m   kyverno-scan       DaemonSet kube-system/kube-proxy: [autogen-validate-limits] fail; validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  Warning  PolicyViolation  16m   kyverno-scan       DaemonSet cilium-test-1/host-netns: [autogen-validate-limits] fail; validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  Warning  PolicyViolation  16m   kyverno-scan       DaemonSet cilium-test-1/host-netns-non-cilium: [autogen-validate-limits] fail; validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  ...
  Warning  PolicyViolation  15m   kyverno-scan       Deployment kyverno/kyverno-background-controller: [autogen-validate-limits] fail; validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/cpu/
  Warning  PolicyViolation  15m   kyverno-scan       Deployment traefik/traefik: [autogen-validate-limits] fail; validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  Warning  PolicyViolation  15m   kyverno-scan       Deployment default/nginx-ssl-deployment: [autogen-validate-limits] fail; validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  Warning  PolicyViolation  15m   kyverno-scan       Deployment kube-system/hubble-ui: [autogen-validate-limits] fail; validation error: CPU and memory limits are required. rule autogen-validate-limits failed at path /spec/template/spec/containers/0/resources/limits/
  Warning  PolicyViolation  11m   kyverno-admission  Pod default/nginx-kyverno-validation: [validate-limits] fail (blocked); validation error: CPU and memory limits are required. rule validate-limits failed at path /spec/containers/0/resources/limits/
```




#### Testar a regra

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kyverno-validation
spec:
  containers:
  - name: nginx-kyverno-validation
    image: nginx:latest
```

```bash
$ kubectl apply -f ./pod.yaml                     
Error from server: error when creating "./pod.yaml": admission webhook "validate.kyverno.svc-fail" denied the request: 

resource Pod/default/nginx-kyverno-validation was blocked due to the following policies 

require-cpu-memory-limits:
  validate-limits: 'validation error: CPU and memory limits are required. rule validate-limits
    failed at path /spec/containers/0/resources/limits/'
```


### Policy tipo Mutate




**File:** mutate-policy-add-label-namespace.yaml

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-label-namespace
spec:
  rules:
  - name: add-label-ns
    match:
      resources:
        kinds:
        - Namespace
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            app.kubernetes.io/owner: "kode.expert"
```

```bash
$ kubectl apply -f ./mutate-policy-add-label-namespace.yaml
```

**Validar policies**

```bash
$ kubectl get clusterpolicies.kyverno.io -o wide
NAME                        ADMISSION   BACKGROUND   READY   AGE   FAILURE POLICY   VALIDATE   MUTATE   GENERATE   VERIFY IMAGES   MESSAGE
add-label-namespace         true        true         True    44s                    0          1        0          0               Ready
require-cpu-memory-limits   true        true         True    32m                    1          0        0          0               Ready
```


#### criar namespace

```bash
$ kubectl create namespace integration 
namespace/integration created
$
$ $ kubectl describe namespace integration 
Name:         integration
Labels:       app.kubernetes.io/owner=kode.expert
              kubernetes.io/metadata.name=integration
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```


### Policy Generate

**Generating Policies** no Kyverno são rules que **criam automaticamente novos recursos** quando um recurso base é criado, sem necessidade de intervenção manual.

**Como funcionam:**

Quando um recurso (ex: Namespace) é criado, a policy gera automaticamente outro recurso (ex: ConfigMap, NetworkPolicy, RBAC) no mesmo cluster.

**Quando usar:**

1. Padrões organizacionais automáticos

- Criar ConfigMap padrão em cada namespace novo
- Gerar NetworkPolicies default para isolamento
- Provisionar RBACs básicos automaticamente

2. Conformidade e governança

- Criar ResourceQuota em cada namespace
- Gerar PodSecurityPolicy automaticamente
- Provisionar monitoring/logging sidecars

3. Infraestrutura como código

- Gerar Ingress automático baseado em label
- Criar secrets padrão em namespaces
- Provisionar storage volumes automaticamente

**Vantagens:**

- ✅ Elimina configuração manual repetitiva
- ✅ Garante padrões em toda a organização
- ✅ Reduz erros humanos
- ✅ Otimiza onboarding de novos namespaces

**Diferença vs Mutate:**

- **Mutate**: modifica o recurso que está sendo criado (ex: adiciona label ao Pod)
- **Generate**: cria um recurso totalmente novo no cluster (ex: cria ConfigMap quando Namespace nasce)

#### Nossa policy

**File:** generate-cm-adding-ns.yaml

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: generate-configmap-for-namespace
spec:
  rules:
    - name: generate-namespace-configmap
      match:
        resources:
          kinds:
            - Namespace
      generate:
        apiVersion: v1
        kind: ConfigMap
        name: ns-default-configmap
        namespace: "{{request.object.metadata.name}}"
        data:
          data:
            LOG_LEVEL: "info"
            LOG_FORMAT: "json"
            TZ: "UTC"
            APP_PORT: "8080"
            REQUEST_TIMEOUT: "5s"
            RETRY_MAX: "3"
            FEATURE_FLAGS: "billing=false,search=true"
            OTEL_EXPORTER_OTLP_ENDPOINT: "http://otel-collector.observability:4317"
            OTEL_SERVICE_NAME: "default-app"
```

```bash
$ kubectl apply -f ./generate-cm-adding-ns.yaml
clusterpolicy.kyverno.io/generate-configmap-for-namespace created
$
$
$ kubectl get clusterpolicies.kyverno.io -o wide
NAME                               ADMISSION   BACKGROUND   READY   AGE   FAILURE POLICY   VALIDATE   MUTATE   GENERATE   VERIFY IMAGES   MESSAGE
add-label-namespace                true        true         True    16m                    0          1        0          0               Ready
generate-configmap-for-namespace   true        true         True    7s                     0          0        1          0               Ready
require-cpu-memory-limits          true        true         True    48m                    1          0        0          0               Ready
```


**Criando o nosso namespace e validando se configmap foi criado**

```bash
$ kubectl create namespace giropops             
namespace/giropops created
$
$ kubectl get configmaps -n giropops  
NAME                   DATA   AGE
kube-root-ca.crt       1      20s
ns-default-configmap   9      20s
```

```bash
$ kubectl describe configmaps -n giropops ns-default-configmap
Name:         ns-default-configmap
Namespace:    giropops
Labels:       app.kubernetes.io/managed-by=kyverno
              generate.kyverno.io/policy-name=generate-configmap-for-namespace
              generate.kyverno.io/policy-namespace=
              generate.kyverno.io/rule-name=generate-namespace-configmap
              generate.kyverno.io/trigger-group=
              generate.kyverno.io/trigger-kind=Namespace
              generate.kyverno.io/trigger-namespace=
              generate.kyverno.io/trigger-uid=75e1fe1e-bd83-44cf-b378-ab7aadbca111
              generate.kyverno.io/trigger-version=v1
Annotations:  <none>

Data
====
APP_PORT:
----
8080

FEATURE_FLAGS:
----
billing=false,search=true

LOG_FORMAT:
----
json

LOG_LEVEL:
----
info

OTEL_EXPORTER_OTLP_ENDPOINT:
----
http://otel-collector.observability:4317

OTEL_SERVICE_NAME:
----
default-app

REQUEST_TIMEOUT:
----
5s

RETRY_MAX:
----
3

TZ:
----
UTC


BinaryData
====

Events:  <none>
```


### Policy que proibe containers runAsRoot


**File:** disallow-root-user.yaml


```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-root-user
  annotations:
    policies.kyverno.io/description: "Disallow containers from running as root user"
spec:
  validationFailureAction: Enforce
#  background: true
  rules:
  - name: check-runAsNonRoot
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Running as root user is not allowed"
      pattern:
        spec:
          containers:
          - securityContext:
              runAsNonRoot: true
```

```bash
$ kubectl apply -f ./disallow-root-user.yaml           
clusterpolicy.kyverno.io/disallow-root-user created

$ kubectl get clusterpolicies.kyverno.io -o wide         
NAME                               ADMISSION   BACKGROUND   READY   AGE   FAILURE POLICY   VALIDATE   MUTATE   GENERATE   VERIFY IMAGES   MESSAGE
add-label-namespace                true        true         True    43m                    0          1        0          0               Ready
disallow-root-user                 true        true         True    51s                    1          0        0          0               Ready
generate-configmap-for-namespace   true        true         True    27m                    0          0        1          0               Ready
require-cpu-memory-limits          true        true         True    75m                    1          0        0          0               Ready
```


**Testando a policy**

```bash
$ kubectl apply -f ./pod.yaml 
Error from server: error when creating "./pod.yaml": admission webhook "validate.kyverno.svc-fail" denied the request: 

resource Pod/default/nginx-kyverno-validation was blocked due to the following policies 

disallow-root-user:
  check-runAsNonRoot: 'validation error: Running as root user is not allowed. rule
    check-runAsNonRoot failed at path /spec/containers/0/securityContext/'
```


**Após também remover os recursos do pod**

```bash
$ kubectl apply -f ./pod.yaml                                                                                                                                                           1 ↵
Error from server: error when creating "./pod.yaml": admission webhook "validate.kyverno.svc-fail" denied the request: 

resource Pod/default/nginx-kyverno-validation was blocked due to the following policies 

disallow-root-user:
  check-runAsNonRoot: 'validation error: Running as root user is not allowed. rule
    check-runAsNonRoot failed at path /spec/containers/0/securityContext/'
require-cpu-memory-limits:
  validate-limits: 'validation error: CPU and memory limits are required. rule validate-limits
    failed at path /spec/containers/0/resources/limits/'
```