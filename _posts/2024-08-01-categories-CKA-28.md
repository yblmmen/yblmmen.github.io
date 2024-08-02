---
title: "[CKA] ServiceAccount Cluster Role Binding"
excerpt: "ServiceAccount Cluster Role Binding"

categories:
  - CKA
tags:
  - [cka, ServiceAccount, Cluster, Role, Binding]

permalink: /cka/2024-08-01-categories-CKA-28.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [28] ServiceAccount Cluster Role Binding

## ❓ClusterRole &amp; ClusterRoleBinding 구성

- Create a new ClusterRole named `deployment-clusterrole`, which only allows to create the following resource types: Deployment, StatefulSet, DaemonSet
- Create a new ServiceAccount named `cicd-token` in the existing namespace `apps`
- Bind the new ClusterRole `deployment-clusterrole` to the new ServiceAccount `cicd-token`, limited to the namespace `apps`

### 실습

```docker
# service account 생성 및 확인
[user@console ~]$ kubectl create serviceaccount cicd-token -n apps
[user@console ~]$ kubectl get sa -n apps

# cluster role 생성
[user@console ~]$ kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployment,statefulset,daemonset

# role binding 생성
[user@console ~]$ kubectl create clusterrolebinding deploy-clusterrolebinding --clusterrole=deployment-clusterrole --serviceaccount=apps:cicd-token

# 확인
[user@console ~]$ kubectl get rolebinding deploy-clusterrolebinding
[user@console ~]$ kubectl describe rolebinding deploy-clusterrolebinding 

```

## 기출문제

Cluster : k8s Context You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace. Task:

- Create a new ClusterRole named deployment-clusterrole , which only allows to create the following resource types: Deployment StatefulSet DaemonSet
- Create a new ServiceAccount named cicd-token in the existing namespace app-team1 .
- Bind the new ClusterRole deployment-clusterrole to the new ServiceAccount cicd-token , limited to the namespace app-team1
- 답안
    
    ```jsx
    kubectl create clusterrole deployment-clusterrole --resource=deployment,statefulset,daemonset --verb=create
    
    ```
    
    ```jsx
    kubectl create serviceaccount cicd-token -n app-team1
    
    ```
    
    ```jsx
    kubectl create clusterrolebinding deployment-clusterrolebinding --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token
    
    ```
