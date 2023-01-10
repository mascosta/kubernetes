# kubernetes
Repositório para consulta de instalação do ambiente.

## Esse guia tem por objetivo instalar o kubernetes, na versão 1.25.X, em um host RHEL like.

## 1 - Instalar pacotes necessários/úteis.


```bash
dnf install epel-release -y && \
dnf install htop vim wget curl \
yum-utils tmux bash-completion -y
```
## 2 - Desabilitar a partição ```swap```.

```bash
# Desabilitando a swap dentro do fstab, também é possível só comentando a linha pertinente

sed -i '/ swap / s/^/#/' /etc/fstab && \

# Desabilitando em tempo real

swapoff -a
```

## 3 - Dando aquela "emperequetada" no VIM :D

```bash

vim /etc/profile.d/vimrc

```

```vim
set autoindent
set smartindent
set number
set ignorecase
set ts=4
set sw=4
set incsearch
set hlsearch
set history=1000
set expandtab
filetype on
filetype plugin on
filetype indent on
syntax on
colorscheme desert
```

## 4 - Adicionando repositórios 

### 4.1 - Kubernetes

```bash
vim /etc/yum.repos.d/kubernetes.repo
```

```vim

[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg

```

### 4.2 - Docker (para o containerd) 

```bash

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

```

## 5 - Instalação do ```containerd```.

```bash

dnf install -y containerd.io --allowerasing 

```

## 6 - Habilitando módulos ```overlay``` e ```br_netfilter``` 

### 6.1 - Em tempo real

```bash

modprobe overlay && \ 
modprobe br_netfilter

```

### 6.2 - Permanentemente

```bash

cat << EOF >> /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

```

## 7 - Configurando  parâmetros ```kubernetes-cri```.

```bash

cat << EOF > /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

```

## 8 - Recarregando configurações pertinentes.

```bash

sysctl --system &&  sysctl -p

```

## 9 - Criando configuração do ```containerd```.

```bash

mkdir -p /etc/containerd && \
containerd config default > /etc/containerd/config.toml && \
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

```

## 10 - Habilitando serviço containerd.

```bash

systemctl enable --now containerd

```

## 11 - Instalando o ```kubernetes```, na versão ```1.25.5```.

```bash

dnf install kubeadm-1.23.9 cri-tools-1.23.0 kubelet-1.23.9 kubectl-1.23.9 socat conntrack-tools -y

```


## 12 - Ativando o serviço do ```kubelet```.

```bash

systemctl enable --now kubelet

```

## 13 - Ativando o auto completion do ```kubeadm``` e ```kubectl```.

```bash

source <(kubectl completion bash) && \
source <(kubeadm completion bash)
cat << EOF >> ~/.bashrc
source <(kubectl completion bash) 
source <(kubeadm completion bash)
EOF

```
