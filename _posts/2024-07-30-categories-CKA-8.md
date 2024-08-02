---
title: "[CKA] NodeSelector"
excerpt: "NodeSelector 실행하기"

categories:
  - CKA
tags:
  - [cka, NodeSelector, pod]

permalink: /cka/2024-07-30-categories-CKA-8.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---

# [8] NodeSelector

# NodeSelector란?

특정 Application Pod를 특정 Node에 실행시켜달라는 것

## ❓Schedule a pod

- 작업 클러스터: k8s
- Schedule a pod as follows: 
    - Name: eshop-store
    - Image: nginx
    - Node selector: disktype=ssd

### Reference

docs에서 nodeselector 검색

[Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

### 실습

```docker
# 클러스터 전환
[user@console ~]$ kubectl config use-context k8s

# node 정보 확인 (label 정보랑 같이)
[user@console ~]$ kubectl get nodes --show-labels
NAME          STATUS
k8s-master    READY
k8s-worker1   READY
k8s-worker2   READY

# node 정보 확인 (disktype label 정보랑 같이)
[user@console ~]$ kubectl get nodes -L disktype
NAME          STATUS   DISKTYPE
k8s-master    READY
k8s-worker1   READY    ssd
k8s-worker2   READY    std

# pod 생성하기
[user@console ~]$ kubectl run eshop-store --image=nginx --dry-run=client -o yaml

# yaml 템플릿으로 yaml 파일 생성
[user@console ~]$ kubectl run eshop-store --image=nginx --dry-run=client -o yaml > eshop-store.yaml

[user@console ~]$ vi eshop-store.yaml

spec:
	nodeSelector:
		disktype: ssd
추가하기

:wq

# 수정한 yaml파일로 pod 생성
[user@console ~]$ kubectl apply -f eshop-store.yaml

# 확인
[user@console ~]$ kubectl get pods -o wide eshop-store
NODE 부분 확인하면 k8s-worker1에 생성된 것 확인

```

## 기출문제 (36번)

Schedule a pod as follows: Name: nginx-kusc00101 Image: nginx Node selector: disk=ssd

- 답안  
    ```jsx
    # 각 노드들이 disk 라벨값을 뭘로 가지는지 확인해서 어느 노드에 할당될지 미리 파악하기
    kubectl get nodes -L disk
    
    ```
    
    ```jsx
    kubectl run nginx-kusc00101 --image=nginx --dry-run=client -o yaml > kusc.yaml
    
    ```
    
    ```jsx
    # nodeSelector부분 container와 동일한 급으로 추가
    vi kusc.yaml
    
    ...
    spec:
    	container:
    		...
    	nodeSelector:
    		disk: ssd
    ...
    
    :wq
    
    ```
    
    ```jsx
    kubectl apply -f kusc.yaml
    
    ```
    
    ```jsx
    # 확인하기
    kubectl get pods
    
    kubectl describe pod nginx-kusc00101
    
    ```
