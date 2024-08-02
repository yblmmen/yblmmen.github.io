---
title: "[CKA] User Cluster Role Binding"
excerpt: "User Cluster Role Binding"

categories:
  - CKA
tags:
  - [cka, User, Cluster, Role, Binding]

permalink: /cka/2024-08-01-categories-CKA-26.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [26] User Cluster Role Binding

## ❓Configuring User API Authentication

- Create a new ClusterRole named `app-clusterrole`, which only allows to get, watch, list the following resource types: Deployment, Service
- Bind the new ClusterRole `app-clusterrole` to the new user `ckcuser`
- User ckauser and ckauser clusters are ready configured.
- To check the results, run the following command: 
    - kubectl config use-context ckauser
- 작업 클러스터: k8s

## Role vs Cluster Role

일반적인 role은 namespace단위로 적용되는 거지만 cluster role은 namespace상관없이 특정 cluster 단위로 적용 된다. (role &lt; cluster role)

## Reference

docs에서 cluster role 검색 → 페이지에서 오른쪽에 Command-line utilities에서 kubectl create clusterrole

[Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#kubectl-create-clusterrole)

### 실습

```docker
# Cluster Role 생성
[user@k8s-master ~]$ kubectl create clusterrole app-clusterrole --verb=get,list,watch --resource=deployment,service
[user@k8s-master ~]$ kubectl get clusterrole app-clusterrole
[user@k8s-master ~]$ kubectl describe clusterrole app-clusterrole

# Cluster RoleBinding 설정
[user@k8s-master ~]$ kubectl create clusterrolebinding app-clusterrolebinding --clusterrole=app-clusterrole --user=ckauser
[user@k8s-master ~]$ kubectl get clusterrolebinding app-clusterrolebinding
[user@k8s-master ~]$ kubectl describe clusterrolebinding app-clusterrolebinding

# 확인해보기
[user@k8s-master ~]$ kubectl config use-context ckauser

[user@k8s-master ~]$ kubectl get deployment
--> 권한 있음

[user@k8s-master ~]$ kubectl get services
--> 권한 있음

[user@k8s-master ~]$ kubectl get secret
--> 권한 없음

```

## 기출문제 (37번)

Context You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace. Task Create a new ClusterRole named deployment-clusterrole, which only allows to create the following resource types:

- Deployment
- StatefulSet
- DaemonSet Create a new ServiceAccount named cicd-token in the existing namespace app-team1. Bind the new ClusterRole deployment-clusterrole lo the new ServiceAccount cicd-token , limited to the namespace app-team1.
- 답안
    
    ```jsx
    kubectl create clusterrole deployment-clusterrole --resource=deployment,statefulset,daemonset --verb=create
    
    ```
    
    ```jsx
    kubectl create serviceaccount cicd-token -n app-team1
    
    ```
    
    ```jsx
    kubectl create rolebinding deployment-rolebinding --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token -n app-team1
    
    ```
