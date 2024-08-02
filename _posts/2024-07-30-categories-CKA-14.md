---
title: "[CKA] init 컨테이너를 포함한 Pod 운영"
excerpt: "init 컨테이너를 포함한 Pod 운영"

categories:
  - CKA
tags:
  - [cka, init, container]

permalink: /cka/2024-07-30-categories-CKA-14.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [14] init 컨테이너를 포함한 Pod 운영

# init 컨테이너에 대해 알아보기

docs에서 init container 검색

[초기화 컨테이너](https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/)

## init 컨테이너란

초기화 컨테이너는 파드의 앱 컨테이너들이 실행되기 전에 실행되는 특수한 컨테이너이다.

이 init 컨테이너가 정상적으로 실행되야 main 컨테이너가 실행되고 실패할 경우 성공할때까지 재실행된다.

비슷한 경우로는 AWS의 user data같은거

## Tip!

이 유형에 대한 문제는 CKA보다는 CKAD에서 더 자주 나오는 유형의 문제이다

## ❓Additional init container

- Add an init container to web-pod(which has been defined in spec file /data/cka/webpod.yaml)
- The init container should create an empty file named /workdir/data.txt
- If /workdir/data.txt is not detected, the Pod should exit.
- Once the spec file has been updated with the init container definition, the Pod should be created.

### 실습

```docker
[user@console ~]$ vi /data/cka/webpod.yaml

apiVersion: v1
kind: Pod
metadata:
	name: web-pod
spec:
	containers:
	- image: busybox:1.28
		name: main
		command: ['sh', '-c', 'if [ !-f /workdir/data.txt ];then exit 1;else sleep 300;fi']
		volumeMounts:
		- name: workdir
			mountPath: "/workdir"
#여기까지 기존내용, 여기서부터
	initContainers:
	- name: init
		image: busybox:1.28
		command: ['sh', '-c', "touch /workdir/data.txt"]
		volumeMounts:
		- name: workdir
			mountPath: "/workdir"
#여기위까지 내용을 추가
	volumes:
	- name: workdir
		emptyDir: {}

:wq

[user@console ~]$ kubectl apply -f /data/cka/webpod.yaml

[user@console ~]$ kubectl get pods 
web-pod라는 이름의 pod가 실행됬는지 확인하기

```

## 기출문제 (40번)

Perform the following tasks: Add an init container to hungry-bear (which has been defined in spec file /opt/KUCC00108/pod-spec-KUC C00108.yaml )

The init container should create an empty file named /workdir/calm.txt If /workdir/calm.txt is not detected, the pod should exit.

Once the spec file has been updated with the init container definition, the pod should be created

- 답안  
    ```jsx
    vi /opt/KUCC00108/pod-spec-KUC C00108.yaml
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: hungry-bear
    spec:
      volumes:
      - name: workdir
        emptyDir: {}
      containers:
      - image: alpine
        name: checker
        command: ["/bin/sh", "-c", "if [ -f /workdir/calm.txt ]; then sleep 100000; else exit 1; fi"]
        volumeMounts:
        - name: workdir
          mountPath: /workdir
      initContainers:
      - name: init-myservice
        image: alpine
        command: ["/bin/sh", "-c", "touch /workdir/calm.txt"]
        volumeMounts:
        - name: workdir
          mountPath: /workdir
    
    :wq
    
    ```
    
    ```jsx
    kubectl apply -f /opt/KUCC00108/pod-spec-KUC C00108.yaml
    
    ```
