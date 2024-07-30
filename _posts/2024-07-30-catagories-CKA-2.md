---
title: "[CKA]  Pod 생성하기 "
excerpt: "Pod 생성하기"

categories:
  - CKA
tags:
  - [cka,pod]

permalink: /cka/2024-07-30-categories-CKA-2.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---


# [2] Pod 생성하기

## ❓Pod 생성하기

- 클러스터: k8s
- Create a new namespace and create a pod in the namespace. 
    - namespace name: ecommerce
    - Pod Name: eshop-main
    - image: nginx:1.17
    - env: DB=mysql

### 실습

```docker
# 클러스터 변경
[user@console ~]$ kubectl config use-context k8s

# namespace 생성
[user@console ~]$ kubectl create namespace ecommerce

# 생성된 namespace 확인
[user@console ~]$ kubectl get namespaces

# pod 생성
# 엔터를 누르기 전에 --dry-run=client 옵션으로 해당 명령어가 문제없는 명령어인지 확인
[user@console ~]$ kubectl run eshop-main --image=nginx:1.17 --env=DB=mysql --namespace=ecommerce --dry-run=client

# 정상동작 확인 후 --dry-client 옵션 지운 후 명령어 실행
[user@console ~]$ kubectl run eshop-main --image=nginx:1.17 --env=DB=mysql --namespace=ecommerce

# Pod 생성됬는지 확인
[user@console ~]$ kubectl get pods --namespace ecommerce


```

## 기출문제 (2번)

Create 2 nginx image pods in which one of them is labelled with env=prod and another one labelled with env=dev and verify the same

- 답안
    
    ```jsx
    kubectl run nginx-prod --image=nginx --labels=env=prod
    
    ```
    
    ```jsx
    kubectl run nginx-dev --image=nginx --labels=env=dev
    
    ```
    
    ```jsx
    # 확인
    kubectl get pods
    
    kubectl get pod nginx-prod -o yaml
    --> labels부분 확인
    
    kubectl get pod nginx-dev -o yaml
    --> labels부분 확인
    
    ```

## 기출문제 (3번)

Create an nginx pod and list the pod with different levels of verbosity

- 답안
    
    ```jsx
    kubectl run nginx --image=nginx
    
    ```
    
    ```jsx
    kubectl get pod nginx --v=7
    
    ```
    
    ```jsx
    kubectl get pod nginx --v=8
    
    ```
    
    ```jsx
    kubectl get pod nginx --v=9
    
    ```
    
    
    - 참고 (verbosity)
    
    [https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/)

## 기출문제 (4번)

Create a pod with environment variables as var1=value1.Check the environment variable in pod

- 답안
    
    ```jsx
    kubectl run test-pod --image=nginx --env=var1=value1
    
    ```
    
    ```jsx
    # 확인
    kubectl get pods
    
    kubectl describe pod test-pod
    --> env 부분 확인
    
    ```

## 기출문제 (6번)

List "nginx-dev" and "nginx-prod" pod and delete those pods

- 답안
    
    ```jsx
    # 모든 네임스페이스에 있는 pod 확인
    kubectl get pods -A
    
    nginx-dev, nginx-prod pod 있는 것 확인
    
    ```
    
    ```jsx
    kubectl delete pod nginx-dev
    
    ```
    
    ```jsx
    kubectl delete pod nginx-prod
    
    ```

## 기출문제 (8번)

Create a pod as follows:

- Name: non-persistent-redis
- container Image: redis
- Volume with name: cache-control
- Mount path: /data/redis

The pod should launch in the staging namespace and the volume must not be persistent.

- 답안
    
    ```jsx
    kubectl create namespace staging 
    
    ```
    
    ```jsx
    # YAML 파일 생성
    vi volume.yaml
    
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: non-persistent-redis
      name: non-persistent-redis
      namespace: staging
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
        volumeMounts:
        - mountPath: /data/redis
          name: cache-control
      volumes:
      - name: cache-control
        emptyDir: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    
    :wq
    
    ```
    
    ```jsx
    kubectl apply -f volume.yaml
    
    ```

## 기출문제 (9번)

Create a pod with i traffic on port 80

- 답안
    
    ```jsx
    kubectl run nginx --image=nginx --port=80
    
    ```

## 기출문제 (11번)

Create a file: /opt/KUCC00302/kucc00302.txt that lists all pods that implement service baz in namespace development. The format of the file should be one pod name per line.

- 답안
    
    ```jsx
    # baz 라는 이름의 service가 있는지 확인
    kubectl get svc baz -n development
    
    ```
    
    ```jsx
    kubectl describe svc baz -n development
    --> 확인해보면 라벨이 name=foo인 것들에 대해 서비스로 묶고 있다.
    
    ```
    
    ```jsx
    kubectl get pods -l name=foo -n development
    
    ```
    
    ```jsx
    kubectl get pods -l name=foo -n development -o NAME > /opt/KUCC00302/kucc00302.txt
    
    ```

