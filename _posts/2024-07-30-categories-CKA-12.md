---
title: "[CKA] Pod Log 추출"
excerpt: "Pod Log 추출"

categories:
  - CKA
tags:
  - [cka, sidercar, pod]

permalink: /cka/2024-07-30-categories-CKA-12.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---
# [12] Pod Log 추출

## ❓Record the extracted log lines

- Monitor the logs of pod custom-app and: Extract log lines corresponding to error file not found. write them to /var/CKA2022/podlog 
    - 작업 클러스터: hk8s

### Reference

docs에서 reference 검색 &gt; logs 검색

[Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs)

### 실습

```docker
[user@console ~]$ kubectl config use-context hk8s

[user@console ~]$ kubectl get pods custom-app

[user@console ~]$ kubectl logs custom-app | grep 'file not found'

[user@console ~]$ kubectl logs custom-app | grep 'file not found' > /var/CKA2022/podlog

# 잘 들어갔는지 확인
[user@console ~]$ cat /var/CKA2022/podlog

```

## 기출문제 (1번)

List pod logs named "frontend" and search for the pattern "started" and write it to a file "/opt/error-logs"

- 답안
    
    ```jsx
    kubectl get pod frontend
    
    ```
    
    ```jsx
    kubectl logs frontend | grep -i "started" > /opt/error-logs
    
    ```
    
    ```jsx
    # 확인
    cat /opt/error-logs
    
    ```

## 기출문제 (7번)

Monitor the logs of pod foo and: Extract log lines corresponding to error unable-to-access-website Write them to /opt/KULM00201/foo

- 답안
    
    ```jsx
    kubectl get pods
    --> foo라는 이름의 pod있는지 확인
    
    ```
    
    ```jsx
    kubectl logs foo | grep unable-to-access-website
    
    ```
    
    ```jsx
    kubectl logs foo | grep unable-to-access-website > /opt/KULM00201/foo
    
    ```

## 기출문제 (17번)

Task Monitor the logs of pod bar and:

- Extract log lines corresponding to error file-not-found
- Write them to /opt/KUTR00101/bar
- 답안  
    ```jsx
    kubectl logs bar | grep file-not-found
    
    ```
    
    ```jsx
    kubectl logs bar | grep file-not-found > /opt/KUTR00101/bar
    
    ```
