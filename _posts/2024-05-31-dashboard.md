---
title: "[Kubernetes]-Dashboard"
excerpt: "쿠버네티스 Dashboard"

categories:
  - Kubernetes
tags:
  - [dashboard, kubernetes]

permalink: /kubernetes/k-dashboard/

toc: true
toc_sticky: true

date: 2024-05-31
last_modified_at: 2024-05-31
---


## Kubenetes Dashboard & SSL

### 1. Dashboard 설치
```
- K8S dashboard 구성 + ca.crt(인증서) -> 보안 접근 구성
- 설치 : kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
- 설치확인 : kubectl get pod --all-namespaces
- 대시보드 log : kubectl -n kubernetes-dashboard logs -f kubernetes-dashboard-6c8b589b99-5f4hg
- 대시보드 describe : kubectl -n kubernetes-dashboard describe pod kubernetes-dashboard-6c8b589b99-5f4hg
``` 
