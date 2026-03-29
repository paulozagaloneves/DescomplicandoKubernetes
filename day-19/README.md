# Day 19 - Helm

## Índice

- [Day 19 - Helm](#day-19---helm)
  - [Índice](#índice)
  - [O que é o Helm e HelmChart](#o-que-é-o-helm-e-helmchart)
  - [Instalando Helm](#instalando-helm)
    - [Helm Help](#helm-help)
  - [Instalando a nossa App](#instalando-a-nossa-app)
    - [Clone](#clone)
    - [Instalando](#instalando)
  - [Criando primeiro Chart](#criando-primeiro-chart)
    - [Estrutura do Chart](#estrutura-do-chart)
      - [Chart.yaml](#chartyaml)
      - [value.yaml](#valueyaml)
      - [templates](#templates)
    - [Deploy](#deploy)
    - [List](#list)
    - [Uninstall](#uninstall)
  - [Melhorando Values](#melhorando-values)
    - [values.yaml](#valuesyaml)
    - [Templates](#templates-1)
    - [Install](#install)

## O que é o Helm e HelmChart

O **Helm** é um gestor de pacotes para Kubernetes.  
Permite instalar, atualizar, versionar e remover aplicações no cluster de forma simples e padronizada.

Um **HelmChart** (ou apenas **Chart**) é o pacote usado pelo Helm.  
Esse pacote inclui templates de recursos Kubernetes (como `Deployment`, `Service` e `Ingress`) e ficheiros de configuração, como o `values.yaml`, onde são definidos os valores personalizados da instalação.

Em resumo:

- **Helm**: a ferramenta que gere aplicações no Kubernetes.
- **Chart**: o pacote/modelo da aplicação.
- **Release**: uma instalação de um Chart num cluster/namespace.

## Instalando Helm

**Referência:** https://helm.sh/pt/docs/intro/install

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```

ou em apenas uma linha de comando

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4 | bash
```

### Helm Help

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ helm     
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment variables:

| Name                               | Description                                                                                                |
|------------------------------------|------------------------------------------------------------------------------------------------------------|
| $HELM_CACHE_HOME                   | set an alternative location for storing cached files.                                                      |
| $HELM_CONFIG_HOME                  | set an alternative location for storing Helm configuration.                                                |
| $HELM_DATA_HOME                    | set an alternative location for storing Helm data.                                                         |
| $HELM_DEBUG                        | indicate whether or not Helm is running in Debug mode                                                      |
| $HELM_DRIVER                       | set the backend storage driver. Values are: configmap, secret, memory, sql.                                |
| $HELM_DRIVER_SQL_CONNECTION_STRING | set the connection string the SQL storage driver should use.                                               |
| $HELM_MAX_HISTORY                  | set the maximum number of helm release history.                                                            |
| $HELM_NAMESPACE                    | set the namespace used for the helm operations.                                                            |
| $HELM_NO_PLUGINS                   | disable plugins. Set HELM_NO_PLUGINS=1 to disable plugins.                                                 |
| $HELM_PLUGINS                      | set the path to the plugins directory                                                                      |
| $HELM_REGISTRY_CONFIG              | set the path to the registry config file.                                                                  |
| $HELM_REPOSITORY_CACHE             | set the path to the repository cache directory                                                             |
| $HELM_REPOSITORY_CONFIG            | set the path to the repositories file.                                                                     |
| $KUBECONFIG                        | set an alternative Kubernetes configuration file (default "~/.kube/config")                                |
| $HELM_KUBEAPISERVER                | set the Kubernetes API Server Endpoint for authentication                                                  |
| $HELM_KUBECAFILE                   | set the Kubernetes certificate authority file.                                                             |
| $HELM_KUBEASGROUPS                 | set the Groups to use for impersonation using a comma-separated list.                                      |
| $HELM_KUBEASUSER                   | set the Username to impersonate for the operation.                                                         |
| $HELM_KUBECONTEXT                  | set the name of the kubeconfig context.                                                                    |
| $HELM_KUBETOKEN                    | set the Bearer KubeToken used for authentication.                                                          |
| $HELM_KUBEINSECURE_SKIP_TLS_VERIFY | indicate if the Kubernetes API server's certificate validation should be skipped (insecure)                |
| $HELM_KUBETLS_SERVER_NAME          | set the server name used to validate the Kubernetes API server certificate                                 |
| $HELM_BURST_LIMIT                  | set the default burst limit in the case the server contains many CRDs (default 100, -1 to disable)         |
| $HELM_QPS                          | set the Queries Per Second in cases where a high number of calls exceed the option for higher burst values |
| $HELM_COLOR                        | set color output mode. Allowed values: never, always, auto (default: never)                                |
| $NO_COLOR                          | set to any non-empty value to disable all colored output (overrides $HELM_COLOR)                           |

Helm stores cache, configuration, and data based on the following configuration order:

- If a HELM_*_HOME environment variable is set, it will be used
- Otherwise, on systems supporting the XDG base directory specification, the XDG variables will be used
- When no other location is set a default location will be used based on the operating system

By default, the default directories depend on the Operating System. The defaults are listed below:

| Operating System | Cache Path                | Configuration Path             | Data Path               |
|------------------|---------------------------|--------------------------------|-------------------------|
| Linux            | $HOME/.cache/helm         | $HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | $HOME/Library/Preferences/helm | $HOME/Library/helm      |
| Windows          | %TEMP%\helm               | %APPDATA%\helm                 | %APPDATA%\helm          |

Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart's dependencies
  env         helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examine a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  push        push a chart to remote
  registry    login to or logout from a registry
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      display the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the helm version information

Flags:
      --burst-limit int                 client-side default throttling limit (default 100)
      --color string                    use colored output (never, auto, always) (default "auto")
      --colour string                   use colored output (never, auto, always) (default "auto")
      --content-cache string            path to the directory containing cached content (e.g. charts) (default "/home/paulo/.cache/helm/content")
      --debug                           enable verbose output
  -h, --help                            help for helm
      --kube-apiserver string           the address and the port for the Kubernetes API server
      --kube-as-group stringArray       group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --kube-as-user string             username to impersonate for the operation
      --kube-ca-file string             the certificate authority file for the Kubernetes API server connection
      --kube-context string             name of the kubeconfig context to use
      --kube-insecure-skip-tls-verify   if true, the Kubernetes API server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kube-tls-server-name string     server name to use for Kubernetes API server certificate validation. If it is not provided, the hostname used to contact the server is used
      --kube-token string               bearer token used for authentication
      --kubeconfig string               path to the kubeconfig file
  -n, --namespace string                namespace scope for this request
      --qps float32                     queries per second used when communicating with the Kubernetes API, not including bursting
      --registry-config string          path to the registry config file (default "/home/paulo/.config/helm/registry/config.json")
      --repository-cache string         path to the directory containing cached repository indexes (default "/home/paulo/.cache/helm/repository")
      --repository-config string        path to the file containing repository names and URLs (default "/home/paulo/.config/helm/repositories.yaml")

Use "helm [command] --help" for more information about a command.

```

## Instalando a nossa App

### Clone

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19 ‹main› (⎈|k8s_40:default)
╰─$ git clone git@github.com:badtuxx/giropops-senhas-labs.git

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19 ‹main●› (⎈|k8s_40:default)
╰─$ cd giropops-senhas-labs/giropops-senhas/
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19 ‹main●› (⎈|k8s_40:default)
╰─$ cd giropops-senhas-labs/giropops-senhas/

```

### Instalando

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$ ls -lh
total 44K
-rw-rw-r-- 1 paulo paulo  431 mar 28 18:42 app-deployment.yaml
-rw-rw-r-- 1 paulo paulo 2,5K mar 28 18:42 app.py
-rw-rw-r-- 1 paulo paulo  767 mar 28 18:42 app-servicemonitor.yaml
-rw-rw-r-- 1 paulo paulo  269 mar 28 18:42 app-service.yaml
-rw-rw-r-- 1 paulo paulo  230 mar 28 18:42 Dockerfile
-rw-rw-r-- 1 paulo paulo  499 mar 28 18:42 redis-deployment.yaml
-rw-rw-r-- 1 paulo paulo  180 mar 28 18:42 redis-service.yaml
-rw-rw-r-- 1 paulo paulo   51 mar 28 18:42 requirements.txt
drwxrwxr-x 4 paulo paulo 4,0K mar 28 18:42 static
-rw-rw-r-- 1 paulo paulo  220 mar 28 18:42 tailwind.config.js
drwxrwxr-x 2 paulo paulo 4,0K mar 28 18:42 templates
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$ kubectl apply -f app-deployment.yaml   
deployment.apps/giropops-senhas created

╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$ kubectl apply -f app-service.yaml   
service/giropops-senhas configured
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$ kubectl apply -f redis-deployment.yaml        
deployment.apps/redis-deployment created
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$ kubectl apply -f redis-service.yaml   
service/redis-service unchanged
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$ kubectl apply -f redis-service.yaml
service/redis-service created

```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$ kubectl get pods                 
NAME                                   READY   STATUS    RESTARTS      AGE
alpine                                 1/1     Running   2 (11d ago)   11d
giropops-senhas-547b654d89-c8hfs       1/1     Running   0             6m33s
giropops-senhas-547b654d89-wqmtg       1/1     Running   0             6m33s
minion-deployment-c84f94b49-q62tz      1/1     Running   0             21d
nginx-ssl-deployment-688dcb8fd-d47dj   1/1     Running   8 (48d ago)   72d
nginx-statefulset-0                    1/1     Running   0             21d
nginx-statefulset-1                    1/1     Running   0             21d
nginx-statefulset-2                    1/1     Running   0             21d
redis-deployment-7d859f6bc4-7m5kw      1/1     Running   0             92s
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-senhas-labs/giropops-senhas ‹main› (⎈|k8s_40:default)
╰─$ kubectl get svc               
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
giropops-senhas         NodePort       10.108.156.146   <none>         5000:32500/TCP               43s
grafana-oltp-dev        ClusterIP      None             <none>         4318/TCP                     65d
kubernetes              ClusterIP      10.96.0.1        <none>         443/TCP                      83d
locust-giropops         LoadBalancer   10.98.162.173    192.168.1.77   80:30923/TCP                 30d
mariadb-dev             ClusterIP      None             <none>         3306/TCP                     65d
minion-service          LoadBalancer   10.102.167.78    192.168.1.72   80:30444/TCP                 73d
nginx-headless          ClusterIP      None             <none>         80/TCP                       82d
nginx-hpa               LoadBalancer   10.102.167.6     192.168.1.76   80:32178/TCP                 37d
nginx-metrics-service   ClusterIP      10.103.136.125   <none>         9113/TCP                     51d
nginx-ssl-service       LoadBalancer   10.99.231.150    192.168.1.73   80:31713/TCP,443:32467/TCP   72d
ops-deck-api-service    LoadBalancer   10.107.159.246   192.168.1.74   80:32258/TCP                 65d
redis-dev               ClusterIP      None             <none>         6379/TCP                     65d
redis-service           ClusterIP      10.111.45.153    <none>         6379/TCP                     55s
```



## Criando primeiro Chart


### Estrutura do Chart

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ tree                                                                       
.
├── charts
├── Chart.yaml
├── templates
│   ├── app-deployment.yaml
│   ├── app-service.yaml
│   ├── redis-deployment.yaml
│   └── redis-service.yaml
└── values.yaml

3 directories, 6 files
```


#### Chart.yaml

**File:** Chart.yaml

```yaml
apiVersion: v2
name: giropops-senhas-chart
description: Gipopops Senhas Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0"
sources:
  - https://github.com/badtuxx/giropops-senhas
maintainers:
  - name: LinuxTips
    email: linux.tips@example.com
```

#### value.yaml

**File:** values.yaml

```yaml
giropopsSenhas:
  name: "giropops-senhas-helm"
  image: "linuxtips/giropops-senhas:1.0"

redis:
  name: "redis-helm"
  image: "redis:8.4"
```

#### templates

**File:** app-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: giropops-senhas
  name: {{ .Values.giropopsSenhas.name }}
spec:
  replicas: 2
  selector:
    matchLabels:
      app: giropops-senhas
  template:
    metadata:
      labels:
        app: giropops-senhas
    spec:
      containers:
      - image: {{ .Values.giropopsSenhas.image }}
        name: giropops-senhas
        ports:
        - containerPort: 5000
        imagePullPolicy: Always
```


**File:** redis-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: {{ .Values.redis.name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: {{ .Values.redis.image }}
        name: redis
        ports:
          - containerPort: 6379
        resources:
          limits:
            memory: "256Mi"
            cpu: "500m"
          requests:
            memory: "128Mi"
            cpu: "250m"
```

### Deploy


```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ helm install giropops-senhas ./   
NAME: giropops-senhas
LAST DEPLOYED: Sun Mar 29 08:52:53 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
```


```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get pods                                  
NAME                                    READY   STATUS    RESTARTS      AGE
alpine                                  1/1     Running   2 (11d ago)   11d
giropops-senhas-helm-547b654d89-277hs   1/1     Running   0             52s
giropops-senhas-helm-547b654d89-wcnnp   1/1     Running   0             52s
minion-deployment-c84f94b49-q62tz       1/1     Running   0             21d
nginx-ssl-deployment-688dcb8fd-d47dj    1/1     Running   8 (49d ago)   73d
nginx-statefulset-0                     1/1     Running   0             21d
nginx-statefulset-1                     1/1     Running   0             21d
nginx-statefulset-2                     1/1     Running   0             21d
redis-helm-dc8cf4ddd-jt4qc              1/1     Running   0             52s
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get svc 
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
giropops-senhas         NodePort       10.109.60.136    <none>         5000:32500/TCP               91s
grafana-oltp-dev        ClusterIP      None             <none>         4318/TCP                     66d
kubernetes              ClusterIP      10.96.0.1        <none>         443/TCP                      83d
locust-giropops         LoadBalancer   10.98.162.173    192.168.1.77   80:30923/TCP                 31d
mariadb-dev             ClusterIP      None             <none>         3306/TCP                     66d
minion-service          LoadBalancer   10.102.167.78    192.168.1.72   80:30444/TCP                 74d
nginx-headless          ClusterIP      None             <none>         80/TCP                       83d
nginx-hpa               LoadBalancer   10.102.167.6     192.168.1.76   80:32178/TCP                 37d
nginx-metrics-service   ClusterIP      10.103.136.125   <none>         9113/TCP                     52d
nginx-ssl-service       LoadBalancer   10.99.231.150    192.168.1.73   80:31713/TCP,443:32467/TCP   73d
ops-deck-api-service    LoadBalancer   10.107.159.246   192.168.1.74   80:32258/TCP                 66d
redis-dev               ClusterIP      None             <none>         6379/TCP                     66d
redis-service           ClusterIP      10.97.250.74     <none>         6379/TCP                     91s
```

### List

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ helm list                      
NAME            NAMESPACE       REVISION        UPDATED                                         STATUS          CHART                           APP VERSION
giropops-senhas default         1               2026-03-29 08:52:53.202781041 +0100 WEST        deployed        giropops-senhas-chart-0.1.0     1.0        
```

### Uninstall

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ helm uninstall giropops-senhas                                                                                                                                                                                            1 ↵
release "giropops-senhas" uninstalled
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get pods               
NAME                                   READY   STATUS    RESTARTS      AGE
alpine                                 1/1     Running   2 (11d ago)   11d
minion-deployment-c84f94b49-q62tz      1/1     Running   0             21d
nginx-ssl-deployment-688dcb8fd-d47dj   1/1     Running   8 (49d ago)   73d
nginx-statefulset-0                    1/1     Running   0             21d
nginx-statefulset-1                    1/1     Running   0             21d
nginx-statefulset-2                    1/1     Running   0             21d
```

## Melhorando Values

### values.yaml

**File:** values.yaml

```yaml
giropopsSenhas:
  name: "giropops-senhas-helm"
  image: "linuxtips/giropops-senhas:1.0"
  replicas: 3
  port: 5000
  labels:
    app: "giropops-senhas"
  resources:
    limits:
      memory: "128Mi"
      cpu: "1"
    requests:
      memory: "64Mi"
      cpu: "500m"
  service:
    name: "giropops-senhas"
    type: ClusterIP
    port: 5000
    targetPort: 5000

redis:
  name: "redis-helm"
  image: "redis:8.4"
  replicas: 1
  port: 6379
  labels:
    app: "redis"
  resources:
    limits:
      memory: "128Mi"
      cpu: "500m"
    requests: 
      memory: "64Mi"
      cpu: "250m"
  service:
    name: "redis-service"
    type: "ClusterIP"
    port: 6379
    targetPort: 6379

```
### Templates

**File:** app-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: {{ .Values.giropopsSenhas.labels.app }}
  name: {{ .Values.giropopsSenhas.name }}
spec:
  replicas: {{ .Values.giropopsSenhas.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.giropopsSenhas.labels.app }}
  template:
    metadata:
      labels:
        app: {{ .Values.giropopsSenhas.labels.app }}
    spec:
      containers:
      - image: {{ .Values.giropopsSenhas.image }}
        name: giropops-senhas
        ports:
        - containerPort: {{ .Values.giropopsSenhas.port }}
        resources:
          limits:
            memory: {{ .Values.giropopsSenhas.resources.limits.memory }}
            cpu: {{ .Values.giropopsSenhas.resources.limits.cpu }}
          requests:
            memory: {{ .Values.giropopsSenhas.resources.requests.memory }}
            cpu: {{ .Values.giropopsSenhas.resources.requests.cpu }}
        imagePullPolicy: Always

```

**File:** app-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.giropopsSenhas.service.name }}
  labels:
    app: {{ .Values.giropopsSenhas.labels.app }}
spec:
  selector:
    app: {{ .Values.giropopsSenhas.labels.app }}
  ports:
    - protocol: TCP
      port: {{ .Values.giropopsSenhas.service.port }}
      targetPort: {{ .Values.giropopsSenhas.service.targetPort }}
      name: tcp-app
  type: {{ .Values.giropopsSenhas.service.type }}

```


**File:** redis-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: {{ .Values.redis.labels.app }}
  name: {{ .Values.redis.name }}
spec:
  replicas: {{ .Values.redis.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.redis.labels.app }}
  template:
    metadata:
      labels:
        app: {{ .Values.redis.labels.app }}
    spec:
      containers:
      - image: {{ .Values.redis.image }}
        name: redis
        ports:
          - containerPort: {{ .Values.redis.port }}
        resources:
          limits:
            memory: {{ .Values.redis.resources.limits.memory }}
            cpu: {{ .Values.redis.resources.limits.cpu }}
          requests:
            memory: {{ .Values.redis.resources.requests.memory }}
            cpu: {{ .Values.redis.resources.requests.cpu }}

```


**File:** redis-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.redis.service.name }}
spec:
  selector:
    app: {{ .Values.redis.labels.app }}
  ports:
    - protocol: TCP
      port: {{ .Values.redis.service.port }}
      targetPort: {{ .Values.redis.service.targetPort }}
  type: {{ .Values.redis.service.type }}
```

### Install


```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ helm install giropops-senhas ./
NAME: giropops-senhas
LAST DEPLOYED: Sun Mar 29 09:20:38 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
```

```bash
╭─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS      AGE
alpine                                 1/1     Running   2 (11d ago)   11d
giropops-senhas-helm-999c859cf-bjn7s   1/1     Running   0             10s
giropops-senhas-helm-999c859cf-r7pcr   1/1     Running   0             10s
giropops-senhas-helm-999c859cf-tkvjj   1/1     Running   0             10s
minion-deployment-c84f94b49-q62tz      1/1     Running   0             21d
nginx-ssl-deployment-688dcb8fd-d47dj   1/1     Running   8 (49d ago)   73d
nginx-statefulset-0                    1/1     Running   0             21d
nginx-statefulset-1                    1/1     Running   0             21d
nginx-statefulset-2                    1/1     Running   0             21d
redis-helm-54d4ccfd49-sht4h            1/1     Running   0             10s

```

```bash
─paulo@discovery ~/workspace/linuxtips/DescomplicandoKubernetes/day-19/giropops-chart ‹main●› (⎈|k8s_40:default)
╰─$ helm list                                                                                                                                                                                                               130 ↵
NAME            NAMESPACE       REVISION        UPDATED                                         STATUS          CHART                           APP VERSION
giropops-senhas default         1               2026-03-29 09:20:38.061285363 +0100 WEST        deployed        giropops-senhas-chart-0.1.0     1.0        
```
