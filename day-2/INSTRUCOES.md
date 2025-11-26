## Desafio Aula 2

### Parte 1:

Criar namespace treinamento-ch2

Ficheiro k8s-treinamento-namespace.yaml

```YAML
apiVersion: v1
kind: Namespace
metadata:
  name: treinamento-ch2
```

```bash
kubectl apply -f k8s-treinamento-namespace.yaml
```

### Parte 2: O Teste de Estresse (O Erro Esperado)

Ficheiro k8s-podfaminto.yaml

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: pod-faminto
  namespace: treinamento-ch2
spec:
  containers:
    - name: stress-container
      image: polinux/stress
      command: ["stress", "--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
      resources:
        requests:
          memory: "100Mi"
        limits:
          memory: "200Mi"
```

```bash
kubectl apply -f k8s-podfaminto.yaml
```

```bash
kubectl describe pod pod-faminto -n treinamento-ch2

Name:             pod-faminto
Namespace:        treinamento-ch2
Priority:         0
Service Account:  default
Node:             aks-agentpool-31830996-vmss000000/172.26.20.14
Start Time:       Wed, 26 Nov 2025 11:35:57 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.13
IPs:
  IP:  10.244.0.13
Containers:
  stress-container:
    Container ID:  containerd://143f267ca7ec96e1604fdc98e56f2ebfff3d8e8da80eac2c1ce6c5ec005b0fe8
    Image:         polinux/stress
    Image ID:      docker.io/polinux/stress@sha256:b6144f84f9c15dac80deb48d3a646b55c7043ab1d83ea0a697c09097aaad21aa
    Port:          <none>
    Host Port:     <none>
    Command:
      stress
      --vm
      1
      --vm-bytes
      250M
      --vm-hang
      1
    State:          Terminated
      Reason:       OOMKilled
      Exit Code:    137
      Started:      Wed, 26 Nov 2025 11:39:09 +0000
      Finished:     Wed, 26 Nov 2025 11:39:09 +0000
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    137
      Started:      Wed, 26 Nov 2025 11:37:34 +0000
      Finished:     Wed, 26 Nov 2025 11:37:34 +0000
    Ready:          False
    Restart Count:  5
    Limits:
      memory:  200Mi
    Requests:
      memory:     100Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-92xf2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-92xf2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/memory-pressure:NoSchedule op=Exists
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  3m14s                default-scheduler  Successfully assigned treinamento-ch2/pod-faminto to aks-agentpool-31830996-vmss000000
  Normal   Pulled     3m12s                kubelet            Successfully pulled image "polinux/stress" in 1.817s (1.817s including waiting). Image size: 4041495 bytes.
  Normal   Pulled     3m11s                kubelet            Successfully pulled image "polinux/stress" in 583ms (583ms including waiting). Image size: 4041495 bytes.
  Normal   Pulled     2m56s                kubelet            Successfully pulled image "polinux/stress" in 611ms (611ms including waiting). Image size: 4041495 bytes.
  Normal   Pulled     2m26s                kubelet            Successfully pulled image "polinux/stress" in 607ms (607ms including waiting). Image size: 4041495 bytes.
  Normal   Pulled     97s                  kubelet            Successfully pulled image "polinux/stress" in 593ms (593ms including waiting). Image size: 4041495 bytes.
  Normal   Pulling    3s (x6 over 3m14s)   kubelet            Pulling image "polinux/stress"
  Normal   Created    2s (x6 over 3m12s)   kubelet            Created container: stress-container
  Normal   Started    2s (x6 over 3m12s)   kubelet            Started container stress-container
  Normal   Pulled     2s                   kubelet            Successfully pulled image "polinux/stress" in 610ms (610ms including waiting). Image size: 4041495 bytes.
  Warning  BackOff    1s (x16 over 3m10s)  kubelet            Back-off restarting failed container stress-container in pod pod-faminto_treinamento-ch2(a9984cf4-f821-4c63-a055-5d712084c1f8)
```