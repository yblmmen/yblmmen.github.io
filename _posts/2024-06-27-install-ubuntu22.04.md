---
title: "[쿠버네티스]-Install Kubernetes Cluster on Ubuntu 22.04"
excerpt: "Install Kubernetes Cluster on Ubuntu 22.04"

categories:
  - Kubernetes
tags:
  - [kubernetes]

permalink: /kubernetes/install-ubuntu22.04/

toc: true
toc_sticky: true

date: 2024-06-27
last_modified_at: 2024-06-27
---

## How to Install Kubernetes Cluster on Ubuntu 22.04

```
Table of Contents
Prerequisites
Lab Setup
1) Set hostname on Each Node
2) Disable Swap & Add kernel Parameters
3) Install Containerd Runtime
4) Add Apt Repository for Kubernetes
5) Install Kubectl, Kubeadm and Kubelet
6) Install Kubernetes Cluster on Ubuntu 22.04
7) Join Worker Nodes to the Cluster
8) Install Calico Network Plugin
9) Test Your Kubernetes Cluster Installation
```

### 1) Set hostname on Each Node

Login to to master node and set hostname via hostnamectl command

```
sudo hostnamectl set-hostname "master1"
exec bash
```

On the worker nodes, run
```
sudo hostnamectl set-hostname "worker1"   // 1st worker node
sudo hostnamectl set-hostname "worker2"   // 1st worker node
exec bash
```

Add the following lines in /etc/hosts file on each node
```
192.168.100.62  master1.innerinfo.io master1
192.168.100.71  worker1.innerinfo.io worker1
192.168.100.72  worker2.innerinfo.io worker2
```

### 2) Disable Swap & Add kernel Parameters

Execute beneath swapoff and sed command to disable swap. Make sure to run the following commands on all the nodes
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Load the following kernel modules on all the nodes
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

Set the following Kernel parameters for Kubernetes, run beneath tee command
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOT
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOT
```

Reload the above changes, run
``
sudo sysctl --system
```

### 3) Install Containerd Runtime

In this guide, we are using containerd runtime for our Kubernetes cluster. So, to install containerd, first install its dependencies
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

Enable docker repository
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

ow, run following apt command to install containerd
```
sudo apt update
sudo apt install -y containerd.io
```

Configure containerd so that it starts using systemd as cgroup.
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

Restart and enable containerd service
```
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 4) Add Apt Repository for Kubernetes

Kubernetes package is not available in the default Ubuntu 22.04 package repositories. So we need to add Kubernetes repository. run following command to download public signing key,
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Next, run following echo command to add Kubernetes apt repository.
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 5) Install Kubectl, Kubeadm and Kubelet

Post adding the repositories, install Kubernetes components like kubectl, kubelet and Kubeadm utility on all the nodes. Execute following set of commands,
```
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### 6) Install Kubernetes Cluster on Ubuntu 22.04

Now, we are all set to initialize Kubernetes cluster. Run the following Kubeadm command on the master node only.
```
kubeadm init --apiserver-advertise-address=192.168.100.62 --pod-network-cidr=10.244.0.0/16
```
Output of above command,

### 6-1) Cluster 초기화
```
// 마스터, 워커노드 공통
sudo kubeadm reset
sudo rm -r /etc/cni/net.d/*
sudo rm -r ~/.kube/config
sudo systemctl restart kubelet
```

After the initialization is complete, you will see a message with instructions on how to join worker nodes to the cluster. Make a note of the kubeadm join command for future reference.

So, to start interacting with cluster, run following commands on the master node,
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

next, try to run following kubectl commands to view cluster and node status
```
kubectl cluster-info
kubectl get nodes
```





### 7) Join Worker Nodes to the Cluster

On each worker node, use the kubeadm join command you noted down earlier after initializing the master node on step 6. It should look something like this:
```
sudo kubeadm join k8smaster.example.net:6443 --token vt4ua6.wcma2y8pl4menxh2 \
   --discovery-token-ca-cert-hash sha256:0494aa7fc6ced8f8e7b20137ec0c5d2699dc5f8e616656932ff9173c94962a36
```

Output from both the worker nodes,

Above output from worker nodes confirms that both the nodes have joined the cluster.Check the nodes status from master node using kubectl command,
```
kubectl get nodes
```

As we can see nodes status is ‘NotReady’, so to make it active. We must install CNI (Container Network Interface) or network add-on plugins like Calico, Flannel and Weave-net.

### 8) Install Calico Network Plugin

A network plugin is required to enable communication between pods in the cluster. Run following kubectl command to install Calico network plugin from the master node,
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml  // 대체 URL
```

Verify the status of pods in kube-system namespace,
```
kubectl get pods -n kube-system
```

Perfect, check the nodes status as well.
```
kubectl get nodes
```
Great, above confirms that nodes are active node. Now, we can say that our Kubernetes cluster is functional.

### 












