---
title: "[CKA] Rolling Update & Rollback"
excerpt: "Rolling Update & Rollback"

categories:
  - CKA
tags:
  - [cka, Rolling, Rollback]

permalink: /cka/2024-07-30-categories-CKA-7.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---
# [7] Rolling Update & Rollback

## ❓Rolling Update

- Create a deployment as follows: 
    - 작업 클러스터: k8s
    - TASK: 
        - name: nginx-app
        - Using container nginx with version 1.11.10-alpine
        - The deployment should contain 3 replicas
- Next, deploy the application with new version 1.11.13-alpine, by performing a rolling update
- Finally, rollback that update to the previous version 1.11.10-alpine

### Reference

docs에서 Deployment검색 Updating a Deployment 부분 확인

[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

### 실습

```docker
# 클러스터 전환
[user@console ~]$ kubectl config use-context k8s

# 조건에 맞춰 deployment 생성
# dry-run 옵션 사용
[user@console ~]$ kubectl create deployment nginx-app --image=nginx:1.11.10-alpine --replicas=3

# 만들어진 deployment 확인
[user@console ~]$ kubectl get deployment nginx-app
NAME          READY
nginx-app     3/3

# nginx-app로 시작하는 pod확인
[user@console ~]$ kubectl get pods |grep nginx-app

# POD안에 있는 container 버전 확인
[user@console ~]$ kubectl describe pod {pod 이름}
container 이미지 버전 nginx:1.11.10-alpine 인거 확인

# docs 참고해서 update
# 여기서 nginx는 container이름
# --record 옵션을 쓰면 히스토리 파악 가능
[user@console ~]$ kubectl set image deployment nginx-app nginx=nginx:1.11.13-alpine --record

# update가 되고있는지 과정 확인
[user@console ~]$ kubectl rollout status deployment/nginx-app

# update가 됬는지 확인
[user@console ~]$ kubectl get pods |grep nginx-app

# 위의 결과에서 나온 하나의 pod 확인
# 버전 업데이트 된 것 확인하기
[user@console ~]$ kubectl describe pod {pod이름}

# rollback 하기 
# 바로 전 단계로 롤백된다
[user@console ~]$ kubectl rollout undo deployment/nginx-app

# rollback 된 것 확인
[user@console ~]$ kubectl get pods | grep nginx-app

[user@console ~]$ kubectl describe pod {pod이름}

```

### Update가 되는 과정

update를 실행하면 기존에 있는 Replicaset에서 새로운 Replicaset이 생성되고 그 새로 생성된 곳에 업데이트 된 버전의 pod가 생성된다.

업데이트 된 버전의 pod가 하나씩 생겨날때마다 기존 버전의 pod가 하나씩 삭제된다.

업데이트가 모두 완료된 후에 get pods명령어를 확인해보면 다른 해쉬값을 가지는 replicaset이 생성된 것을 확인할 수 있다.
