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

### 2. Dashboard 보안접근
> 접근권한 > RBAC(Role Based Access Control) > ClusterRoleBing

```
- mkdir dashboard_token && cd $_
- vi Cluster
- vi ClusterRoleBinding.yaml
```

### 3. 파일 커스텀 
```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - nodePort: 30880	# <-- 여기 노드 포트 뚫
      port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort		# <-- 노드 포트를 쓰겠다는 설정
```

### 4. 대시보드 클러스터 바인딩 생성
```
cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

### 5. 대시보드 사용자 서비스 맵핑
```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

### 6. 토큰 제한시간 변경 (선택 사항)
```
kubectl edit -n kubernetes-dashboard deployments.apps kubernetes-dashboard

# ... content before...

    spec:
      containers:
      - args:
        - --auto-generate-certificates
        - --namespace=kubernetes-dashboard
        - --token-ttl=0 # <-- 이걸 추가
      image: kubernetesui/dashboard:v2.6.0
```

### 7. 발행된 토큰으로 로그인
> kubectl -n kubernetes-dashboard create token admin-user


### 8. 토큰 확인
```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}') 
```




