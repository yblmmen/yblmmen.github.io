---
title: "[CKA] Kubernetes Troubleshooting"
excerpt: "Kubernetes Troubleshooting"

categories:
  - CKA
tags:
  - [cka,  Kubernetes, Troubleshooting]

permalink: /cka/2024-08-01-categories-CKA-23.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [23] Kubernetes Troubleshooting (1)

## ❓Not Ready 상태의 노드 활성화

- A Kubernetes worker node, named hk8s-w2 is in state NotReady.
- Investigate why this is the case, and perform any appropriate steps to bring the node to a `Ready` state, ensuring that any changes are made permanent.

## 확인해야 할 것

- docker가 실행중인지?
- kubelet이 실행중인지?
- kube-proxy가 실행중인지?
- CNI가 실행중인지? (컨테이너와 컨테이너 사이에 통신이 가능한지)

—&gt; 이 모든 것이 실행중이여야 컨테이너가 Ready상태가 된다.

### 실습

```docker
[user@console ~]$ kubectl get nodes
hk8s-w2가 Not Ready상태임

[user@console ~]$ ssh hk8s-w2

[user@hk8s-w2 ~]$ sudo -i

# docker가 running중인지 확인
[root@hk8s-w2 ~]# systemctl status docker
active상태임

# kubelet이 running중인지 확인
[root@hk8s-w2 ~]# systemctl status kubelet
inactive상태 뿐만 아니라 disable 상태

# kubelet 실행 (--now 옵션은 지금 당장 시작하고 다음 시작때에도 시작)
[root@hk8s-w2 ~]# systemctl enable --now kubelet

[root@hk8s-w2 ~]# systemctl status kubelet
active 상태 확인

# kube-proxy가 running중인지 확인
[root@hk8s-w2 ~]# exit

[user@hk8s-w2 ~]$ exit

# kube-proxy와 CNI는 console에서 확인
[user@console ~]$ kubectl get pods -n kube-system -o wide
calico라고 나와있는 것이 CNI이다.
kube-proxy도 running중
CNI 종류로는 calico, flannel 등이 있다.

```

## 기출문제 (5번)

Given a partially-functioning Kubernetes cluster, identify symptoms of failure on the cluster. Determine the node, the failing service, and take actions to bring up the failed service and restore the health of the cluster. Ensure that any changes are made permanently.

- 답안
    
    ```jsx
    # 확인해야 할 것 4개
    # 1. docker가 실행중인지
    --> ssh로 노드 접속한 다음 systemctl status docker로 확인
    --> active상태가 아니면 systemctl start docker, systemctl enable docker
    
    # 2. kubelet이 실행중인지
    --> ssh로 노드 접속한 다음 systemctl status kubelet
    --> active상태가 아니라면 systemctl start kubelet, systemctl enable kubelet
    
    # 3. kube-proxy가 실행중인지
    # 4. CNI가 실행중인지
    --> console에서 실행
    --> kubectl get pods -n kube-system -o wide
    
    ```

## 기출문제 (32번)

A Kubernetes worker node, named wk8s-node-0 is in state NotReady. Investigate why this is the case, and perform any appropriate steps to bring the node to a Ready state, ensuring that any changes are made permanent.

- 답안
    
    ```jsx
    # 확인해야 할 것 4개
    # 1. docker가 실행중인지
    --> ssh로 노드 접속한 다음 systemctl status docker로 확인
    --> active상태가 아니면 systemctl start docker, systemctl enable docker
    
    # 2. kubelet이 실행중인지
    --> ssh로 노드 접속한 다음 systemctl status kubelet
    --> active상태가 아니라면 systemctl start kubelet, systemctl enable kubelet
    
    # 3. kube-proxy가 실행중인지
    # 4. CNI가 실행중인지
    --> console에서 실행
    --> kubectl get pods -n kube-system -o wide
    
    ```
