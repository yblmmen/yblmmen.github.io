---
title: "[쿠버네티스]-Istio-1"
excerpt: "Istio-1"

categories:
  - Kubernetes
tags:
  - [kubernetes, istio]

permalink: /kubernetes/cni/

toc: true
toc_sticky: true

date: 2025-04-24
last_modified_at: 2025-04-24
---

##  **1장 서비스 메시 소개**  

---

### **1\. 서비스 메시란 ?**

> 서비스메시란 애플리케이션 대신 프로세스 외부에서 투명하게 네트워크 트래픽을 처리하는 분산형 애플리케이션 인프라를 말한다

-   안전하고 복원력 있고 관찰 가능하고 제어할 수 있게 하는 분산 애플리케이션 네트워킹 인프라 특정 프로그래밍 언어나 프레임워크에 의존하지 않고, 중요한 애플리케이션-네트워킹 기능을 애플리케이션 외부에서 구축
-   등장배경 : 마이크로서비스 아키텍처 환경의 시스템 전체 모니터링의 어려움, 운영 시 시스템 장애나 문제 발생할 때 원인과 병목 구간 찾기 어려움 내부망 진입점에 역할을 하는 GW(예. API Gateway) 경우 모든 동작 처리에 무거워지거나, 내부망 내부 통신 제어는 어려움  
    

[##_Image|kage@bVCTYE/btsNiwDU8oY/PPRQWW8e5aOvWzK5Bg2Cr1/img.png|CDM|1.3|{"originWidth":1792,"originHeight":1024,"style":"alignCenter","filename":"image (5).png"}_##]

 **\-** **개념 :** 마이크로서비스 간에 매시 형태의 통신이나 그 경로를 제어 - 예) 이스티오(Istio), 링커드(Linkerd)

 **- 기본동작 :** 파드 간 통신 경로에 프록시를 놓고 트래픽 모니터링이나 트래픽 컨트롤 → 기존 애플리케이션 코드수정 필요없음!

   1. 기존 통신 환경

