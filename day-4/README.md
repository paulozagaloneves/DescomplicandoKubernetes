# Aula 4

## Criar um deployment da aplicação nginx

**Nome do deployment:** nginx-deployment
**Imagem:** nginx
**Réplicas:** 1

**Arquivo:** `k8s-nginx-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx-deployment
  labels:
    app:  nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-deployment
  replicas: 1
  strategy: {}
  template:
    metadata:
      labels:
        app:  nginx-deployment
    spec:
      containers:
      - name:  nginx
        image:  nginx:latest
        resources:
          requests:
            cpu: "0.25"
            memory: 50Mi
          limits:
            cpu: "0.5"
            memory: 100Mi
        ports:
        - containerPort:  80
          name: http
      restartPolicy: Always
```

```bash
$ kubectl apply -f k8s-nginx-deployment.yaml
 deployment.apps/nginx-deployment created
```

**Obter lista de deployments**

```bash
$ kubectl get deployments.apps
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment       1/1     1            1           11m
nginx-tpc-deployment   3/3     3            3           7d15h
```

**Obter replicasets**

```bash
$ kubectl get replicasets.apps              
NAME                              DESIRED   CURRENT   READY   AGE
nginx-deployment-584b7b986        1         1         1       11s
nginx-tpc-deployment-5865b9fc78   3         3         3       7d15h
```

## Aumentando número de réplicas

**Aumentando o número de réplicas**

```bash
$ kubectl scale deployment nginx-deployment --replicas=3
deployment.apps/nginx-deployment scaled
```

```bash
$ kubectl get replicasets.apps                          
NAME                              DESIRED   CURRENT   READY   AGE
nginx-deployment-584b7b986        3         3         3       3m22s
nginx-tpc-deployment-5865b9fc78   3         3         3       7d15h
```

> [!NOTE]
> Manteve o mesmo replicaset


**Usando yaml manifest**

**Arquivo:** `k8s-nginx-deployment-3replicas.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx-deployment
  labels:
    app:  nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-deployment
  replicas: 3
  strategy: {}
  template:
    metadata:
      labels:
        app:  nginx-deployment
    spec:
      containers:
      - name:  nginx
        image:  nginx:latest
        resources:
          requests:
            cpu: "0.25"
            memory: 50Mi
          limits:
            cpu: "0.5"
            memory: 100Mi
        ports:
        - containerPort:  80
          name: http
      restartPolicy: Always
```

```bash
$ kubectl apply -f k8s-nginx-deployment-3replicas.yaml 
deployment.apps/nginx-deployment configured
```

> [!NOTE]
> Manteve o mesmo replicaset porque apenas mudaram o número de réplicas


**Arquivo:** `k8s-nginx-deployment-3replicas-novaversao.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx-deployment
  labels:
    app:  nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx-deployment
  replicas: 3
  strategy: {}
  template:
    metadata:
      labels:
        app:  nginx-deployment
    spec:
      containers:
      - name:  nginx
        image:  nginx:1.19.2
        resources:
          requests:
            cpu: "0.25"
            memory: 50Mi
          limits:
            cpu: "0.5"
            memory: 100Mi
        ports:
        - containerPort:  80
          name: http
      restartPolicy: Always
```

```bash
$ kubectl apply -f k8s-nginx-deployment-3replicas-novaversao.yaml 
deployment.apps/nginx-deployment configured
```

** Visualizar de novo os replicasets**

```bash
$ kubectl get replicasets.apps                                   
NAME                              DESIRED   CURRENT   READY   AGE
nginx-deployment-584b7b986        0         0         0       15m
nginx-deployment-5b8d6f9568       3         3         3       2m34s
nginx-tpc-deployment-5865b9fc78   3         3         3       7d15h
```

> [!NOTE]
> Antigo replicaset nginx-deployment-584b7b986 ficou com 0 pods
> Criado novo replicaset nginx-deployment-5b8d6f9568 com 3 pods

## Observando detalhes do deployment

```bash
$ kubectl describe deployment nginx-deployment  
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sat, 06 Dec 2025 09:19:56 +0000
Labels:                 app=nginx-deployment
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=nginx-deployment
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-deployment
  Containers:
   nginx:
    Image:      nginx:1.19.2
    Port:       80/TCP (http)
    Host Port:  0/TCP (http)
    Limits:
      cpu:     500m
      memory:  100Mi
    Requests:
      cpu:         250m
      memory:      50Mi
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  nginx-deployment-584b7b986 (0/0 replicas created)
NewReplicaSet:   nginx-deployment-5b8d6f9568 (3/3 replicas created)
Events:
  Type    Reason             Age                From                   Message
  ----    ------             ----               ----                   -------
  Normal  ScalingReplicaSet  19m                deployment-controller  Scaled up replica set nginx-deployment-584b7b986 from 0 to 1
  Normal  ScalingReplicaSet  15m                deployment-controller  Scaled down replica set nginx-deployment-584b7b986 from 3 to 1
  Normal  ScalingReplicaSet  14m (x2 over 17m)  deployment-controller  Scaled up replica set nginx-deployment-584b7b986 from 1 to 3
  Normal  ScalingReplicaSet  6m38s              deployment-controller  Scaled up replica set nginx-deployment-5b8d6f9568 from 0 to 1
  Normal  ScalingReplicaSet  6m32s              deployment-controller  Scaled down replica set nginx-deployment-584b7b986 from 3 to 2
  Normal  ScalingReplicaSet  6m32s              deployment-controller  Scaled up replica set nginx-deployment-5b8d6f9568 from 1 to 2
  Normal  ScalingReplicaSet  6m26s              deployment-controller  Scaled down replica set nginx-deployment-584b7b986 from 2 to 1
  Normal  ScalingReplicaSet  6m26s              deployment-controller  Scaled up replica set nginx-deployment-5b8d6f9568 from 2 to 3
  Normal  ScalingReplicaSet  6m21s              deployment-controller  Scaled down replica set nginx-deployment-584b7b986 from 1 to 0
```

> [!NOTE]
> Conditions:
> 
> Type           Status  Reason
> 
>  ----           ------  ------
> 
>  Available      True    MinimumReplicasAvailable
> 
>  Progressing    True    NewReplicaSetAvailable
> 
> OldReplicaSets:  nginx-deployment-584b7b986 (0/0 replicas created)
> 
> NewReplicaSet:   nginx-deployment-5b8d6f9568 (3/3 replicas created)
> 


## Rollback com rollout

```bash
$ kubectl rollout undo deployment nginx-deployment
deployment.apps/nginx-deployment rolled back

$ kubectl get replicasets.apps                    
NAME                              DESIRED   CURRENT   READY   AGE
nginx-deployment-584b7b986        3         3         3       25m
nginx-deployment-5b8d6f9568       0         0         0       12m
nginx-tpc-deployment-5865b9fc78   3         3         3       7d15h
```

> [!NOTE]
> O replicatset "novo" nginx-deployment-5b8d6f9568 ficou esvaziado.
> O replicatset "anterior" nginx-deployment-5b8d6f9568 ficou com 3 pods.