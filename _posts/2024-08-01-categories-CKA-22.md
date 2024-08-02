---
title: "[CKA]  Kubernetes Upgrade"
excerpt: " Kubernetes Upgrade"

categories:
  - CKA
tags:
  - [cka,  Kubernetes, Upgrade]

permalink: /cka/2024-08-01-categories-CKA-22.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [22] Kubernetes Upgrade

## ❓Cluster Upgrade - only Master

- upgrade system: k8s-master
- Given an existing Kubernetes cluster running version `1.22.4`, upgrade all of the Kubernetes control plane and node components on the master node only to version `1.23.3`
- Be sure to `drain` the `master` node before upgrading it and `uncordon` it after the upgrade.

### Reference

docs에서 upgrade kubeadm 검색

[Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

### 실습

```docker
[user@console ~]$ kubectl config use-context k8s

[user@console ~]$ ssh k8s-master

[user@k8s-master ~]$ sudo -i

# 시험환경은 ubuntu 환경이니까 헷갈려 하지 않기
# Ubuntu
[root@k8s-master ~]# apt-update
[root@k8s-master ~]# apt-cache madison kubeadm

# Ubuntu
[root@k8s-master ~]# apt-mark unhold kubeadm && \\
 apt-get update && apt-get install -y kubeadm=1.23.3-00 && \\
 apt-mark hold kubeadm

# v1.23.3 버전으로 업그레이드 할 수 있는 master component 목록 확인
[root@k8s-master ~]# kubeadm upgrade plan v1.23.3
[root@k8s-master ~]# sudo kubeadm upgrade apply v1.23.3

# 현재 우리는 문제에서 요구한대로 k8s-master로 ssh를 통해 들어와서 작업중인데
# 이 마스터노드를 master노드를 drain 하는 작업을 해야하기 때문에
# exit로 console로 나와서 작업해줘야 한다.
[root@k8s-master ~]# exit
[user@k8s-master ~]$ exit
[user@console ~]$ kubectl drain k8s-master --ignore-daemonsets

# 이제 다시 마스터 노드로 들어간다
[user@console ~]$ ssh k8s-master

# Ubuntu
[user@k8s-master ~]$ sudo apt-mark unhold kubelet kubectl && \\
apt-get update && \\
apt-get install -y kubelet=1.23.3-00 kubectl=1.23.3-00 && \\
apt-mark hold kubelet kubectl

[user@k8s-master ~]$ sudo -i 
[root@k8s-master ~]# systemctl daemon-reload
[root@k8s-master ~]# systemctl restart kubelet

# 이제 다 끝나서 다시 콘솔로 나온 후에 언코던을 해준다
[root@k8s-master ~]# exit
[user@k8s-master ~]$ exit
[user@console ~]$ kubectl uncordon k8s-master
[user@console ~]$ kubectl get nodes

```

## 기출문제

upgrade system : k8s-worker1, k8s-worker2 k8s 클러스터의 모든 worker를 1.23.3 으로 업그레이드 하시오.

- 답안
    
    ```jsx
    # Ubuntu 환경
    kubectl config use-context k8s
    
    ```
    
    ```jsx
    ssh k8s-worker1
    
    sudo -i
    
    ```
    
    ```jsx
    apt-mark unhold kubeadm && \\
    apt-get update && apt-get install -y kubeadm=1.23.3-00 && \\
    apt-mark hold kubeadm
    
    ```
    
    ```jsx
    sudo kubeadm upgrade node
    
    ```
    
    ```jsx
    # console 창으로 이동
    exit
    
    ```
    
    ```jsx
    kubectl drain k8s-worker1 --ignore-daemonsets
    
    ```
    
    ```jsx
    ssh k8s-worker1
    
    ```
    
    ```jsx
    apt-mark unhold kubelet kubectl && \\
    apt-get update && apt-get install -y kubelet=1.23.3-00 kubectl=1.23.3-00 && \\
    apt-mark hold kubelet kubectl
    
    ```
    
    ```jsx
    sudo systemctl daemon-reload
    
    sudo systemctl restart kubelet
    
    ```
    
    ```jsx
    # console 창으로 이동
    exit
    
    ```
    
    ```jsx
    kubectl uncordon k8s-worker1
    --> k8s-worker2도 동일하게 진행
    
    ```
    
    ```jsx
    kubectl get nodes
    --> 노드 버전 확인
    
    ```

## 기출문제 (25번)

Given an existing Kubernetes cluster running version 1.20.0, upgrade all of the Kubernetes control plane and node components on the master node only to version 1.20.1. Be sure to drain the master node before upgrading it and uncordon it after the upgrade.

You are also expected to upgrade kubelet and kubectl on the master node.

- 답안  
    ```jsx
    kubectl get nodes
    --> node 이름 확인
    
    ```
    
    ```jsx
    ssh {Node 이름}
    
    sudo -i 
    
    ```
    
    ```jsx
    # Ubuntu 버전
    apt update
    
    ```
    
    ```jsx
    apt-cache madison kubeadm
    
    ```
    
    ```jsx
    apt-mark unhold kubeadm && \\
     apt-get update && apt-get install -y kubeadm=1.20.1-00 && \\
     apt-mark hold kubeadm
    
    ```
    
    ```jsx
    kubeadm version
    
    ```
    
    ```jsx
    kubeadm upgrade plan
    
    ```
    
    ```jsx
    sudo kubeadm upgrade apply v1.20.1
    
    ```
    
    ```jsx
    # console 창으로 이동
    exit
    
    ```
    
    ```jsx
    kubectl drain <node-to-drain> --ignore-daemonsets
    
    ```
    
    ```jsx
    ssh {Node 이름}
    
    ```
    
    ```jsx
    apt-mark unhold kubelet kubectl && \\
    apt-get update && apt-get install -y kubelet=1.20.1-00 kubectl=1.20.1-00 && \\
    apt-mark hold kubelet kubectl
    
    ```
    
    ```jsx
    sudo systemctl daemon-reload
    
    sudo systemctl restart kubelet
    
    ```
    
    ```jsx
    # console 창으로 이동
    exit
    
    ```
    
    ```jsx
    kubectl uncordon <node-to-drain>
    
    ```
    
    ```jsx
    # 버전 확인
    kubectl get nodes
    -> 업그레이드 한 Node 버전 확인
    
    ```