[##_Image|kage@dt3XYw/btsNkIQZZZK/aTGKBfuFJODE0a5w5ZWpP1/img.png|CDM|1.3|{"originWidth":1837,"originHeight":320,"style":"alignCenter","width":826,"height":144,"filename":"1.png"}_##]

   2. 애플리케이션 수정 없이, 모든 애플리케이션 통신 사이에 Proxy 를 두고 통신 해보자

[##_Image|kage@bg4ten/btsNh1YsZzy/cH3wokNhCvcp8ZwNbQi6Ik/img.png|CDM|1.3|{"originWidth":1837,"originHeight":449,"style":"alignCenter","width":823,"height":201,"filename":"2.png"}_##]

-   파드 내에 사이드카 컨테이너로 주입되어서 동작
-   Proxy 컨테이너가 Application 트래픽을 가로채야됨 → **iptables rule** 구현 ⇒ 가능한 이유는?

   3. Proxy 는 결국 DataPlane 이니, 이를 중앙에서 관리하는 ControlPlane을 두고 중앙에서 관리를 하자

[##_Image|kage@rFt7M/btsNiOFq6wv/uWKDFJotaXDOVc28QTo970/img.png|CDM|1.3|{"originWidth":1837,"originHeight":880,"style":"alignCenter","width":812,"height":389,"filename":"3.png"}_##]

-   Proxy 는 중앙에서 설정 관리가 잘되는 툴을 선택. 즉, 원격에서 동적인 설정 관리가 유연해야함 → 풍부한 API 지원이 필요 ⇒ Envoy
    -   '구글 IBM 리프트(Lyft)'가 중심이 되어 개발하고 있는 오픈 소스 소프트웨어이며, C++ 로 구현된 **고성능 Proxy** 인 엔보이(Envoy)
    -   네트워크의 투명성을 목표, 다양한 **필터체인** 지원(L3/L4, HTTP L7), **동적 configuration API 제공, api 기반 hot reload 제공**
-   중앙에서 어떤 동작/설정을 관리해야 될까? 라우팅, 보안 통신을 위한 mTLS 관련, 동기화 상태 정보 등
-   서비스 메시란 **데이터 플레인**과 **컨트롤 플레인**으로 구성된 아키텍처                                                                                     
-   데이터 플레인 : 애플리케이션 계층 프록시를 사용해 애플리케이션 대신 네트워킹 트래픽을 관리
-   컨트롤 플레인 :  데이터 플레인을 설정하고 프록시를 관리

[##_Image|kage@bLRyRO/btsNiJxirWu/KHUEbvIAAl19ctzlY88XA0/img.png|CDM|1.3|{"originWidth":965,"originHeight":612,"style":"alignCenter","caption":"출처: https://kimdoky.github.io/devops/2025/04/10/study-istio-week1/","filename":"image.png"}_##]

-   데이터 플레인과 컨트롤 플레인이 모여 모든 클라우드 네트워크 아키텍처에 필요한 다음과 같은 **중요 기능**을 제공

       - 서비스 복원력

       - 관찰 가능성 신호

       - 트래픽 제어 기능

       - 보안

       - 강제 정책 Policy enforcement

-   **트래픽 모니터링** : 요청의 '에러율, 레이턴시, 커넥션 개수, 요청 개수' 등 메트릭 모니터링, 특정 서비스간 혹은 특정 요청 경로로 필터링 → 원인 파악 용이!
-   **트래픽 컨트롤** : 트래픽 시프팅(Traffic shifting), 서킷 브레이커(Circuit Breaker), 폴트 인젝션(Fault Injection), 속도 제한(Rate Limit)
    -   트래픽 시프팅(Traffic shifting) : 예시) 99% 기존앱 + 1% 신규앱 , 특정 단말/사용자는 신규앱에 전달하여 단계적으로 적용하는 카니리 배포 가능
    -   서킷 브레이커(Circuit Breaker) : 목적지 마이크로서비스에 문제가 있을 시 접속을 차단하고 출발지 마이크로서비스에 요청 에러를 반환 (연쇄 장애, 시스템 전제 장애 예방)
    -   폴트 인젝션(Fault Injection) : 의도적으로 요청을 지연 혹은 실패를 구현
    -   속도 제한(Rate Limit) : 요청 개수를 제한

---

### <참고자료> 

[##_Image|kage@6sHii/btsNkvK7piY/xYGFSYKPmWvwxk2NhAF4J1/img.png|CDM|1.3|{"originWidth":1040,"originHeight":1792,"style":"alignCenter","filename":"CleanShot_2024-10-18_at_00.11.37.png"}_##]

---

### **2\. Istio 서비스 소개**

> 이스티오 istio 는 그리스어로 ‘돛’을 의미

-   오픈소스 구현체이며 구글, IBM, 리프트가 주도
-   서비스 아키텍처에 복원력과 관찰 가능성을 투명한 방식으로 추가하는 데 도움
-   데이터 플레인은 엔보이 프록시를 사용하며 서비스 프록시 인스턴스가 함께 배포되도록 애플리케이션을 구성하는 데 도움
-   쿠버네티스, 오픈시프트와 같은 배포 플랫폼은 물론, 가상머신 같은 기존 배포 환경에서도 이스티오 기반 서비스 메시를 사용
-   이스티오의 컨트롤 플에인은 최종 사용자/운영자용 API, 프록시용 설정 API, 보안 설정, 정책 선언 등을 제공하는 몇 가지 구성 요소로 이루어짐
-   애플리케이션은 더 이상 서킷 브레이커, 시간 초과, 재시도, 서비스 디스커버리, 로드 밸런싱 등을 위해 언어별 복원력 라이브러리가 필요하지 않음.
-   서비스가 mTLS를 바로 사용할 수 있도록 키 및 인증서 발급, 설치, 로테이션을 관리
-   애플리케이션 네트워킹 경로의 양 끝단을 제어하는 덕분에 기본적으로 트래픽을 투명하게 암호화
-   기본적으로 보안 활성화
-   컨트롤 플레인 istiod 는 라우팅, 보안, 텔레메트릭 수집, 복원력을 처리하는 이스티오 프록시를 설정하는 데 사용
-   워크로드에 ID를 부여하고 이를 인증서에 포함
-   ID를 사용해 강력한 접근 제어 정책을 구현
-   할당량, 속도 제한, 조직 정책을 구현
-   애플리케이션 대신 외부에서 투명한 방식으로 구현하는 인프라

[##_Image|kage@bKVaii/btsNh4HvHAG/RKhJJ5GsXbZ2fzNT56cl21/img.png|CDM|1.3|{"originWidth":1635,"originHeight":1131,"style":"alignCenter","caption":"https://istio.io/latest/docs/ops/deployment/architecture/"}_##][##_Image|kage@c7fDKq/btsNivrrSKy/EZsfxDUhl0eWh5ObNB33LK/img.png|CDM|1.3|{"originWidth":1280,"originHeight":636,"style":"widthContent","filename":"image (6).png"}_##]

-   **파일럿(Pilot)**: 모든 Envoy 사이드카에서 프록시 라우팅 규칙을 관리하며, 서비스 디스커버리와 로드 밸런싱 설정을 제공
-   **갤리(Galley)**: Istio와 쿠버네티스(TLS 연결 및 파일럿에 필요한 설정)를 연결해 주는 역할을 합니다. 서비스 메시 구성 데이터를 검증하고 변환
-   **시타델(Citadel)**: 보안 기능을 담당하며, TLS 인증서 발급 및 관리를 통해 서비스 간 통신의 암호화를 수행  

---

### **3\. Istio 구성요소**

> Istio 구성요소와 envoy : 컨트롤 플레인(istiod) , 데이터 플레인(istio-proxy > envoy)

-   **istiod** : **Pilot**(데이터 플레인과 통신하면서 라우팅 규칙을 동기화, ADS), **Gally**(Istio 와 K8S 연동, Endpoint 갱신 등), **Citadel**(연결 암호화, 인증서 관리 등)
-   **Istio proxy** : Golang 으로 작성되었고 envoy 래핑한 Proxy, istiod와 통신하고 서비스 트래픽을 통제, 옵저버빌리티를 위한 메트릭 제공

[##_Image|kage@tPyCx/btsNjbM6q1g/PgktQFjGbhhQPKWA8DgEx1/img.png|CDM|1.3|{"originWidth":1236,"originHeight":845,"style":"alignCenter","width":763,"height":522,"caption":"출처 : https://kimdoky.github.io/devops/2025/04/10/study-istio-week1/"}_##]

-   이스티오는 각 **파드** 안에 **사이드카**로 **엔보이 프록시**가 들어가 있는 형태
-   모든 마이크로서비스간 통신은 엔보이를 통과하여, **메트릭을 수집**하거나 **트래픽 컨트롤**을 할 수 있음
-   트래픽 컨트롤을 하기위해 엔보이 프록시에 **전송 룰**을 설정 → **컨트롤 플레인**의 **이스티오**가 정의된 정보를 기반으로 **엔보이 설정**을 하게 함
-   마이크로서비스 간의 통신을 mutual TLS 인증(**mTLS**)으로 서로 TLS 인증으로 암호화 할 수 있음
-   각 애플리케이션은 **파드** 내의 엔보이 프록시에 접속하기 위해 **localhost 에 TCP 접속**을 함
-   Envoy는 구성을 동적으로 관리하기 위한 강력한 API를 제공
-   Envoy의 **xDS Sync API**는 아래와 같은 레이어에서 동작                                                                                                      
    
    -   **LDS** - Listener Discovery Service
    -   **RDS** - Route Discovery Service
    -   **CDS** - Cluseter Discovery Service
    -   **EDS** - Endpoint Discovery Service
    

[##_Image|kage@b4fIA2/btsNiuUf2W6/cQSTLf6NKyy1lPuKIC7Nq0/img.png|CDM|1.3|{"originWidth":1828,"originHeight":1040,"style":"alignCenter","caption":"츨처 : https://www.notion.so/gasidaseo/1-Istio-1cd50aec5edf80c7891fd91d7f5e7433?pvs=4#1cd50aec5edf8147be3af7b479bd2e73"}_##]

  
  

> Istio 구성요소와 envoy : 컨트롤 플레인(istiod) - ADS 를 이용한 Configuration 동기화 - 데이터 플레인(istio-proxy → envoy)

-   **Cluster** : envoy 가 트래픽을 포워드할 수 있는 논리적인 서비스 (엔드포인트 세트), 실제 요청이 처리되는 IP 또는 엔드포인트의 묶음을 의미.
-   **Endpoint** : IP 주소, 네트워크 노드로 클러스터로 그룹핑됨, 실제 접근이 가능한 엔드포인트를 의미. 엔드포인트가 모여서 하나의 Cluster 가 된다.
-   **Listener** : 무엇을 받을지 그리고 어떻게 처리할지 IP/Port 를 바인딩하고, 요청 처리 측면에서 다운스트림을 조정하는 역할.
-   **Route** : Listener 로 들어온 요청을 어디로 라우팅할 것인지를 정의. 라우팅 대상은 일반적으로 Cluster 라는 것에 대해 이뤄지게 된다.
-   **Filter** : Listener 로부터 서비스에 트래픽을 전달하기까지 요청 처리 파이프라인
-   **UpStream** : envoy 요청을 포워딩해서 연결하는 백엔드 네트워크 노드 - 사이드카일때 application app, 아닐때 원격 백엔드
-   **DownStream** : An entity connecting to envoy, In non-sidecar models this is a remote client

> Envoy는 구성을 동적으로 관리하기 위한 강력한 API를 제공이 중요합니다.  
>   
>   
>   

[##_Image|kage@dcoSR2/btsNjNZNVqY/4DmZY84C2hgzjRFHy2PnuK/img.png|CDM|1.3|{"originWidth":501,"originHeight":410,"style":"alignCenter","filename":"image (7).png"}_##]

-   Service Mesh 솔루션이나, Gateway API 구현체들을 Enovy를 내부적으로 사용하고 있으며, Envoy가 제공하는 동적 구성을 위한 API (xDS Sync API)를 이용하여 다양한 네트워크 정책을 구성하게 됩니다.
-   Envoy의 **xDS** Sync API는 아래와 같은 레이어에서 동작하게 됩니다.
    -   LDS - Listener Discovery Service
    -   RDS - Route Discovery Service
    -   CDS - Cluseter Discovery Service
    -   EDS - Endpoint Discovery Service

[##_Image|kage@c39s8Q/btsNiyBH4EJ/VapB4MkQjOdt0sirT0i160/img.png|CDM|1.3|{"originWidth":966,"originHeight":417,"style":"alignCenter","filename":"image (9).png"}_##][##_Image|kage@bD8Uag/btsNg7rm3vU/fgvQltqvhVG4m8iyiMiCW1/img.png|CDM|1.3|{"originWidth":765,"originHeight":499,"style":"widthContent","filename":"CleanShot_2024-10-13_at_20.01.19 (1).png"}_##]

##   **2장 이스티오 실습 과제**   

---

### **\-  과제 실습환경**

-   Proxmox 하이퍼바이저
-   VM - 1식
-   우분투 22.04
-   vCPU 8, Memory 16G, HDD 150G
-   docker (kind - k8s 1.23.17) , istio 1.17.8

※ 개인 PC 와 노트북 사양이 좋지 못하여 실습환경을 구축하는데 제약이 있어 베어메탈 서버를 활용

### **1\. 도커 설치**

```
# 도커설치 
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh

# 설치확인
docker info
docker ps
sudo systemctl status docker
cat /etc/group | grep docker
```

필자는 VSCODE 환경에 익숙치 않아서 VSCODE 개발 편리 설정은 제외하였음

### **2\. kind 및 관리 툴 설치**

```
# 기본 사용자 디렉터리 이동
cd $PWD
pwd
/home/ubuntu

#
sudo systemctl stop apparmor && sudo systemctl disable apparmor

# 
sudo apt update && sudo apt-get install bridge-utils net-tools jq tree unzip kubectx kubecolor -y

# Install Kind
sudo curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
sudo chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind --version

# Install kubectl
sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo chmod +x kubectl
sudo mv ./kubectl /usr/bin
sudo kubectl version --client=true

# Install Helm
sudo curl -s https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version

# Source the completion
source <(kubectl completion bash)
echo 'source <(kubectl completion bash)' >> ~/.bashrc

# Alias kubectl to k
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc

# Install Kubeps & Setting PS1
git clone https://github.com/jonmosco/kube-ps1.git
echo -e "source $PWD/kube-ps1/kube-ps1.sh" >> ~/.bashrc
cat <<"EOT" >> ~/.bashrc
KUBE_PS1_SYMBOL_ENABLE=true
function get_cluster_short() {
  echo "$1" | cut -d . -f1
}
KUBE_PS1_CLUSTER_FUNCTION=get_cluster_short
KUBE_PS1_SUFFIX=') '
PS1='$(kube_ps1)'$PS1
EOT

# .bashrc 적용을 위해서 logout 후 터미널 다시 접속 하자
exit
```

### **2\. kind 및 관리 툴 설치**

```
# 클러스터 배포 전 확인
docker ps

# Create a cluster with kind
kind create cluster

# 클러스터 배포 확인
kind get clusters
kind get nodes
kubectl cluster-info

# 노드 정보 확인
kubectl get node -o wide

# 파드 정보 확인
kubectl get pod -A
kubectl get componentstatuses

# 컨트롤플레인 (컨테이너) 노드 1대가 실행
docker ps
docker images

# kube config 파일 확인
cat ~/.kube/config
혹은
cat $KUBECONFIG # KUBECONFIG 변수 지정 사용 시

# nginx 파드 배포 및 확인 : 컨트롤플레인 노드인데 파드가 배포 될까요?
kubectl run nginx --image=nginx:alpine
kubectl get pod -owide

# 노드에 Taints 정보 확인
kubectl describe node | grep Taints
Taints:             <none>

# 클러스터 삭제
kind delete cluster

# kube config 삭제 확인
cat ~/.kube/config
혹은
cat $KUBECONFIG # KUBECONFIG 변수 지정 사용 시
```

#### **(옵션) 전체 삭제**

```
#!/bin/bash

# 1. 기존 Kind 삭제
kind delete clusters --all
sudo rm -f /usr/local/bin/kind
rm -rf ~/.kube/config ~/.cache/kind

# 2. 도커 리소스 정리 (주의!)
docker ps -aq | xargs docker stop
docker system prune -a --volumes -f

# 3. Kind 재설치
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# 4. 클러스터 생성 및 확인
kind create cluster --name fresh-cluster
kubectl cluster-info
kubectl get nodes
```

### **4\. k8s(1.23.17) 배포**

```
#
git clone https://github.com/AcornPublishing/istio-in-action
cd istio-in-action/book-source-code-master

# pwd
root@ubuntu:/home/ubuntu/istio-in-action/book-source-code-master# pwd
/home/ubuntu/istio-in-action/book-source-code-master

code .

# 
kind create cluster --name myk8s --image kindest/node:v1.23.17 --config - <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30000 # Sample Application (istio-ingrssgateway)
    hostPort: 30000
  - containerPort: 30001 # Prometheus
    hostPort: 30001
  - containerPort: 30002 # Grafana
    hostPort: 30002
  - containerPort: 30003 # Kiali
    hostPort: 30003
  - containerPort: 30004 # Tracing
    hostPort: 30004
  - containerPort: 30005 # kube-ops-view
    hostPort: 30005
  extraMounts:
  - hostPath: /home/ubuntu/istio-in-action/book-source-code-master  # pwd 경로
    containerPath: /istiobook
networking:
  podSubnet: 10.10.0.0/16
  serviceSubnet: 10.200.1.0/24
EOF

# 설치 확인
docker ps

# 노드에 기본 툴 설치
docker exec -it myk8s-control-plane sh -c 'apt update && apt install tree psmisc lsof wget bridge-utils net-tools dnsutils tcpdump ngrep iputils-ping git vim -y'


# (참고)클러스터 삭제
kind delete cluster --name myk8s
docker ps
cat ~/.kube/config
```

#### **\- (옵션)편의성 툴 설치**

```
# (옵션) kube-ops-view
helm repo add geek-cookbook https://geek-cookbook.github.io/charts/
helm install kube-ops-view geek-cookbook/kube-ops-view --version 1.2.2 --set service.main.type=NodePort,service.main.ports.http.nodePort=30005 --set env.TZ="Asia/Seoul" --namespace kube-system
kubectl get deploy,pod,svc,ep -n kube-system -l app.kubernetes.io/instance=kube-ops-view

## kube-ops-view 접속 URL 확인
open "http://localhost:30005/#scale=1.5"
open "http://localhost:30005/#scale=1.3"

# (옵션) metrics-server
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm install metrics-server metrics-server/metrics-server --set 'args[0]=--kubelet-insecure-tls' -n kube-system
kubectl get all -n kube-system -l app.kubernetes.io/instance=metrics-server
```

[##_Image|kage@9eiBF/btsNkhMFbb5/KfSGJkBTso348gmBmMl40K/img.png|CDM|1.3|{"originWidth":1060,"originHeight":624,"style":"alignCenter","caption":"kube-ops-view 접속 화면"}_##]

#### **\- Utils 배포**

> krew 설치 - https://krew.sigs.k8s.io/docs/user-guide/setup/install/

1\. git 설치상태 확인

2\. krew install script

```
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```

3\. 환경변수 설정

```
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

4\. 설치확인

```
(⎈|kind-myk8s:N/A) ubuntu@ubuntu:~$ kubectl-krew list
PLUGIN         VERSION
access-matrix  v0.5.0
df-pv          v0.3.0
get-all        v1.3.8
krew           v0.4.5
neat           v2.0.4
oomd           v0.0.7
rbac-tool      v1.20.0
rolesum        v1.5.5
sniff          v1.6.2
stern          v1.32.0
tail           v0.17.4
view-secret    v0.14.0
whoami         v0.0.46
```

### **5\. Istio 1.17.8 배포 (for 베어메탈 WM)** 

```
# myk8s-control-plane 진입 후 설치 진행
docker exec -it myk8s-control-plane bash
-----------------------------------
# 코드 파일들 마운트 확인
tree /istiobook/ -L 1

# istioctl 설치
export ISTIOV=1.17.8
echo 'export ISTIOV=1.17.8' >> /root/.bashrc

curl -s -L https://istio.io/downloadIstio | ISTIO_VERSION=$ISTIOV sh -
tree istio-$ISTIOV -L 2 # sample yaml 포함
cp istio-$ISTIOV/bin/istioctl /usr/local/bin/istioctl

# 검증
istioctl version --remote=false

(⎈|kind-myk8s:N/A) ubuntu@ubuntu:~$ docker exec -it myk8s-control-plane bash
root@myk8s-control-plane:/# istioctl version --remote=false
1.17.8  // 버전 확인
  

# default 프로파일 컨트롤 플레인 배포
istioctl x precheck # 설치 전 k8s 조건 충족 검사
istioctl profile list
istioctl install --set profile=default -y
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Installation complete


# 설치 확인 : istiod, istio-ingressgateway, crd 등
kubectl get istiooperators -n istio-system
NAME              REVISION   STATUS   AGE
installed-state                       4d10h
 

kubectl get istiooperators -n istio-system -o yaml
...
  spec:
    components:
      base:
        enabled: true
      cni:
        enabled: false
      egressGateways:
      - enabled: false
        name: istio-egressgateway
      ingressGateways:
      - enabled: true
        name: istio-ingressgateway
      istiodRemote:
        enabled: false
      pilot:
        enabled: true
    hub: docker.io/istio
    meshConfig:
      defaultConfig:
        proxyMetadata: {}
      enablePrometheusMerge: true
    profile: default
    ...
      pilot:
        autoscaleEnabled: true
        autoscaleMax: 5
        autoscaleMin: 1
        configMap: true
        cpu:
          targetAverageUtilization: 80
        deploymentLabels: null
        enableProtocolSniffingForInbound: true
        enableProtocolSniffingForOutbound: true
        env: {}
        image: pilot
        keepaliveMaxServerConnectionAge: 30m
        nodeSelector: {}
        podLabels: {}
        replicaCount: 1
        traceSampling: 1
      telemetry:
        enabled: true
        v2:
          enabled: true
          metadataExchange:
            wasmEnabled: false
          prometheus:
            enabled: true
            wasmEnabled: false
          stackdriver:
            configOverride: {}
            enabled: false
            logging: false
            monitoring: false
            topology: false


# 설치확인
kubectl get all,svc,ep,sa,cm,secret,pdb -n istio-system


NAME                                       READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-996bc6bb6-s2rgz   1/1     Running   0          4d11h
pod/istiod-7df6ffc78d-qgtv6                1/1     Running   0          4d11h
                                    4d10h
service/istio-ingressgateway   NodePort    10.200.1.46    <none>        15021:31099/TCP,80:30000/TCP,443:30228/TCP   4d11h
service/istiod                 ClusterIP   10.200.1.186   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP        4d11h
 
NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           4d11h
deployment.apps/istiod                 1/1     1            1           4d11h

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-996bc6bb6   1         1         1       4d11h
replicaset.apps/istiod-7df6ffc78d                1         1       

NAME                                                      TYPE                                  DATA   AGE

secret/istio-ca-secret                                    istio.io/ca-root                      5      4d11h
secret/istio-ingressgateway-service-account-token-h49h5   kubernetes.io/service-account-token   3      4d11h
secret/istio-reader-service-account-token-9z9jw           kubernetes.io/service-account-token   3      4d11h
secret/istiod-service-account-token-cwmzb                 kubernetes.io/service-account-token   3      4d11h
secret/istiod-token-6xvdx                                 kubernetes.io/service-account-token   3      4d11h
   3      4d10h

NAME                                              MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
poddisruptionbudget.policy/istio-ingressgateway   1               N/A               0                     4d11h
poddisruptionbudget.policy/istiod                 1               N/A               0                     4d11h


 
#
kubectl get crd | grep istio.io | sort
root@myk8s-control-plane:/# kubectl get crd | grep istio.io | sort
authorizationpolicies.security.istio.io    2025-04-07T12:10:11Z
destinationrules.networking.istio.io       2025-04-07T12:10:11Z
envoyfilters.networking.istio.io           2025-04-07T12:10:12Z
gateways.networking.istio.io               2025-04-07T12:10:12Z
istiooperators.install.istio.io            2025-04-07T12:10:



istioctl verify-install # 설치 확인

root@myk8s-control-plane:/# istioctl verify-install

✔ Istio is installed and verified successfully


# 보조 도구 설치
kubectl apply -f istio-$ISTIOV/samples/addons

#
kubectl get pod -n istio-system
NAME                                   READY   STATUS    RESTARTS   AGE
istio-ingressgateway-996bc6bb6-s2rgz   1/1     Running   0          4d11h
istiod-7df6ffc78d-qgtv6                1/1     Running   0          4d11h



# 빠져나오기
exit
-----------------------------------


# 

kubectl get cm -n istio-system istio -o yaml
kubectl get cm -n istio-system istio -o yaml | kubectl neat
```

**\- 코드 파일들 마운트 확인**

[##_Image|kage@bzfQzj/btsNiWoCXq7/NKhKmvfeOB15iaXEyaiKw0/img.png|CDM|1.3|{"originWidth":1335,"originHeight":719,"style":"alignLeft","width":859,"height":463}_##]

### **6\. 서비스 메시 첫 애플리케이션 배포**

> **필자는 방법 2 - namespace에 label을 추가하는 방식으로 설치하였음**

```
#
kubectl create ns istioinaction

# 방법1 : yaml에 sidecar 설정을 추가
cat services/catalog/kubernetes/catalog.yaml
docker exec -it myk8s-control-plane istioctl kube-inject -f /istiobook/services/catalog/kubernetes/catalog.yaml
...
  - args:
        - proxy
        - sidecar
        - --domain
        - $(POD_NAMESPACE).svc.cluster.local
        - --proxyLogLevel=warning
        - --proxyComponentLogLevel=misc:error
        - --log_output_level=default:info
        - --concurrency
        - "2"
        env:
        - name: JWT_POLICY
          value: third-party-jwt
        - name: PILOT_CERT_PROVIDER
          value: istiod
        - name: CA_ADDR
          value: istiod.istio-system.svc:15012
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
  ...
        image: docker.io/istio/proxyv2:1.13.0
        name: istio-proxy




# 방법2 : namespace에 레이블을 추가하면 istiod (오퍼레이터)가 해당 namepsace의 pod spec에 자동으로 sidecar 설정을 주입
kubectl label namespace istioinaction istio-injection=enabled
kubectl get ns --show-labels


# mutatingwebhookconfiguration 확인
(⎈|kind-myk8s:N/A) ubuntu@ubuntu:/$ kubectl get mutatingwebhookconfigurations.admissionregistration.k8s.io
NAME                         WEBHOOKS   AGE
istio-revision-tag-default   4          4d11h # 특정 revision의 사이드카 주입 설정 관리
istio-sidecar-injector       4          4d11h # Istio는 각 애플리케이션 Pod에 Envoy 사이드카 프록시를 자동으로 주입
                                              ## 네임스페이스나 Pod에 istio-injection=enabled 라벨이 있어야 작동 


kubectl get mutatingwebhookconfiguration istio-sidecar-injector -o yaml

#
kubectl get cm -n istio-system istio-sidecar-injector -o yaml | kubectl neat
```

[##_Image|kage@bb89mr/btsNg8wQpYz/78BkwckBGxUn692TkwfDkK/img.png|CDM|1.3|{"originWidth":751,"originHeight":474,"style":"widthContent","filename":"CleanShot_2024-10-13_at_19.59.32.png"}_##]

```
#
cat services/catalog/kubernetes/catalog.yaml
kubectl apply -f services/catalog/kubernetes/catalog.yaml -n istioinaction

cat services/webapp/kubernetes/webapp.yaml 
kubectl apply -f services/webapp/kubernetes/webapp.yaml -n istioinaction

#
kubectl get pod -n istioinaction
NAME                     READY   STATUS    RESTARTS   AGE
catalog-6cf4b97d-jx8xw   2/2     Running   0          29s
webapp-7685bcb84-zlxmv   2/2     Running   0          29s

# catalog 디플로이먼트에서 파드 관련 spec
kubectl get deploy -n istioinaction catalog -o jsonpath="{.spec.template.spec}" | jq

# catalog 파드 관련 spec : 위 디플로이먼트와 파드 spec 을 비교해보자
kubectl get pod -n istioinaction -l app=catalog -o jsonpath="{.items[0].spec}" | jq


# 접속 테스트용 netshoot 파드 생성
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: netshoot
spec:
  containers:
  - name: netshoot
    image: nicolaka/netshoot
    command: ["tail"]
    args: ["-f", "/dev/null"]
  terminationGracePeriodSeconds: 0
EOF

# catalog 접속 확인
kubectl exec -it netshoot -- curl -s http://catalog.istioinaction/items/1 | jq
{
  "id": 1,
  "color": "amber",
  "department": "Eyewear",
  "name": "Elinor Glasses",
  "price": "282.00"
}

# webapp 접속 확인 : webapp 서비스는 다른 서비스에서 데이터를 집계해 브라우저에 시각적으로 표시한다. 
## 즉 webapp은 다른 백엔드 서비스의 파사드 facade 역할을 한다.
kubectl exec -it netshoot -- curl -s http://webapp.istioinaction/api/catalog/items/1 | jq
{
  "id": 1,
  "color": "amber",
  "department": "Eyewear",
  "name": "Elinor Glasses",
  "price": "282.00"
}
```

```
# 아래 방법 대신 임시 사용
kubectl port-forward -n istioinaction deploy/webapp 8080:8080
확인 후 CTRL+C 로 종료


# 우분투 22.04 minimal 버전으로 CLI 브라우저 설치 필요
sudo apt install w3m -y


# 
w3m http://localhost:8080
```

**\- CLI 브라우저 접속확인**

> w3m http://localhost:8080 을 통해 webapp 페이지 결과 출력

[##_Image|kage@cIjqOd/btsNiWCiURu/nUyRemKiEOsGNYgYiPKZGK/img.png|CDM|1.3|{"originWidth":1030,"originHeight":478,"style":"alignCenter","caption":"webapp 서비스는 다른 서비스에서 데이터를 집계하여 표시"}_##]

\- webapp 서비스는 다른 서비스에서 데이터를 집계해 브라우저에 시각적으로 표시해줌

\- kind 환경에서는 외부에서 서비스 접근을 위하여 **istio-ingressgateway** 서비스를 사용하여 내부 서비스 접근가능

\- Istio-ingressgateway의 경우  포트 80 의 경우 노드포트 30000 으로 설정되어 있으므로 외부에서 접근할 때

  http://192.168.9.248:3000 으로 웹페이지 접속 가능함

[##_Image|kage@ccwsww/btsNi5fHsjK/9yzXX95hTzSOKQAikBtNMK/img.png|CDM|1.3|{"originWidth":1501,"originHeight":79,"style":"alignCenter","caption":"istio-ingressgateway 서비스"}_##]

**\- 단, Iisto 의 Gateway 와 VirtualService 를 생성하여 webapp 서비스를 노출**

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: webapp-gateway
  namespace: istioinaction
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webapp-vs
  namespace: istioinaction
spec:
  hosts:
  - "*"
  gateways:
  - webapp-gateway
  http:
  - route:
    - destination:
        host: webapp
        port:
          number: 80
          

#
kubectl apply -f ingress-gateway.yaml


#
(⎈|kind-myk8s:default) ubuntu@ubuntu:~$ kubectl get gateway,vs -n istioinaction
NAME                                         AGE
gateway.networking.istio.io/webapp-gateway   4s

NAME                                           GATEWAYS             HOSTS   AGE
virtualservice.networking.istio.io/webapp-vs   ["webapp-gateway"]   ["*"]   4s
```

\- 외부 호스트에서 http://192.168.9.248:30000 (노드포트) 를 통해  webapp 서비스 접근

[##_Image|kage@cFhTYr/btsNgypbs5F/ET55lmjdjY9KuqAAcnw54k/img.png|CDM|1.3|{"originWidth":1864,"originHeight":1420,"style":"alignCenter","caption":"webapp 서비스가 다른 서비스의 데이터를 집계하여 시각화"}_##]

#### **\- Istio  복원력, 관찰가능성, 트래픽 제어**

[##_Image|kage@Izn3x/btsNjDwf0nL/rKv9SOPLYkTYCpTDKGVaTk/img.png|CDM|1.3|{"originWidth":1257,"originHeight":611,"style":"widthContent","filename":"image (2).png"}_##]

```
# istioctl proxy-status : 단축어 ps
docker exec -it myk8s-control-plane istioctl proxy-status
docker exec -it myk8s-control-plane istioctl ps

#
cat ch2/ingress-gateway.yaml
cat <<EOF | kubectl -n istioinaction apply -f -
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: webapp-gateway
  namespace: istioinaction
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webapp-vs
  namespace: istioinaction
spec:
  hosts:
  - "*"
  gateways:
  - webapp-gateway
  http:
  - route:
    - destination:
        host: webapp
        port:
          number: 80
EOF

#
kubectl get gw,vs -n istioinaction
NAME                                         AGE
gateway.networking.istio.io/webapp-gateway   48m

NAME                                           GATEWAYS             HOSTS   AGE
virtualservice.networking.istio.io/webapp-vs   ["webapp-gateway"]   ["*"]   48m

# istioctl proxy-status : 단축어 ps
docker exec -it myk8s-control-plane istioctl proxy-status
NAME                                                  CLUSTER        CDS        LDS        EDS        RDS        ECDS         ISTIOD                      VERSION
catalog-6cf4b97d-7zsrk.istioinaction                  Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-7df6ffc78d-qgtv6     1.17.8
istio-ingressgateway-996bc6bb6-s2rgz.istio-system     Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-7df6ffc78d-qgtv6     1.17.8
webapp-7685bcb84-wwqgg.istioinaction                  Kubernetes     SYNCED     SYNCED     SYNCED     SYNCED     NOT SENT     istiod-7df6ffc78d-qgtv6     1.17.8

ISTIOIGW=istio-ingressgateway-996bc6bb6-647tx.istio-system
WEBAPP=webapp-7685bcb84-nfntj.istioinaction

# istioctl proxy-config : 단축어 pc
docker exec -it myk8s-control-plane istioctl proxy-config all $ISTIOIGW
docker exec -it myk8s-control-plane istioctl proxy-config all $WEBAPP

docker exec -it myk8s-control-plane istioctl proxy-config listener $ISTIOIGW
docker exec -it myk8s-control-plane istioctl proxy-config route $ISTIOIGW
docker exec -it myk8s-control-plane istioctl proxy-config cluster $ISTIOIGW
docker exec -it myk8s-control-plane istioctl proxy-config endpoint $ISTIOIGW
docker exec -it myk8s-control-plane istioctl proxy-config log $ISTIOIGW

docker exec -it myk8s-control-plane istioctl proxy-config listener $WEBAPP
docker exec -it myk8s-control-plane istioctl proxy-config route $WEBAPP
docker exec -it myk8s-control-plane istioctl proxy-config cluster $WEBAPP
docker exec -it myk8s-control-plane istioctl proxy-config endpoint $WEBAPP
docker exec -it myk8s-control-plane istioctl proxy-config log $WEBAPP

# envoy 가 사용하고 있는 인증서 정보 확인
docker exec -it myk8s-control-plane istioctl proxy-config secret $ISTIOIGW
docker exec -it myk8s-control-plane istioctl proxy-config secret $WEBAPP


#
docker exec -it myk8s-control-plane istioctl proxy-config routes deploy/istio-ingressgateway.istio-system
NAME          DOMAINS     MATCH                  VIRTUAL SERVICE
http.8080     *           /*                     webapp-virtualservice.istioinaction
              *           /stats/prometheus*
              *           /healthz/ready*


# istio-ingressgateway 서비스 NodePort 변경 및 nodeport 30000로 지정 변경
kubectl get svc,ep -n istio-system istio-ingressgateway
kubectl patch svc -n istio-system istio-ingressgateway -p '{"spec": {"type": "NodePort", "ports": [{"port": 80, "targetPort": 8080, "nodePort": 30000}]}}'
kubectl get svc -n istio-system istio-ingressgateway

# istio-ingressgateway 서비스 externalTrafficPolicy 설정 : ClientIP 수집 확인
kubectl patch svc -n istio-system istio-ingressgateway -p '{"spec":{"externalTrafficPolicy": "Local"}}'
kubectl describe svc -n istio-system istio-ingressgateway

#
kubectl stern -l app=webapp -n istioinaction
kubectl stern -l app=catalog -n istioinaction

#
curl -s http://127.0.0.1:30000/api/catalog | jq
curl -s http://127.0.0.1:30000/api/catalog/items/1 | jq
curl -s http://127.0.0.1:30000/api/catalog -I | head -n 1

# webapp 반복 호출
while true; do curl -s http://127.0.0.1:30000/api/catalog/items/1 ; sleep 1; echo; done
while true; do curl -s http://127.0.0.1:30000/api/catalog -I | head -n 1 ; date "+%Y-%m-%d %H:%M:%S" ; sleep 1; echo; done
while true; do curl -s http://127.0.0.1:30000/api/catalog -I | head -n 1 ; date "+%Y-%m-%d %H:%M:%S" ; sleep 0.5; echo; done
```

#### **\- Istio 관찰가능성**

[##_Image|kage@CyAR4/btsNiMOfqms/rxBM6HBql6ait73sga31u1/img.png|CDM|1.3|{"originWidth":542,"originHeight":346,"style":"widthContent","filename":"CleanShot_2025-04-05_at_16.31.05.png"}_##]

```
# NodePort 변경 및 nodeport 30001~30003으로 변경 : prometheus(30001), grafana(30002), kiali(30003), tracing(30004)
kubectl patch svc -n istio-system prometheus -p '{"spec": {"type": "NodePort", "ports": [{"port": 9090, "targetPort": 9090, "nodePort": 30001}]}}'
kubectl patch svc -n istio-system grafana -p '{"spec": {"type": "NodePort", "ports": [{"port": 3000, "targetPort": 3000, "nodePort": 30002}]}}'
kubectl patch svc -n istio-system kiali -p '{"spec": {"type": "NodePort", "ports": [{"port": 20001, "targetPort": 20001, "nodePort": 30003}]}}'
kubectl patch svc -n istio-system tracing -p '{"spec": {"type": "NodePort", "ports": [{"port": 80, "targetPort": 16686, "nodePort": 30004}]}}'

# Prometheus 접속 : envoy, istio 메트릭 확인
w3m http://127.0.0.1:30001

# Grafana 접속
w3m http://127.0.0.1:30002

# Kiali 접속 1 : NodePort
w3m http://127.0.0.1:30003

# (옵션) Kiali 접속 2 : Port forward
kubectl port-forward deployment/kiali -n istio-system 20001:20001 &
w3m http://127.0.0.1:20001

# tracing 접속 : 예거 트레이싱 대시보드
w3m http://127.0.0.1:30004
```

**\- 노드포트 변경 결과**

[##_Image|kage@csvjJT/btsNi1EoHag/qV9rslMrhidhAwcLC7Oc6k/img.png|CDM|1.3|{"originWidth":1456,"originHeight":259,"style":"widthContent"}_##]

**\- 프로메테우스 접속 : http://192.168.9.248:30001**

[##_Image|kage@cVvZvw/btsNi7YSFSO/bzgiKLr4dMFAhpgjnpkzAk/img.png|CDM|1.3|{"originWidth":2841,"originHeight":1719,"style":"widthContent"}_##]

**\- 그라파타 접속 : http://192.168.9.248:30002** 

대시보드 - Istio Service Dashboard ⇒ 상단 Service (webapp.. 선택)

[##_Image|kage@cQa7sx/btsNi4A6SKg/fzL1v3Hb3kwyKsSAO18S20/img.png|CDM|1.3|{"originWidth":3811,"originHeight":1882,"style":"alignCenter"}_##]

**\- kiali 확인 : http://192.168.9.248:30003**

메트릭(프로메테우스)과 트레이싱(zipkin, jaeger, tempo)을 수집하여 해당 정보를 기반으로 시각화

[##_Image|kage@dGolu8/btsNiJdc1IF/dKW72ryOYikJ8bLskOiyEk/img.png|CDM|1.3|{"originWidth":759,"originHeight":661,"style":"widthContent","caption":"https://kiali.io/docs/architecture/architecture/","filename":"image (1).png"}_##]

\- Namespace 를 istioincation 로 선택 후 Graph (Traffic, Versioned app graph) 에서 Display 옵션 중 ‘Traffic Distribution’ 과 ‘Traffic Animation’ 활성화! , Service nods, Security 체크

[##_Image|kage@zMz51/btsNktTWslG/b6skgrDJL32dKP32Mznze1/img.png|CDM|1.3|{"originWidth":2836,"originHeight":1684,"style":"widthContent"}_##]

---

#### \- Istio catalog에 의도적으로 500에러를 재현하고 retry로 복원력 높이기

[##_Image|kage@dnlEzp/btsNiZNiTVD/4hM0imkbZOy7DIhp5DoMFk/img.png|CDM|1.3|{"originWidth":398,"originHeight":211,"style":"widthContent","filename":"CleanShot_2025-04-05_at_16.44.25.png"}_##]

**\- bin/chaos.sh {에러코드} {빈도} - chaos.sh 500 50 (500에러를 50% 빈도로 재현)**

```
#!/usr/bin/env bash

if [ $1 == "500" ]; then

    POD=$(kubectl get pod | grep catalog | awk '{ print $1 }')
    echo $POD

    for p in $POD; do
        if [ ${2:-"false"} == "delete" ]; then
            echo "Deleting 500 rule from $p"
            kubectl exec -c catalog -it $p -- curl  -X POST -H "Content-Type: application/json" -d '{"active":
        false,  "type": "500"}' localhost:3000/blowup
        else
            PERCENTAGE=${2:-100}
            kubectl exec -c catalog -it $p -- curl  -X POST -H "Content-Type: application/json" -d '{"active":
            true,  "type": "500",  "percentage": '"${PERCENTAGE}"'}' localhost:3000/blowup
            echo ""
        fi
    done


fi
```

```
#
docker exec -it myk8s-control-plane bash
----------------------------------------
# istioinaction 로 네임스페이스 변경
cat /etc/kubernetes/admin.conf
kubectl config set-context $(kubectl config current-context) --namespace=istioinaction
cat /etc/kubernetes/admin.conf

cd /istiobook/bin/ 
./chaos.sh 500 100 # 모니터링 : kiali, grafana, tracing

./chaos.sh 500 50 # 모니터링 : kiali, grafana, tracing


kubectl config set-context $(kubectl config current-context) --namespace=default
cat /etc/kubernetes/admin.conf
----------------------------------------
```

#### **※ Istio control plane 제공 기능**

-   운영자가 원하는 라우팅/회복력 routing/resilience 동작을 지정할 수 있는 API
-   데이터 플레인 설정을 위한 API
-   데이터 플레인에 대한 서비스 검색 추상화 service discovery abstraction
-   사용자 정책 specifying usage policies 지정 API
-   인증서 발급 및 순환 Certificate issuance and rotation
-   워크로드 ID 할당 Workload identity assignment
-   통합 원격 측정 컬렉션 Unified telemetry collection
-   서비스 프록시 사이드카 주입 Service-proxy sidecar injection
-   네트워크 경계 지정 및 접근 방법 Specification of network boundaries and how to access them

#### **\- Bookinfo sample application 배포 실습**

[##_Image|kage@dV1X0t/btsNjM7xYPd/oMokkTPcUxinQ6Y5L1PUk0/img.png|CDM|1.3|{"originWidth":662,"originHeight":453,"style":"widthContent","filename":"image (4).png"}_##]

```
#
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

# 확인 : 서비스 어카운트(sa)는 spiffe 에 svid 에 사용됨
kubectl get all,sa

# product 웹 접속 확인
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

# productpage 파드 로그
kubectl logs -l app=productpage -c istio-proxy --tail=-1
kubectl logs -l app=productpage -c productpage -f
```

[##_Image|kage@cdFX8s/btsNi8Dz08G/qX6Gimlq0uVyIILsvONeXk/img.png|CDM|1.3|{"originWidth":1909,"originHeight":1639,"style":"alignCenter","caption":"http://192.168.9.248:30000/productpage"}_##][##_Image|kage@n5vAu/btsNjDwmZJI/njFXFFJGBbRqTOKpl89Yu1/img.png|CDM|1.3|{"originWidth":3802,"originHeight":1825,"style":"alignCenter","caption":"kiali 를 통해 traffic graph 모니터링"}_##]

##  **최신버전 설치실습**  

---

> **실습 환경 : docker (kind - k8s 1.32.2) , istio 1.25.1**

### **\- kind : k8s(1.23.2) 배포**

```
# 
kind create cluster --name myk8s --image kindest/node:v1.32.2 --config - <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30000 # Sample Application
    hostPort: 30000
  - containerPort: 30001 # Prometheus
    hostPort: 30001
  - containerPort: 30002 # Grafana
    hostPort: 30002
  - containerPort: 30003 # Kiali
    hostPort: 30003
  - containerPort: 30004 # Tracing
    hostPort: 30004
  - containerPort: 30005 # kube-ops-view
    hostPort: 30005
networking:
  podSubnet: 10.10.0.0/16
  serviceSubnet: 10.200.1.0/24
EOF

# 설치 확인
docker ps

# 노드에 기본 툴 설치
docker exec -it myk8s-control-plane sh -c 'apt update && apt install tree psmisc lsof wget bridge-utils net-tools dnsutils tcpdump ngrep iputils-ping git vim -y'
```

### **\- 편의성 툴 설치**

```
# (옵션) kube-ops-view
helm repo add geek-cookbook https://geek-cookbook.github.io/charts/
helm install kube-ops-view geek-cookbook/kube-ops-view --version 1.2.2 --set service.main.type=NodePort,service.main.ports.http.nodePort=30005 --set env.TZ="Asia/Seoul" --namespace kube-system
kubectl get deploy,pod,svc,ep -n kube-system -l app.kubernetes.io/instance=kube-ops-view

## kube-ops-view 접속 URL 확인
open "http://localhost:30005/#scale=1.5"
open "http://localhost:30005/#scale=1.3"

# (옵션) metrics-server
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm install metrics-server metrics-server/metrics-server --set 'args[0]=--kubelet-insecure-tls' -n kube-system
kubectl get all -n kube-system -l app.kubernetes.io/instance=metrics-server
```

### **\- Istio 1.25.1 설치**

```
# istioctl 설치
export ISTIOV=1.25.1
echo 'export ISTIOV=1.25.1' | sudo tee -a /root/.bashrc

curl -s -L https://istio.io/downloadIstio | ISTIO_VERSION=$ISTIOV sh -
tree istio-$ISTIOV -L 2 # sample yaml 포함
cp istio-$ISTIOV/bin/istioctl /usr/local/bin/istioctl
istioctl version --remote=false


# default 프로파일 배포
cd istio-$ISTIOV
cat <<EOF | istioctl install -y -f - 
# IOP configuration used to install the demo profile without gateways.
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  profile: demo
  components:
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
    egressGateways:
    - name: istio-egressgateway
      enabled: false
EOF
```

### **\- 공통 설정 : addon 등 설치**

```
# 설치 확인 : istiod(데몬, 컨트롤플레인), istio-ingressgateway, crd 등
kubectl get all,svc,ep,sa,cm,secret,pdb -n istio-system
kubectl get crd | grep istio.io | sort

# istio-ingressgateway 서비스 NodePort 변경 및 nodeport 30000로 지정 변경
kubectl patch svc -n istio-system istio-ingressgateway -p '{"spec": {"type": "NodePort", "ports": [{"port": 80, "targetPort": 8080, "nodePort": 30000}]}}'
kubectl get svc -n istio-system istio-ingressgateway

# istio-ingressgateway 서비스 externalTrafficPolicy 설정 : ClientIP 수집 확인 용도
kubectl patch svc -n istio-system istio-ingressgateway -p '{"spec":{"externalTrafficPolicy": "Local"}}'
kubectl describe svc -n istio-system istio-ingressgateway


# default 네임스페이스에 istio-proxy sidecar 주입 설정 - Docs
kubectl label namespace default istio-injection=enabled
kubectl get ns --show-labels


# addon 설치
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system

#
kubectl get pod,svc -n istio-system

# NodePort 변경 및 nodeport 30001~30003으로 변경 : prometheus(30001), grafana(30002), kiali(30003), tracing(30004)
kubectl patch svc -n istio-system prometheus -p '{"spec": {"type": "NodePort", "ports": [{"port": 9090, "targetPort": 9090, "nodePort": 30001}]}}'
kubectl patch svc -n istio-system grafana -p '{"spec": {"type": "NodePort", "ports": [{"port": 3000, "targetPort": 3000, "nodePort": 30002}]}}'
kubectl patch svc -n istio-system kiali -p '{"spec": {"type": "NodePort", "ports": [{"port": 20001, "targetPort": 20001, "nodePort": 30003}]}}'
kubectl patch svc -n istio-system tracing -p '{"spec": {"type": "NodePort", "ports": [{"port": 80, "targetPort": 16686, "nodePort": 30004}]}}'

# Prometheus 접속 : envoy, istio 메트릭 확인
open http://127.0.0.1:30001

# Grafana 접속
open http://127.0.0.1:30002

# Kiali 접속 1 : NodePort
open http://127.0.0.1:30003

# (옵션) Kiali 접속 2 : Port forward
kubectl port-forward deployment/kiali -n istio-system 20001:20001 &
open http://127.0.0.1:20001

# tracing 접속 : 예거 트레이싱 대시보드
open http://127.0.0.1:30004
```

### **\- Bookinfo sample application 배포**

```
#
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

# 확인 : 서비스 어카운트(sa)는 spiffe 에 svid 에 사용됨
kubectl get all,sa

# product 웹 접속 확인
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

# productpage 파드 로그
kubectl logs -l app=productpage -c istio-proxy --tail=-1
kubectl logs -l app=productpage -c productpage -f
```

### **\- 서비스 외부 노출**

```
# Istio Gateway/VirtualService 설정
cat samples/bookinfo/networking/bookinfo-gateway.yaml
apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 8080
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
          
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

# Istio Gateway/VirtualService 설정 확인
kubectl get gw,vs
istioctl proxy-status

# productpage 파드의 istio-proxy 로그 확인 Access log 가 출력 - Default access log format : 링크
kubectl logs -l app=productpage -c istio-proxy -f
kubectl stern -l app=productpage

# productpage 웹 접속 : 새로고침
open http://127.0.0.1:30000/productpage
curl -v -s http://127.0.0.1:30000/productpage | grep -o "<title>.*</title>"

# 반복 접속
for i in {1..10};  do curl -s http://127.0.0.1:30000/productpage | grep -o "<title>.*</title>" ; done
for i in {1..100}; do curl -s http://127.0.0.1:30000/productpage | grep -o "<title>.*</title>" ; done

while true; do curl -s http://127.0.0.1:30000/productpage | grep -o "<title>.*</title>" ; echo "--------------" ; sleep 1; done
while true; do curl -s http://127.0.0.1:30000/productpage | grep -o "<title>.*</title>" ; echo "--------------" ; sleep 0.5; done
while true; do curl -s http://127.0.0.1:30000/productpage | grep -o "<title>.*</title>" ; echo "--------------" ; sleep 0.1; done
```

**\- kiali 서비스**

[##_Image|kage@cWp4pJ/btsNhrJTNMx/MUR1qiHRqTPPOVImoyE1pK/img.png|CDM|1.3|{"originWidth":1898,"originHeight":1055,"style":"alignCenter"}_##]

#### **\- (참고) istio-ingressgateway 에 istio-proxy 에도 로깅 변경 해보자**

```
kubectl exec -it deploy/istio-ingressgateway -n istio-system -- curl -X POST http://localhost:15000/logging
kubectl exec -it deploy/istio-ingressgateway -n istio-system -- curl -X POST http://localhost:15000/logging?http=debug
kubectl exec -it deploy/istio-ingressgateway -n istio-system -- curl -X POST http://localhost:15000/logging?http=info
```

#### **\- (참고) istio-proxy 파드에 envoy 컨테이너 admin 페이지 접속**

```
# istio-proxy 파드에 envoy 컨테이너 admin 접속 포트 포워딩 설정
kubectl port-forward deployment/deploy-websrv 15000:15000 &

# envoy 컨테이너 admin 페이지 접속
open http://localhost:15000
```

**※ 실습 완료 후 삭제** 

```
kind delete cluster --name myk8s
```

---

## 

---

## **\[도전과제1\]** 

#### **☞ Istio 관리(설정, 업그레이드 등)에 편리성을 제공하는 Sail Operator 설치 및 사용**  

[##_Image|kage@qXIzt/btsNiL3eY6X/nYfjUTJ6oVvwrVSpkk1MFK/img.png|CDM|1.3|{"originWidth":917,"originHeight":575,"style":"widthContent","filename":"CleanShot_2025-04-11_at_13.09.41.png"}_##]

### **\- sail-operator  설치**

```
# Install the Sail Operator using Helm
helm repo add sail-operator https://istio-ecosystem.github.io/sail-operator
helm repo update
kubectl create namespace sail-operator
helm install sail-operator sail-operator/sail-operator --version 1.0.0 -n sail-operator

# 설치확인
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "sail-operator" chart repository
...Successfully got an update from the "metrics-server" chart repository
...Successfully got an update from the "geek-cookbook" chart repository
...Successfully got an update from the "prometheus-community" chart repository
Update Complete. ⎈Happy Helming!⎈
namespace/sail-operator created
NAME: sail-operator
LAST DEPLOYED: Sat Apr 12 14:59:46 2025
NAMESPACE: sail-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None

# 서비스 상태 확인
(⎈|kind-myk8s:N/A) ubuntu@ubuntu:/$ kubectl get pods -n sail-operator
NAME                             READY   STATUS    RESTARTS   AGE
sail-operator-86555cd677-jbkrr   1/1     Running   0          108s
```

**\- Check the operator pod is running:**

```
$ kubectl get pods -n sail-operator
NAME                             READY   STATUS    RESTARTS   AGE
sail-operator-56bf994f49-j67ft   1/1     Running   0          87s
```

**\- Create Istio and IstioRevisionTag resources**

```
cat <<EOF | kubectl apply -f-
apiVersion: sailoperator.io/v1
kind: Istio
metadata:
  name: default
spec:
  namespace: istio-system
  updateStrategy:
    type: RevisionBased
    inactiveRevisionDeletionGracePeriodSeconds: 30
  version: v1.24.2
---
apiVersion: sailoperator.io/v1
kind: IstioRevisionTag
metadata:
  name: default
spec:
  targetRef:
    kind: Istio
    name: default
EOF
```

**\- Note that the IstioRevisionTag has a target reference to the Istio resource with the name default**

[##_Image|kage@b9dkIx/btsNiL3jidE/vQ9uqhFt4fYvzxw2UjbQYk/img.png|CDM|1.3|{"originWidth":1489,"originHeight":519,"style":"widthContent"}_##]

Note that the IstioRevisionTag status is NotReferencedByAnything. This is because there are currently no resources using the revision default-v1-24-2.

#### **\- Deploy sample application**

[https://istio.io/latest/blog/2025/sail-operator-ga/](https://istio.io/latest/blog/2025/sail-operator-ga/)

 [Sail Operator 1.0.0 released: manage Istio with an operator

Dive in into the basics of the Sail Operator and check out an example to see how easy it is to use it to manage Istio.

istio.io](https://istio.io/latest/blog/2025/sail-operator-ga/)

```
# Install the Sail Operator using Helm

helm repo add sail-operator https://istio-ecosystem.github.io/sail-operator
helm repo update
kubectl create namespace sail-operator
helm install sail-operator sail-operator/sail-operator --version 1.0.0 -n sail-operator

# The operator is now installed in your cluster:
NAME: sail-operator
LAST DEPLOYED: Tue Mar 18 12:00:46 2025
NAMESPACE: sail-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None

# Check the operator pod is running
root@myk8s-control-plane:/# kubectl get pods -n sail-operator
NAME                             READY   STATUS    RESTARTS   AGE
sail-operator-86555cd677-jbkrr   1/1     Running   0          9h


# Create Istio and IstioRevisionTag resources
kubectl create ns istio-system
cat <<EOF | kubectl apply -f-
apiVersion: sailoperator.io/v1
kind: Istio
metadata:
  name: default
spec:
  namespace: istio-system
  updateStrategy:
    type: RevisionBased
    inactiveRevisionDeletionGracePeriodSeconds: 30
  version: v1.24.2
---
apiVersion: sailoperator.io/v1
kind: IstioRevisionTag
metadata:
  name: default
spec:
  targetRef:
    kind: Istio
    name: default
EOF

# Check the state of the resources created:

# istiod pods are running
(⎈|kind-myk8s:N/A) ubuntu@ubuntu:~$ kubectl get pods -n istio-system istiod-default-v1-24-2-6c4dd448b6-ntm4j
NAME                                      READY   STATUS    RESTARTS   AGE
istiod-default-v1-24-2-6c4dd448b6-ntm4j   1/1     Running   0          2m24s
(⎈|kind-myk8s:N/A) ubuntu@ubuntu:~$

# Istio resource created
(⎈|kind-myk8s:N/A) ubuntu@ubuntu:~$ kubectl get istio
NAME      REVISIONS   READY   IN USE   ACTIVE REVISION   STATUS    VERSION   AGE
default   1           1       0        default-v1-24-2   Healthy   v1.24.2   2m30s

IstioRevisionTag resource created

(⎈|kind-myk8s:N/A) ubuntu@ubuntu:~$ kubectl get istiorevisiontag
NAME      STATUS        IN USE   REVISION   AGE
default   RefNotFound   True                8h


# Deploy sample application
# Create a namespace and label it to enable Istio injection:

kubectl create namespace sample
kubectl label namespace sample istio-injection=enabled

# kubectl get istiorevisiontag
NAME      STATUS    IN USE   REVISION          AGE
default   Healthy   True     default-v1-24-2   6m24s

# Deploy the sample application
(⎈|kind-myk8s:N/A) ubuntu@ubuntu:~$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.25/samples/sleep/sleep.yaml -n sample
serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created

# Confirm the proxy version of the sample app matches the control plane version:
root@myk8s-control-plane:/# istioctl proxy-status | grep sleep-868c754c4b-d4h9m.sample
sleep-868c754c4b-d4h9m.sample                          Kubernetes     SYNCED (77s)     SYNCED (77s)     SYNCED (76s)     SYNCED (77s)     IGNORED     istiod-9c54bfcbf-hww5m                                       1.25.1
```

#### **\- Upgrade the Istio control plane to version 1.24.3**

```
root@myk8s-control-plane:/# kubectl patch istio default -n istio-system --type='merge' -p '{"spec":{"version":"v1.24.3"}}'
istio.sailoperator.io/default patched
root@myk8s-control-plane:/#
root@myk8s-control-plane:/#
root@myk8s-control-plane:/#
root@myk8s-control-plane:/# kubectl get istio
NAME      REVISIONS   READY   IN USE   ACTIVE REVISION   STATUS           VERSION   AGE
default   1           0       0        default-v1-24-3   IstiodNotReady   v1.24.3   12m
root@myk8s-control-plane:/#
root@myk8s-control-plane:/#
root@myk8s-control-plane:/# kubectl get istiorevisiontag
NAME      STATUS        IN USE   REVISION   AGE
default   RefNotFound   True                9h
root@myk8s-control-plane:/#
root@myk8s-control-plane:/#  kubectl get istiorevision
NAME              TYPE   READY   STATUS           IN USE   VERSION   AGE
default-v1-24-3          False   IstiodNotReady   False    v1.24.3   17s
root@myk8s-control-plane:/#
root@myk8s-control-plane:/#
root@myk8s-control-plane:/#  kubectl get istiorevision
NAME              TYPE   READY   STATUS           IN USE   VERSION   AGE
default-v1-24-3          False   IstiodNotReady   False    v1.24.3   24s

root@myk8s-control-plane:/#  kubectl rollout restart deployment -n sample
deployment.apps/sleep restarted

root@myk8s-control-plane:/#  kubectl get istiorevision
NAME              TYPE   READY   STATUS    IN USE   VERSION   AGE
default-v1-24-3          True    Healthy   False    v1.24.3   116s
root@myk8s-control-plane:/#
root@myk8s-control-plane:/#
root@myk8s-control-plane:/#
root@myk8s-control-plane:/# kubectl get istio
NAME      REVISIONS   READY   IN USE   ACTIVE REVISION   STATUS    VERSION   AGE
default   1           1       0        default-v1-24-3   Healthy   v1.24.3   14m
```

## **\[도전과제 2\]** 

#### **☞ Istio Getting started with the Kubernetes Gateway API**

사전 지식 : K8S Ingress → K8S Gateway API 흐름 , Istio 전용의 Ingress GW API → K8S 표준 GW API(for Service Mesh) 흐름










