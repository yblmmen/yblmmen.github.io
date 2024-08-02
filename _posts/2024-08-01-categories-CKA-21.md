---
title: "[CKA] Check Resource Information"
excerpt: "Check Resource Information"

categories:
  - CKA
tags:
  - [cka, check, resource, information]

permalink: /cka/2024-08-01-categories-CKA-21.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---


# [21] Check Resource Information

## ❓Check Resource Information

- List all PVs sorted by name saving the full `kubectl` output to `/var/CKA2022/my_volumes`
- Use `kubectl` ’s own functionally for sorting the output, and do not manipulate it any further.
- 작업 클러스터: k8s

### 실습

```docker
[user@console ~]$ kubectl config use-contest k8s

[user@console ~]$ kubectl get svc 

```

[![image.png](http://138.2.116.150/uploads/images/gallery/2023-05/scaled-1680-/jyuimage.png)](http://138.2.116.150/uploads/images/gallery/2023-05/jyuimage.png)

```docker
# 이건 yaml 파일 형태로 출력하는 것
[user@console ~]$ kubectl get svc kubernetes -o yaml

# 이건 json 파일 형태로 출력하는 것
[user@console ~]$ kubectl get svc kubernetes -o json

json 형태로 된 포맷을 보면 .metadata.name 이런식으로 보이는데 이 뜻이 metadata 밑에 있는 name 이런 뜻

# 문제에서 name으로 sort하라고 했으니깐
[user@console ~]$ kubectl get pv --sort-by=.metadata.name

[user@console ~]$ kubectl get pv --sort-by=.metadata.name > /var/CKA2022/my_volumes

# 확인
[user@console ~]$ cat /var/CKA2022/my_volumes

```
