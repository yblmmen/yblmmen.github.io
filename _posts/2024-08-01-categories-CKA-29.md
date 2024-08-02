---
title: "[CKA] Kube-DNS"
excerpt: "Kube-DNS"

categories:
  - CKA
tags:
  - [cka, Kube-DNS]

permalink: /cka/2024-08-01-categories-CKA-29.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [29] Kube-DNS

# Kube-DNS란?

Kube-DNS는 도메인 이름을 IP 주소로 해석하는 역할을 한다. 이를 통해 K8s 클러스터 내부의 다른 리소스(서비스, Pod 등)를 도메인 이름을 통해 찾을 수 있다. Kube-DNS는 각 노드에 배포되며, 클러스터 내의 모든 Pod에서 사용할 수 있는 DNS 서버 역할을 수행 한.

## 나오는 문제 유형

- CoreDNS의 동작원리를 알고 있는가?
- 컨테이너 내부에 접속해서 서비스 또는 pod에 대한 DNS 질의를 해봐라

## ❓Service and DNS Lookup 구성

- Create a `nginx` pod called `nginx-resolver` using image `nginx`, expose it internally with a service called `nginx-resolver-service`.
- Test that you are able to look up the service and pod names from within the cluster.
- Use the image: `busybox:1.28` for dns lookup. 
    - Record results in /tmp/nginx.svc and /tmp/nginx.pod
    - Pod: **nginx-resolver** created
    - **Service DNS Resolution** recorded correctly
    - **Pod DNS resolution** recorded correctly

### Reference

docs에서 DNS 검색

[DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

### DNS 질의하는 법

1. Service에 대해 질의하는 법

nslookup `{service 이름}`. `{service namespace}`.svc.cluster.local

nslookup `{Cluster IP}`

2. Pod에 대해 질의하는 법

`{Pod IP}`. `{Pod namespace}`.pod.cluster.local

### 실습

```jsx
kubectl run nginx-resolver --image=nginx 

kubectl expose pod nginx --name=nginx-resolver-service --port=80 --target-port=80

# 확인
kubectl get pod nginx-resolver -o wide
--> IP정보 확인하기

kubectl get service nginx-resolver-service

kubectl run test --image=busybox:1.28 --rm -it --restart=Never -- /bin/sh 

cat /etc/resolv.conf
--> DNS 서버 IP 확인

nslookup nginx-resolver-service.default.svc.cluster.local
nslookup {Cluster IP}
--> 두 개의 결과가 같은지 확인

nslookup {Pod IP}.default.pod.cluster.local
--> DNS 서버 IP맞는지 확인

exit

vi /tmp/nginx.svc
--> service lookup 결과 안에 결과 넣어주기

vi /tmp/nginx.pod
--> pod lookup 결과 안에 결과 넣어주기

```

## 기출문제 (31번)

Create a deployment as follows: Name: nginx-random Exposed via a service nginx-random Ensure that the service &amp; pod are accessible via their respective DNS records.

The container(s) within any pod(s) running as a part of this deployment should use the nginx Image.

Next, use the utility nslookup to look up the DNS records of the service &amp; pod and write the output to /opt/KUNW00601/service.dns and /opt/KUNW00601/pod.dns respectively.

- 답안
    
    ```jsx
    kubectl create deploy nginx-random --image=nginx
    
    ```
    
    ```jsx
    kubectl expose deploy nginx-random --name=nginx-random --port=80 --target-port=80
    
    ```
    
    ```jsx
    kubectl run test --image=busybox:1.28 --rm -it --restart=Never -- /bin/sh
    
    ```
    
    ```jsx
    cat /etc/resolv.conf
    
    nslookup nginx-random.default.svc.cluster.local
    --> 결과 복사
    
    nslookup 10.244.1.24.default.pod.cluster.local
    --> 결과 복사
    
    # pod에서 나오기
    exit
    
    ```
    
    각각 결과 복사한거 파일에 넣기
