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
> Login to to master node and set hostname via hostnamectl command
```
sudo hostnamectl set-hostname "master1"
exec bash
```
> On the worker nodes, run
```
sudo hostnamectl set-hostname "worker1"   // 1st worker node
sudo hostnamectl set-hostname "worker2"   // 1st worker node
exec bash
```
> Add the following lines in /etc/hosts file on each node
```
192.168.100.62  master1.innerinfo.io master1
192.168.100.71  worker1.innerinfo.io worker1
192.168.100.72  worker2.innerinfo.io worker2
```

### 2) Disable Swap & Add kernel Parameters
> Execute beneath swapoff and sed command to disable swap. Make sure to run the following commands on all the nodes
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

> Load the following kernel modules on all the nodes
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

> Set the following Kernel parameters for Kubernetes, run beneath tee command
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOT
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOT
```

> Reload the above changes, run
``
sudo sysctl --system
```

### 3) Install Containerd Runtime
> In this guide, we are using containerd runtime for our Kubernetes cluster. So, to install containerd, first install its dependencies
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```











