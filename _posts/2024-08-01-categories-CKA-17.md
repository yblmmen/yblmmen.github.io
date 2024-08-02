---
title: "[CKA] Secret 운영"
excerpt: "Secret 운영"

categories:
  - CKA
tags:
  - [cka, secret]

permalink: /cka/2024-08-01-categories-CKA-17.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [17] Secret 운영

## ❓Secret을 생성 후 Pod에 전달

- Create a kubernetes secret and expose using a file in the pod.
- 작업 클러스터: k8s

1. Create a kubernetes secret as follows:
    
    
    1. Name: super-secret
    2. DATA:
        
        password=secretpass
2. Create a Pod named `pod-secrets-via-file`, using the `redis` image, which mounts a secret named `super-secret` at `/secrets`
3. Create a second Pod named `pod-secrets-via-env` using the `redis` image which exports password as PASSWORD

### Reference

docs에서 secret검색 → Use Cases 부분

[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

### 실습

```docker
[user@console ~]$ kubectl config use-context k8s

[user@console ~]$ kubectl get secrets

[user@console ~]$ kubectl describe super-secrets

[user@console ~]$ kubectl get secret super-secrets
```
