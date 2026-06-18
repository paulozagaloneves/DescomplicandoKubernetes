# Instalação do Cluster Kubernetes no Proxmox (PVE)

Este guia detalha os passos executados para configurar o nó de control plane de um cluster Kubernetes em uma VM no Proxmox, conforme o histórico de comandos utilizados.

## 1. Pré-requisitos

- VM criada no Proxmox (Debian/Ubuntu)
- Acesso root ou sudo
- Rede configurada

## 2. Preparação do Sistema

### Instale o agente QEMU (opcional, mas recomendado para integração com Proxmox):

```bash
sudo apt install qemu-guest-agent -y
systemctl status qemu-guest-agent
```

### Desabilite o swap:

```bash
sudo swapoff -a
```

Para desabilitar permanentemente, remova ou comente a linha do swap em `/etc/fstab`.

### Ative módulos do kernel necessários:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

### Ajuste parâmetros do sysctl:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```

## 3. Atualize o sistema

```bash
sudo apt update && sudo apt dist-upgrade -y
```

## 4. Instale dependências

```bash
sudo apt install -y apt-transport-https ca-certificates curl gpg
```

## 5. Adicione o repositório do Kubernetes

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
```

## 6. Instale kubelet, kubeadm e kubectl

```bash
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## 7. Instale e configure o containerd

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

```bash
sudo apt install -y containerd.io
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable --now kubelet
```

## 8. Inicialize o cluster Kubernetes

**control-plane**

> Altere o endereço IP conforme o IP da sua VM.

```bash
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=192.168.1.60
```

**workers**
```bash
sudo kubeadm join 192.168.1.60:6443 --token f95556.a534uxzas51vqf7n        --discovery-token-ca-cert-hash sha256:e03be7c916b0b0f229dcf827e5df63040eebc603f7caaea43e67bfaba7c2c026
```

## 9. Configure o acesso kubectl para o usuário

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 10. Instale o Cilium como CNI

```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}

# Instale o Cilium no cluster
cilium install
cilium status --wait
```

## 11. Instale e habilite o Hubble (Observabilidade do Cilium)

O Hubble é a ferramenta de observabilidade do Cilium. Siga os passos abaixo para instalar e habilitar:

```bash
# Habilite o Hubble no Cilium
cilium hubble enable

# (Opcional) Habilite a interface web do Hubble
cilium hubble enable --ui

# Instale o binário do Hubble CLI
HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/master/stable.txt)
HUBBLE_ARCH=amd64
curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
sha256sum --check hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
sudo tar xzvfC hubble-linux-${HUBBLE_ARCH}.tar.gz /usr/local/bin
rm hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}

# Verifique o status do Hubble
hubble status -P

# Observe o tráfego em tempo real
hubble observe -P

# Verifique o pod do relay do Hubble
kubectl -n kube-system get pods -l k8s-app=hubble-relay
```

## 12. Verifique o cluster

```bash
kubectl get nodes
```

---

**Referência:**
Arquivo `history.txt` do nó control plane.
