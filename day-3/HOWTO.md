# K8S Deployment

## Manifest

*Exemplo manifesto como criar um deployment*

  
  k8s-nginx-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  # Pod template specification
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```bash
$ kubectl apply -f k8s-nginx-deployment.yaml
deployment.apps/nginx-deployment configured
```

*Visualizar manifesto do Deployment instalado*

```bash
$ kubectl get deployments.apps nginx-deployment -o yaml
```

*Obter lista de pods com label app=nginx*

```bash
$ kubectl get pods -l app=nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5995856f4f-2whwk   1/1     Running   0          4m29s
nginx-deployment-5995856f4f-tsmsm   1/1     Running   0          4m29s
nginx-deployment-5995856f4f-w8tzb   1/1     Running   0          4m29s
```

# Criar 'template' de manifesto

```bash
$ kubectl create deployment nginx-deployment --image nginx:latest --replicas=4 --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        resources: {}
status: {}
```

## Obter replicaset

```bash
$ kubectl get replicasets.apps
NAME                              DESIRED   CURRENT   READY   AGE
nginx-deployment-5995856f4f       3         3         3       21m
nginx-tpc-deployment-5865b9fc78   3         3         3       4d2h
```

## Create Namespace

```bash
$ kubectl create namespace giropops --dry-run=client -o yaml > k8s-namespace-giropops.yaml

$ kubectl apply -f k8s-namespace-giropops.yaml
namespace/giropops created
```

## ROLLOUT

Fazer rollout
```bash
$ kubectl rollout status deployment -n giropops nginx-deployment
deployment "nginx-deployment" successfully rolled out
```

Atualizar algo no manifesto de deployment

```bash

$ kubectl apply -f k8s-deployment-giropops-strategy.yaml
deployment.apps/nginx-deployment configured

$  kubectl rollout status deployment -n giropops nginx-deployment
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 5 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 6 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 7 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 8 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 8 of 10 updated replicas are available...
Waiting for deployment "nginx-deployment" rollout to finish: 9 of 10 updated replicas are available...
deployment "nginx-deployment" successfully rolled out
```

## Rollback

```bash
$ kubectl rollout undo -n giropops deployment nginx-deployment
deployment.apps/nginx-deployment rolled back
```

## Rollback History

```bash
ubectl rollout history -n giropops deployment nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
4         <none>
5         <none>
```

*Consultar uma revisão específica*

```bash
$ kubectl rollout history -n giropops deployment nginx-deployment --revision 6
deployment.apps/nginx-deployment with revision #6
Pod Template:
  Labels:       app=nginx-deployment
        pod-template-hash=9d9d65669
  Containers:
   nginx:
    Image:      nginx:1.20
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors:       <none>
  Tolerations:  <none>
```

## Rollout Restart Pods

```bash
$ kubectl rollout restart deployment -n giropops nginx-deployment
deployment.apps/nginx-deployment restarted
```

> [!TIP]
> Usando scale

```bash
$ kubectl scale deployment -n giropops --replicas 0 nginx-deployment
deployment.apps/nginx-deployment scaled

$ kubectl scale deployment -n giropops --replicas 10 nginx-deployment
deployment.apps/nginx-deployment scaled
```