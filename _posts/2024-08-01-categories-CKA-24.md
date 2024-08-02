---
title: "[CKA] Kubernetes Troubleshooting-2"
excerpt: "Kubernetes Troubleshooting-2"

categories:
  - CKA
tags:
  - [cka,  Kubernetes, notready]

permalink: /cka/2024-08-01-categories-CKA-24.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [24] Kubernetes Troubleshooting (2)

## ❓Not Ready 상태의 노드 활성화

- A Kubernetes worker node, named hk8s-w2 is in state NotReady.
- Investigate why this is the case, and perform any appropriate steps to bring the node to a `Ready` state, ensuring that any changes are made permanent.

### 실습

```docker
[user@console ~]$ kubectl config use-context hk8s

[user@console ~]$ kubectl get nodes
hk8s-w2가 Not Ready상태임

[user@console ~]$ ssh hk8s-w2

[user@hk8s-w2 ~]$ sudo -i

# docker가 running중인지 확인
[root@hk8s-w2 ~]# systemctl status docker
inactive상태뿐만 아니라 disable 상태

[root@hk8s-w2 ~]# systemctl enable --now docker

[root@hk8s-w2 ~]# systemctl status docker
active상태, enable 상태

```
