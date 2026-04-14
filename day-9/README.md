# Instalação do NGINX Ingress Controller (MetalLB)

Este guia detalha a instalação do NGINX Ingress Controller em clusters Kubernetes com MetalLB.

Referência: https://kubernetes.github.io/ingress-nginx/deploy/#quick-start

## 1. Instalação via Manifestos (YAML)

Para ambientes bare metal com MetalLB, o projeto oficial fornece um manifesto específico que configura o serviço do controller como `LoadBalancer`.

```bash
# Aplicar o manifesto oficial para Bare Metal
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.14.1/deploy/static/provider/cloud/deploy.yaml

# Verificar o status dos pods
kubectl get pods -n ingress-nginx -w

# Verificar as portas NodePort alocadas
kubectl get svc -n ingress-nginx
```

## 2. Instalação via Helm

O Helm é a forma recomendada para gerenciar o ciclo de vida do controller.

```bash
# Adicionar o repositório oficial
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Instalação básica para Bare Metal (usando LoadBalancer)
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer
```

### Notas para Bare Metal:

* **NodePort:** Por padrão, o Kubernetes alocará portas no intervalo 30000-32767.
* **HostNetwork:** Se precisar que o Ingress escute diretamente nas portas 80/443 dos nós, adicione `--set controller.hostNetwork=true` ao comando Helm.
* **MetalLB:** Se o seu cluster possui MetalLB configurado, você pode alterar o `service.type` para `LoadBalancer`.

## 3. Verificar a Instalação

Certifique-se de que os Pods estão em execução:

```bash
kubectl get pods -n ingress-nginx
```

Verifique as portas mapeadas (especialmente se estiver usando NodePort):

```bash
kubectl get svc -n ingress-nginx
```

## 4. Remover o NGINX Ingress Controller

```bash
kubectl delete all  --all -n ingress-nginx
```