## 기출문제 (12번)

Create a pod named kucc8 with a single app container for each of the following images running inside (there may be between 1 and 4 images specified): nginx + redis + memcached.

- 답안
    
    ```jsx
    kubectl run kucc8 --image=nginx --dry-run=client -o yaml
    --> yaml 파일 복사
    
    ```
    
    ```jsx
    vi kucc.yaml
    
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: kucc8
      name: kucc8
    spec:
      containers:
      - image: nginx
        name: nginx
      - image: redis
        name: redis
      - image: memcached
        name: memcached
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    
    :wq
    
    ```
    
    ```jsx
    kubectl apply -f kucc.yaml
    
    ```
    
    ```jsx
    # 확인
    kubectl get pods
    
    kubectl describe pod kucc8
    --> 컨테이너 3개 생성된 것 확인
    
    ```

## 기출문제 (18번)

Check the image version in pod name nginx without the describe command

- 답안
    
    ```jsx
    kubectl get pod nginx -o jsonpath='{.spec.container[].image}{"\\n"}'
    
    ```

## 기출문제 (19번)

Print pod name and start time to "/opt/pod-status" file

- 답안
    
    ```jsx
    # json 파일로 확인 후 start time이 어디에 속해있는지 확인
    kubectl get pods -o json
    
    ```
    
    ```jsx
    kubectl get pods -o jsonpath='{range.items[*]}{.metadata.name}{"\\t"}{.status.startTime}{"\\n"}{end}’ > /opt/pod-status
    
    ```

## 기출문제 (23번)

Get IP address of the pod "nginx-dev”

- 답안
    
    ```jsx
    kubectl get pod nginx-dev -o jsonpath='{.status.podIP}{"\\n"}'
    
    ```

## 기출문제 (24번)

List all the pods sorted by name

- 답안
    
    ```jsx
    kubectl get pods --sort-by=.metadata.name
    
    ```

## 기출문제 (26번)

List the nginx pod with custom columns POD\_NAME and POD\_STATUS

- 답안
    
    ```jsx
    kubectl get pod -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"
    
    ```
    
    
    - 참고
    
    [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## 기출문제 (27번)

List all the pods sorted by created timestamp

- 답안
    
    ```jsx
    kubectl get pods --sort-by=.metadata.creationTimestamp
    
    ```

## 기출문제 (29번)

Get list of all pods in all namespaces and write it to file "/opt/pods-list.yaml”

- 답안
    
    ```jsx
    kubectl get pods -A > /opt/pods-list.yaml
    
    ```

## 기출문제 (34번)

Get list of all the pods showing name and namespace with a jsonpath expression.

- 답안
    
    ```jsx
    kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\\t"}{.metadata.namespace}{"\\n"}{end}'
    
    ```

## 기출문제 (38번)

Create a busybox pod and add "sleep 3600" command

- 답안
    
    ```jsx
    kubectl run busybox --image=busybox -- /bin/sh -c "sleep 3600"
    
    ```

## 기출문제 (42번)

Create a nginx pod with label env=test in engineering namespace

- 답안
    
    ```jsx
    kubectl create namespace engineering
    
    ```
    
    ```jsx
    kubectl run nginx --image=nginx --labels=env=test -n engineering
    
    ```

## 기출문제 (44번)

Check the Image version of nginx-dev pod using jsonpath

- 답안
    
    ```jsx
    kubectl get pod nginx-dev -o json
    
    ```
    
    ```jsx
    kubectl get pod nginx-dev -o jsonpath='{.spec.containers[].image}{"\\n"}'
    
    ```

## 기출문제 (57번)

List all the pods showing name and namespace with a json path expression

- 답안
    
    ```jsx
    kubectl get pods -A -o=jsonpath='{range .items[*]}{.metadata.name}{"\\t"}{.metadata.namespace}{"\\n"}{end}'
    
    ```

## 기출문제 (59번)

Create a pod as follows: Name: mongo Using Image: mongo In a new Kubernetes namespace named: my-website

- 답안
    
    ```jsx
    kubectl create namespace my-website
    
    ```
    
    ```jsx
    kubectl run mongo --image=mongo --namespace=my-website
    
    ```

## 기출문제 (66번)

Create a pod that echo "hello world" and then exists.

Have the pod deleted automatically when it's completed

- 답안
    
    ```jsx
    # rm옵션은 pod를 생성하고 실행한 후 해당 pod를 삭제하는 옵션
    # it옵션은 쉘로 접속하여 컨테이너 상태를 파악하는 옵션
    kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
    
    ```

## 기출문제 (68번)

Schedule a Pod as follows:

- Name: kucc1
- App Containers: 2
- Container Name/Images: o nginx o consul
- 답안
