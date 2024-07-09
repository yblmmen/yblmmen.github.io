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
// 대시보드 배포 파일 다운로드
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

// 대시보드 서비스 nodeport 수정
 kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
    # 45번 라인에 NodePort 추가
    39 spec:
    40   ports:
    41     - port: 443
    42       targetPort: 8443
    43   selector:
    44     k8s-app: kubernetes-dashboard
    45   type: NodePort

// 수정한 배포 파일로 대시보드 배포
kubectl apply -f recommended.yaml

```

#### 쿠버네티스 대시보드 7.x 버전부터는 helm을 이용한 설치방법을 권고한다.
```
// Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/

// Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard

```

#### 대시보드에서 metrics 정보를 표시하기 위해 metrics server를 설치한다.
```
kubectl create -f https://raw.githubusercontent.com/k8s-1pro/install/main/ground/k8s-1.27/metrics-server-0.6.3/metrics-server.yaml
```

#### 대시보드 포트포워딩
```
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443 --address 0.0.0.0
```


### 2. 대시보드 서비스 확인
```
kubectl get services -n kubernetes-dashboard
// 확인한 포트 번호 기록

# 방화벽에 대시보드 포트 추가
firewall-cmd --permanent --add-port=확인한포트번호/tcp
firewall-cmd --reload

// 웹 브라우저에서 대시보드 접속

https://마스터노드IP:확인한포트번호
 
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
// 클러스터 관리자에 대한 역할 바인딩 생성
cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
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
        - --enable-skip-login               # 로그인 스킵
        - --auto-generate-certificates
        - --namespace=kubernetes-dashboard
        - --token-ttl=0                     # 토큰 만료 제거
      image: kubernetesui/dashboard:v2.6.0
```


### 7. 토큰 확인
```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}') 
```


### 8. 발행된 토큰으로 로그인
> kubectl -n kubernetes-dashboard create token admin-user



