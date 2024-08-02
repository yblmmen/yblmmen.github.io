---
title: "[CKA] Ingress 구성"
excerpt: "Ingress 구성"

categories:
  - CKA
tags:
  - [cka, ingress]

permalink: /cka/2024-08-01-categories-CKA-18.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [18] Ingress 구성

# Ingress란?

인그레스는 클러스터 외부에서 클러스터 내부 서비스로 HTTP와 HTTPS 경로를 노출한다.

트래픽 라우팅은 인그레스 리소스에 정의된 규칙에 의해 컨트롤된다.

### Reference

docs에서 ingress 검색

[인그레스(Ingress)](https://kubernetes.io/ko/docs/concepts/services-networking/ingress/)

## ❓Ingress 구성

- Application Service 운영 
    - 작업 클러스터: k8s
    - `ingress-nginx` namespace에 nginx 이미지를 `app=nginx` 레이블을 가지고 실행하는 `nginx` pod를 구성하시오
    - 앞서 생성한 nginx pod를 서비스하는 nginx service를 생성하세요
    - 현재 `appjs-service` Service는 이미 동작 중입니다. 별도 구성이 필요 없습니다.
- Ingress 구성 
    - `app-ingress.yaml` 파일을 생성하여 다음 조건의 ingress 서비스를 구성하세요 
        - ingress name: `app-ingress`
        - NODE\_PORT:30080/ 접속했을 때 nginx 서비스로 연결
        - NODE\_PORT:30080/app 접속했을 때 appjs-service 서비스로 연결
        - ingress 구성에 다음 annotations을 포함시키시오
            
            annotations:
            
            [kubernetes.io/ingress.class:](http://kubernetes.io/ingress.class:) nginx

### 실습

```docker
[user@console ~]$ kubectl config use-context k8s 

# ingress-nginx 네임스페이스에 있는 파드 가져옴
[user@console ~]$ kubectl get pods -n ingress-nginx

# 조건에 충족하는 파드를 run 명령으로 ingress-nginx 네임스페이스에 생성
[user@console ~]$ kubectl run nginx --namespace=ingress-nginx --image=nginx --labels=app=nginx

[user@console ~]$ kubectl get pods -n ingress-nginx

# 서비스를 생성하기 위해 방금 생성한 파드를 expose함
[user@console ~]$ kubectl expose -n ingress-nginx pod nginx --port 80 --target-port 80

[user@console ~]$ kubectl get svc -n ingress-nginx

[user@console ~]$ kubectl describe svc -n ingress-nginx nginx

---------- docs에서 ingress 검색 -> The Ingress resource 검색해서 다 긁어옴 ----------

# yaml파일 vi로 만들어서 긁어온 내용 복붙하고 수
[user@console ~]$ vi app-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
	namespace: ingress-nginx
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
		kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
			- path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
# 여기 위 number 포트는 위에서kubectl get svc -n ingress-nginx했을 때 
# nginx, appjs-service가 어떤 포트 사용하고 있는지 확인하고 넣어준다.
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: appjs-service
            port:
              number: 80
			
:wq

# namespace가 ingress-nginx인 곳에서 돌아가고 있는 서비스 목록 확인
[user@console ~]$ kubectl get svc -n ingress-nginx
appjs-service서비스는 80포트 사용
nginx 서비스는 80포트 사용

[user@console ~]$ kubectl apply -f app-ingress.yaml

[user@console ~]$ kubectl get ingress -n ingress-nginx

[user@console ~]$ kubectl describe ingress -n ingress-nginx

[user@console ~]$ kubectl get nodes

[user@console ~]$ curl k8s-worker1:30080

[user@console ~]$ curl k8s-worker1:30080/app

```

## 기출문제 (30번)

Task Create a new nginx Ingress resource as follows:

- Name: ping
- Namespace: ing-internal
- Exposing service hi on path /hi using service port 5678
- hi서비스는 있다고 가정
- 답안  
    ```jsx
    vi ping.yaml
    
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      namespace: ing-internal
      name: ping
    spec:
      rules:
      - http:
          paths:
          - path: /hi
            pathType: Prefix
            backend:
              service:
                name: hi
                port:
                  number: 5678
    
    :wq
    
    ```
    
    ```jsx
    kubectl apply -f ping.yaml
    
    ```
