---
title: "[CKA] ServiceAccount Role Binding"
excerpt: "ServiceAccount Role Binding"

categories:
  - CKA
tags:
  - [cka, ServiceAccount, Role, Binding]

permalink: /cka/2024-08-01-categories-CKA-27.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [27] ServiceAccount Role Binding

## ❓Service Account, Role and Role Binding

- Create the ServiceAccount named `pod-access` in a new namespace called `apps`
- Create a Role with the name `pod-role` and the RoleBinding named `pod-rolebinding`
- Map the ServiceAccount from the previous step to the API resources `Pods` with the operations `watch, list, get`
- 작업 클러스터: k8s

### Reference

[Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#kubectl-create-role)

[Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

### 실습

```docker
# apps 이라는 namespace가 있는지 확인
[user@console ~]$ kubectl get namespace apps

# 없으면 namespace 생성
[user@console ~]$ kubectl create namespace apps

# service account 생성 및 확인
[user@console ~]$ kubectl create serviceaccount pod-access -n apps
[user@console ~]$ kubectl get serviceaccount -n apps

# apps namespace에 role 생성 및 확인
[user@console ~]$ kubectl create role pod-role --verb=watch,list,get --resource=pods -n apps
[user@console ~]$ kubectl get role -n apps
[user@console ~]$ kubectl describe role -n apps pod-role

# role binding 생성 및 확인
[user@console ~]$ kubectl create rolebinding pod-rolebinding --role=pod-role --serviceaccount=apps:pod-access --namespace=apps
[user@console ~]$ kubectl get rolebinding -n apps
[user@console ~]$ kubectl describe rolebinding -n apps pod-rolebinding

```
