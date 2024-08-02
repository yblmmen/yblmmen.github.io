---
title: "[CKA] Node 정보 수집"
excerpt: "Node 정보 수집"

categories:
  - CKA
tags:
  - [cka, Node, pod]

permalink: /cka/2024-07-30-categories-CKA-10.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---

# [10] Node 정보 수집

## ❓Check Ready Nodes

- Check to see how many nodes are ready (not including nodes tainted NoSchedule) and write the number to /var/CKA2022/RN0001 
    - not including nodes tainted NoSchedule —&gt; tainted에 NoSchedule 적혀있는건 제외

### 실습

```docker
# ready 단어와 일치하는 것만 결과 출력
[user@console ~]$ kubectl get nodes | grep -i -w ready
hk8s-m    Ready
hk8s-w1   Ready

[user@console ~]$ kubectl describe node hk8s-m | grep -i NoSchedule

[user@console ~]$ kubectl describe node hk8s-w1 | grep -i NoSchedule

```

[![image.png](http://138.2.116.150/uploads/images/gallery/2023-05/scaled-1680-/SFLimage.png)](http://138.2.116.150/uploads/images/gallery/2023-05/SFLimage.png)

hk8s-m 노드에는 NoSchedule 정보가 포함되어 있음

```docker
# 결과값 파일에 저장
[user@console ~]$ echo "1" > /var/CKA2022/RN0001

[user@console ~]$ cat /var/CKA2022/RN0001
1

```

## ❓Count the number of nodes that are ready to run normal workloads

- Determine how many nodes in the cluster are ready to run normal workloads (i.e workloads that do not have any special tolerations)
- Output this number to the file /var/CKA2022/NODE-Count

### 실습

```docker
# ready 상태인 노드 출력
[user@console ~]$ kubectl get nodes | grep -i -w ready

[user@console ~]$ kubectl get nodes | grep -i -w ready | wc -l
2

[user@console ~]$ kubectl get nodes | grep -i -w ready | wc -l > /var/CKA2022/NODE-Count

[user@console ~]$ cat /var/CKA2022/NODE-Count
2
```
