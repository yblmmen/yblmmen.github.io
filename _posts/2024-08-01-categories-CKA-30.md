---
title: "[CKA] Network Policy"
excerpt: "Network Policy"

categories:
  - CKA
tags:
  - [cka, Network, Policy]

permalink: /cka/2024-08-01-categories-CKA-30.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [30] Network Policy

쉽게 말하면 Pod에 대한 방화벽을 설정해주는 것이다.

## ❓Network Policy with Namespaces

- Create a new Network Policy named allowed-port-from-namespace in the existing namespace devops.
- Ensure that the new Network Policy allows pods in namespace migops to connect to port 80 of pods in namespace devops

### 실습

```docker
# docs에서 NetworkPolicy검색하고 The NetworkPolicy resource 부분  
vi allow-port-from-namespace.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allowed-port-from-namespace
  namespace: devops
spec:
  podSelector:
    matchLabels:
      app: web
# namespace devops에 있던 pod가 가지는 라벨
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              team: migops
# migops라는 namespace가 가지는 라벨
      ports:
        - protocol: TCP
          port: 80

:wq

kubectl apply -f allow-port-from-namespace.yaml
kubectl get networkpolicies -n devops allow-port-from-namespace

```

## 기출문제 (46번)

Task Create a new NetworkPolicy named allow-port-from-namespace in the existing namespace echo. Ensure that the new NetworkPolicy allows Pods in namespace my-app to connect to port 9000 of Pods in namespace echo. Further ensure that the new NetworkPolicy:

- does not allow access to Pods, which don't listen on port 9000
- does not allow access from Pods, which are not in namespace my-app
- 답안  
    //답안을 찾아야된다.
