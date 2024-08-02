---
title: "[CKA] Persistent Volume Claim"
excerpt: "Persistent Volume Claim"

categories:
  - CKA
tags:
  - [cka, Persistent, volume, Claim]

permalink: /cka/2024-08-01-categories-CKA-20.md/

toc: true
toc_sticky: true

date: 2024-08-01
last_modified_at: 2024-08-01
---

# [20] Persistent Volume Claim을 사용하는 Pod 운영

# ❓Application with Persistent Volume Claim

- Create a new PersistentVolumeClaim 
    - Name: `app-volume`
    - Storage Class: `app-hostpath-sc`
    - Capacity: `10Mi`
- Create a new pod which mounts the PersistentVolumeClaim as a volume 
    - Name: `web-server-pod`
    - Image: `nginx`
    - Mount Path: `/usr/share/nginx/html`
- Configure the new pod to have `ReadWriteMany` access on the volume.
- 작업 클러스터: k8s

### Reference

docs에서 persistent volume 검색 후 → PersistentVolumeClaim부분

[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

docs에서 persistent volume 검색 후 → claims as volumes

[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

### 실습

```docker
[user@console ~]$ kubectl config use-context k8s

[user@console ~]$ vi app-volume-pvc.yaml

#docs에서 Persistent volume 검색 후 PersistentVolumeClaims 부분 긁어옴

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-volume
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Mi
  storageClassName: app-hostpath-sc

:wq

[user@console ~]$ kubectl apply -f app-volume-pvc.yaml

[user@console ~]$ kubectl get pvc 

[user@console ~]$ vi web-server-pod.yaml

#docs에서 persistent volume 검색 -> Claims as volumes 부분 긁어옴

apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: app-volume
  
:wq

[user@console ~]$ kubectl apply -f web-server-pod.yaml

[user@console ~]$ kubectl get pods

[user@console ~]$ kubectl describe pod web-server-pod

```

## 기출문제 (61번)

kubectl config use-context k8s Create a new PersistentVolumeClaim:

Name: pv-volume

Class: app-hostpath-sc

Capacity: 10Mi Create a new Pod which mounts the PersistentVolumeClaim as a volume: Name: web-server-pod Image: nginx Mount path: /usr/share/nginx/html Configure the new Pod to have ReadWriteMany access on the volume. Finally, using kubectl edit or kubectl patch expand the PersistentVolumeClaim to a capacity of 70Mi and record that change.

- 답안
    
    ```jsx
    
    
    ```

## 기출문제 (13번)

List all persistent volumes sorted by capacity, saving the full kubectl output to /opt/KUCC00102/volume\_list.

Use kubectl 's own functionality for sorting the output, and do not manipulate it any further

- 답안
    
    ```jsx
    kubectl get pv --sort-by=.spec.capacity.storage > /opt/KUCC00102/volume_list
    
    ```

## 기출문제 (16번)

Create a persistent volume with name app-data, of capacity 2Gi and access mode ReadWriteMany.

The type of volume is hostPath and its location is /srv/app-data.

- 답안
    
    ```jsx
    vi pv.yaml
    
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: app-data
    spec:
      capacity:
        storage: 2Gi
      volumeMode: Filesystem
      accessModes:
        - ReadWriteMany
      persistentVolumeReclaimPolicy: Recycle
      storageClassName: slow
      hostPath:
        path: /srv/app-data
    
    :wq
    
    ```
    
    ```jsx
    kubectl apply -f pv.yaml
    
    ```

## 기출문제 (55번)

Task Create a persistent volume with name app-data , of capacity 1Gi and access mode ReadOnlyMany. The type of volume is hostPath and its location is /srv/app-data .

- 답안
    
    ```jsx
    vi pv.yaml
    
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: app-data
    spec:
      capacity:
        storage: 1Gi
      volumeMode: Filesystem
      accessModes:
        - ReadOnlyMany
      persistentVolumeReclaimPolicy: Recycle
      storageClassName: slow
      hostPath:
        path: /srv/app-data
    
    :wq
    
    ```
    
    ```jsx
    kubectl apply -f pv.yaml
    
    ```
