---
title: "[CKA] Deployment & Expose the Service"
excerpt: "Deployment & Expose the Service"

categories:
  - CKA
tags:
  - [cka, sidercar, pod]

permalink: /cka/2024-07-30-categories-CKA-11.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---

# [11] Deployment & Expose the Service

## ❓Deploy and Service

- Reconfigure the existing deployment `front-end` and add a port specification named `http` exposing port `80/tcp` of the existing container nginx.
- Create a new service named `front-end-svc` exposing the container port http.
- Configure the new service to also expose the individual Pods via a `NodePort` on the nodes on which they are scheduled. 
    - 작업 클러스터: k8s

### Reference

docs에서 nodeport검색

[Service](https://kubernetes.io/docs/concepts/services-networking/service/)

### 실습

```docker
[user@console ~]$ kubectl config use-context k8s

[user@console ~]$ kubectl get deployment front-end

[user@console ~]$ kubectl get deployment front-end -o yaml > front-end.yaml

[user@console ~]$ vi front-end.yaml

기존 파일에 밑의 내용 추가

spec:
  containers:
  - image: nginx
		name: http
#여기까지가 기존내용, 아래 내용 전부 넣기
    ports:
      - containerPort: 80
        name: http

---
apiVersion: v1
kind: Service
metadata:
  name: front-end-svc
spec:
	type: NodePort
  selector:
    run: nginx
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: http

:wq

# 충돌되지 않도록 삭제 후 생성
[user@console ~]$ kubectl delete deployment front-end

[user@console ~]$ kubectl apply -f front-end.yaml

```

## 기출문제 (31번)

Create a deployment as follows: Name: nginx-random Exposed via a service nginx-random Ensure that the service &amp; pod are accessible via their respective DNS records.

The container(s) within any pod(s) running as a part of this deployment should use the nginx Image.

Next, use the utility nslookup to look up the DNS records of the service &amp; pod and write the output to /opt/KUNW00601/service.dns and /opt/KUNW00601/pod.dns respectively.

- 답안
    
    ```jsx
    kubectl create deploy nginx-random --image=nginx --dry-run=client -o yaml
    ```

## 기출문제 (63번)

Create and configure the service front-end-service so it's accessible through NodePort and routes to the existing pod named front-end.

- 답안  
    ```jsx
    kubectl expose pod front-end --port=80 --protocol=TCP --target-port=80 --name=front-end-service --type=NodePort
    ```
