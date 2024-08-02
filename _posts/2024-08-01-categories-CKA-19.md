---
title: "[CKA] Persistent Volume 생성"
excerpt: "Persistent Volume 생성"

categories:
  - CKA
tags:
  - [cka, Persistent, Volume]

permalink: /cka/2024-08-01-categories-CKA-19.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [19] Persistent Volume 생성

# Persistent Volume이란

클러스터 내의 지속적인 데이터 저장을 위한 추상화된 스토리지를 제공한다. 클러스터 외부 스토리지 리소스를 클러스터 내부에서 사용할 수 있게 하며, 데이터의 지속성과 데이터 볼륨에 대한 독립성을 보장 한다. 그 외 클러스터 관리자에 의해 생성되고 관리되며, Pod 및 다른 K8s 리소스에서 사용할 수 있다. 이를 통해 애플리케이션에서 데이터를 영속적으로 저장하고 공유할 수 있다.

### Reference

docs에서 persistent volume 검색

[퍼시스턴트 볼륨](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/)

## ❓Create Persistent Volume

- Create a Persistent Volume with name `app-config` of capacity `1Gi` and access mode `ReadWriteMany`
- StorageClass: az-c
- The type of Volume is `hostPath` and its location is `/srv/app-config`
- 작업 클러스터: k8s

### 실습

```docker
[user@console ~]$ kubectl config use-context k8s

[user@console ~]$ vi app-config-pv.yaml

#docs에서 persistent volume 검색 --> Persistent Volumes 부분 긁어옴

apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-config
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  storageClassName: az-c
	hostPath:
		path: /srv/app-config

:wq

[user@console ~]$ kubectl apply -f app-config-pv.yaml

[user@console ~]$ kubectl get pv

```
