---
title: "[CKA] Node 관리"
excerpt: "Node 관리"

categories:
  - CKA
tags:
  - [cka, Node, pod]

permalink: /cka/2024-07-30-categories-CKA-9.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---

# [9] Node 관리

# Node 관련된 명령어 정리

```docker
# 현재 시스템의 노드 정보 확인하기
kubectl get nodes 

# 더 자세한 정보 확인
kubectl get nodes -o wide

# 해당 node에 대한 자세한 정보 확인
kubectl describe node {node 이름}

# pod에 대한 자세한 정보 확인
kubectl get pods -o wide

# 특정 node에는 배치되지 않도록 하기 (스케줄링 중지)
kubectl cordon k8s-worker1

# 배치되지 않도록 해줬던 설정을 해제하기 (스케줄링 시작)
kubectl uncordon k8s-worker1

# 해당 node에 동작되고 있는 모든 pod들이 삭제됨
kubectl drain k8s-worker2
daemonset으로 실행되고 있는 pod들은 삭제 안됨 -> --ignore-daemonsets 옵션을 넣어주면 해당 에러를 무시하고 모든 pod들을 삭제한다
해당 옵션 사용하면 스케줄링도 자동으로 중지된다.

# k8s-worker2를 다시 스케줄링 가능하게 하기
kubectl uncordon k8s-worker2
이렇게 해도 원래 worker1에서 동작중이던 pod들이 옮겨지지는 않는다.

이 이후에 scale out 명령을 수행하면 k8s-worker2에도 스케줄링 된다.

```

## ❓Management Node

- Set the node named k8s-worker1 as unavailable and reschedule all the pods running on it.
- 작업 클러스터: k8s

### Reference

docs에서 reference 검색 후 cordon 찾기

[Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#cordon)

### 실습

```docker
# 클러스터 전환
[user@console ~]$ kubectl config use-context k8s

# daemonset에 의해 구동중인 pod 정보들도 확인
# k8s-worker1에서 구동중인 모든 pod정보 확인
[user@console ~]$ kubectl get pods -o wide -A | grep worker1

# k8s-worker1에 있는 모든 pod들 삭제
# 해당 node에서 사용하고 있는 스토리지도 삭제 (--delete-emptydir-data)
[user@console ~]$ kubectl drain k8s-worker1 --ignore-daemonsets --force --delete-emptydir-data

이렇게 drain하면 이제 더이상 k8s-worker1에는 스케줄링 되지 않는다.
그리고 k8s-worker1에 있던 pod들은 k8s-worker2 node에 재배치된다. (reschedule)

# node 정보 확인
[user@console ~]$ kubectl get pods -o wide

# k8s-worker1에서는 더이상 스케줄링 안받는다는 상태 확인
[user@console ~]$ kubectl get nodes
```
