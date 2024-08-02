---
title: "[CKA] NodePort 서비스 생성"
excerpt: "NodePort 서비스 생성"

categories:
  - CKA
tags:
  - [cka, NodePort, service]

permalink: /cka/2024-07-30-categories-CKA-15.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [15] NodePort 서비스 생성

# NodePort란?

클러스터 외부의 사용자가 클러스터 내에 위치한 서비스에 접근하도록 할 수 있게 하는 서비스타입

## ❓NodePort Service

- Create the service as type NodePort with port 32767 for the nginx pod with the pod selector app:webui
- 작업 클러스터: k8s

### Reference

docs에서 nodeport 검색 → Type Nodeport 부분

[Service](https://kubernetes.io/docs/concepts/services-networking/service/)

### 실습

```docker
[user@console ~]$ kubectl config use-context k8s

[user@console ~]$ kubectl get pod --selector app=webui

[user@console ~]$ vi myservice.yaml

docs에 나와있는 example 가공

apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app:webui
  ports:
    - port: 80
      targetPort: 80
      nodePort: 32767

:wq

[user@console ~]$ cat myservice.yaml

[user@console ~]$ kubectl apply -f myservice.yaml

[user@console ~]$ kubectl get services my-service

[user@console ~]$ kubectl get nodes
ready상태인 노드 아무거나 본 다음

# 결과 출력 확인
[user@console ~]$ curl {node이름}:32767

```

## 기출문제 (63번)

Create and configure the service front-end-service so it's accessible through NodePort and routes to the existing pod named front-end.

- 답안
    
    ```jsx
    kubectl expose pod front-end --name=front-end-service --port=80 --target-port=80 --type=NodePort
    
    ```
