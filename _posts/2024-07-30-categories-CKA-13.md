---
title: "[CKA] CPU 사용량이 높은 Pod 검색"
excerpt: "CPU 사용량이 높은 Pod 검색"

categories:
  - CKA
tags:
  - [cka, sidercar, pod]

permalink: /cka/2024-07-30-categories-CKA-13.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---

# [13] CPU 사용량이 높은 Pod 검색

## ❓Find a Pod that consumes a lot of CPU

- From the pod label name=overloaded-cpu, find pods running high CPU workloads and write the name of the pod consuming most CPU to the file /var/CKA2022/cpu\_load\_pod.txt 
    - 작업 클러스터: hk8s

### Reference

docs에서 reference 검색 → top 검색

[Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#top)

### 실습

```docker
[user@console ~]$ kubectl config use-context hk8s

[user@console ~]$ kubectl top pods -l name=overloaded-cpu

# cpu사용량이 높은 순대로 정렬
[user@console ~]$ kubectl top pods -l name=overloaded-cpu --sort-by=cpu

```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/825153b6-dea6-423e-8589-bc67c988086d/Untitled.png)

```docker
[user@console ~]$ echo "campus-01" > /var/CKA2022/cpu_load_pod.txt

[user@console ~]$ cat /var/CKA2022/cpu_load_pod.txt

```

## 기출문제 (20번)

Task From the pod label name=cpu-utilizer, find pods running high CPU workloads and write the name of the pod consuming most CPU to the file /opt/KUTR00401/KUTR00401.txt (which already exists).

- 답안  
    ```jsx
    kubectl top pod -l name=cpu-utilizer --sort-by=cpu
    --> CPU 사용량 제일 높은 pod의 이름 확인
    
    ```
    
    ```jsx
    echo '{pod이름}' > /opt/KUTR00401/KUTR00401.txt
    
    ```
