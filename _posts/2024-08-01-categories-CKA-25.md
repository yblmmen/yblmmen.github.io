---
title: "[CKA] User Role Binding"
excerpt: "User Role Binding"

categories:
  - CKA
tags:
  - [cka, User, Role, Binding]

permalink: /cka/2024-08-01-categories-CKA-25.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [25] User Role Binding

## ❓Configuring User API Authentication

- TASK: 
    - Create the kubeconfig named `ckauser`
        - username: `ckauser`
        - `ckauser` cluster must be operated with the privileges of the `ckauser` account
        - certificate location: /data/cka/ckauser.crt, /data/cka/ckauser.key
        - context-name: `ckauser`
    - Create a role named `pod-role` that can `create, delete, watch, list, get` pods.
    - Create the following rolebinding: 
        - name: `pod-rolebinding`
        - role: `pod-role`
        - user: `ckauser`
- 작업 클러스터: k8s

### Reference

[Certificate Signing Requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)

[Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

### 실습

```docker
[user@console ~]$ ssh k8s-master

# pod-role이라는 이름의 role 생성
[user@k8s-master ~]$ kubectl create role pod-role --verb=create,delete,watch,list,get --resource=pods
[user@k8s-master ~]$ kubectl get role pod-role
[user@k8s-master ~]$ kubectl describe role pod-role

# role-binding 생성
[user@k8s-master ~]$ kubectl create rolebinding pod-rolebinding --role=pod-role --user=ckauser
[user@k8s-master ~]$ kubectl get rolebinding pod-rolebinding
[user@k8s-master ~]$ kubectl describe rolebinding pod-rolebinding

# context안에 user를 넣어줘야함
[user@k8s-master ~]$ kubectl config set-credentials ckauser --client-key=/data/cka/ckauser.key --client-certificate=/data/cka/ckauser.crt --embed-certs=true

# ckauser가 인증서 기반으로 생성된것 확인 가능
[user@k8s-master ~]$ kubectl config view
[user@k8s-master ~]$ kubectl config set-context ckauser --cluster=kubernetes --user=ckauser

# 테스트 해보기
[user@k8s-master ~]$ kubectl config use-context ckauser
[user@k8s-master ~]$ kubectl get pods

[user@k8s-master ~]$ kubectl get service
--> service에 대한 권한이 없기 때문에 불가능

# 원래 클러스터로 복귀
[user@k8s-master ~]$ kubectl config use-context kubernetes-admin@kubernetes

```
