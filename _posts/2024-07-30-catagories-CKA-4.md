---
title: "[CKA]  Multi-Container Pod 생성하기 "
excerpt: "Multi-Container Pod 생성하기"

categories:
  - CKA
tags:
  - [cka, multi, pod]

permalink: /cka/2024-07-30-categories-CKA-4.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---

# [4] Multi-Container Pod 생성하기

## ❓Multi-Container Pod 생성하기

- Create pod 
    - 작업 클러스터: hk8s
    - Create a pod named lab004 with 3 containers running: nginx, redis, memcached

### 실습

```docker
# 이미지가 nginx이고 이름이 lab004인 pod를 --dry-run 옵션으로 생성하고 해당 yaml파일을 multi.yaml이라는 이름의 파일로 출력
[user@console ~]$ kubectl run lab004 --image=nginx --dry-run=client -o yaml > multi.yaml

[user@console ~]$ ls
multi.yaml

[user@console ~]$ vi multi.yaml
apiVersion: v1
kind: Pod
metadata: 
	name: lab004
spec:
	containers:
	- image: nginx
		name: nginx
	- image: redis
		name: redis
	- image: memcached
		name: memcached

:wq

# multi.yaml 파일가지고 실행
[user@console ~]$ kubectl apply -f multi.yaml 

[user@console ~]$ kubectl get pods
lab004  0/3  ContainerCreating

# lab004 pod안에 어떤 컨테이너가 동작중인지 확인
[user@console ~]$ kubectl describe pod lab004


```
