# [3] Static Pod 생성하기

일반적으로 kubectl 명령어를 실행하면 그 명령어가 API 서버에 전달되어 명령어 실행이 이루어 지는데 Static Pod의 경우에는 API 서버에 명령어가 전달되지 않는다.

바로 각 워커노드에 있는 kubelet에 전달한다.

## ❓Static Pod 생성하기

- Configure kubelet hosting to start a pod on the node
- TASK: 
    - node: hk8s-w1
    - pod Name: web
    - image: nginx

### 실습

```docker
# image가 nginx이고 이름이 web인 pod를 테스트로 실행하고 그 yaml파일 출력
[user@console ~]$ kubectl run web --image=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
	creationTimestamp: null
	labels:
		run: web
	name: web
spec: 
	containers:
	- image: nginx
		name: web
		resources: {}
	dnsPolicy: ClusterFirst
	restartPolicy: Always
Status: {}

# hk8s-w1 노드 접속
[user@console ~]$ ssh hk8s-w1

# root권한으로 전환
[user@hk8s-w1 ~]$ sudo -i

# kubelet의 config 파일 확인
# 기본 Pod 경로가 /etc/kubernetes/manifests 디렉토리로 지정되어있는거 확인
[root@hk8s-w1 ~]# cat /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests

[root@hk8s-w1 ~]# cd /etc/kubernetes/manifests

[root@hk8s-w1 manifests]# ls

# 아까 복사해놓은 yaml 템플릿을 약간 수정한 다음 저장
[root@hk8s-w1 manifests]# vi web-pod.yaml
apiVersion: v1
kind: Pod
metadata:
	name: web
spec: 
	containers:
	- image: nginx
		name: web

:wq

# 파일을 생성하자마자 node에 있는 kubelet이 pod를 동작시킨다.
# 이 파일을 삭제하면 pod도 같이 삭제된다.

[root@hk8s-w1 manifests]# exit

[user@hk8s-w1 manifests]$ exit

[user@console ~]$ kubectl get pods
web-hk8s-w1 라는 이름의 pod가 구동중인 것 확인

```

## 기출문제 (60번)

Configure the kubelet systemd- managed service, on the node labelled with name=wk8s -node-1, to launch a pod containing a single container of Image httpd named webtool automatically. Any spec files required should be placed in the /etc/kubernetes/manifests directory on the node

- 답안  
    ```jsx
    ssh wk8s-node-1
    
    ```
    
    ```jsx
    cat /var/lib/kubelet/config.yaml
    --> 여기서 staticPodPath가 /etc/kubernetes/manifests로 되어있는지 확인
    
    ```
    
    ```jsx
    cd /etc/kubernetes/manifests
    
    ```
    
    ```jsx
    kubectl run webtool --image=httpd --dry-run=client -o yamlvi webtool-pod.yaml
    
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        name: wk8s-node-1
      name: webtool
    spec:
      containers:
      - image: httpd
        name: webtool
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    
    :wq
    
    ```
