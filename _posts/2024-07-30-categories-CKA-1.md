---
title: "[CKA]  ETCD Backup & Restore "
excerpt: "ETCD Backup & Restore"

categories:
  - CKA
tags:
  - [cka, etcd, bakcup, restore]

permalink: /cka/2024-07-30-categories-CKA-1.md/

toc: true
toc_sticky: true

date: 2024-07-30
last_modified_at: 2024-07-30
---

# [1] ETCD Backup & Restore

## ❓ETCD Backup &amp; Restore

- 작업 시스템: k8s-master
    
    First, create a snapshot of the existing etcd instance running at `https://127.0.0.1:2379`, saving the snapshot to `/data/etcd-snapshot.db`.
    
    Next, restore an existing, previous snapshot located at `/data/etcd-snapshot-previous.db`.
    
    The following TLS certificates/key are supplied for connecting to the server with etcdctl.
    
    CA certificate: `/etc/kubernetes/pki/etcd/ca.crt`
    
    Client certificate: `/etc/kubernetes/pki/etcd/server.crt`
    
    Client key: `/etc/kubernetes/pki/etcd/server.key`

##### **설명:**

etcd는 key:value 형태로 데이터를 저장하며 etcd도 하나의 pod형태로 구성되어 있다.

etcd에 저장되는 데이터들은 모두 /var/lib/etcd 디렉토리에 저장된다.

이 디렉토리를 하나의 파일로 저장하는 것을 백업한다고 표현하며 쿠버네티스 상에서는 etcd를 스냅샷 떴다고 표현한다.

etcd 백업 후 restore 할 때에는 현재 구동중인 /var/lib/etcd 디렉토리에 overwrite하면 안되고 임의의 다른 디렉토리에 restore해야한다.

임의의 다른 디렉토리에 restore한 후 etcd pod에게 /var/lib/etcd 디렉토리가 아닌 restore한 다른 디렉토리를 바라보게 함으로써 restore한다.

- 과정 
    - etcd 스냅샷 생성
    - 생성한 스냅샷으로 /var/lib/etcd 디렉토리가 아닌 임의의 디렉토리에 restore
    - config파일을 수정해 etcd pod가 restore한 임의의 디렉토리를 바라보게 한다.

### Reference

docs에서 etcd backup 검색

[Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

### 실습

```docker
# 현재 위치중인 cluster 확인
[user@console ~]$ kubectl config current-context
k8s

# 현재 위치중인 k8s 클러스터에 위치한 k8s-master 노드로 이동
[user@console ~]$ ssh k8s-master

# user계정으로는 etcd 백업을 하지 못하므로 root계정으로 전환해서 사용하여야 함

```

```docker
# etcd 명령어를 사용하기 전에 etcd가 설치되어 있는지 확인
[user@k8s-master ~]$ etcdctl version
etcdctl version: 3.5.2
API version: 3.5

# kubernetes docs에 나와있는 etcd backup 예제
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \\
--cacert=<trusted-ca-file> \\
--cert=<cert-file> \\
--key=<key-file> \\
snapshot save <backup-file-location>

# 위의 예제를 사용해 명령어 실행
# 이 명령어를 실행할때는 sudo를 해줘야한다
[user@k8s-master ~]$ sudo ETCDCTL_API=3 etcdctl \\
--endpoints=https://127.0.0.1:2379 \\
--cacert=/etc/kubernetes/pki/etcd/ca.crt \\
--cert=/etc/kubernetes/pki/etcd/server.crt \\
--key=/etc/kubernetes/pki/etcd/server.key \\
snapshot save /data/etcd-snapshot.db

# 스냅샷이 생성됬는지 확인
[user@k8s-master ~]$ sudo ls -l /data/etcd-snapshot.db

# 문제에 나와있는 /data/etcd-snapshot-previous.db 파일이 있는지 확인
[user@k8s-master ~]$ sudo ls -l /data/etcd-snapshot-previous.db

# kubernetes docs에 나와있는 etcd restore 예제
ETCDCTL_API=3 etcdctl \\
--data-dir <data-dir-location> \\
snapshot restore snapshotdb

# 임의의 디렉토리에 이전 etcd 스냅샷을 이용해 복구
[user@k8s-master ~]$ sudo ETCDCTL_API=3 etcdctl \\
--data-dir /var/lib/etcd-new \\
snapshot restore /data/etcd-snapshot-previous.db


```

```docker
[user@k8s-master ~]$ cd /etc/kubernetes/manifest

[user@k8s-master manifest]$ ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

# hostPath 부분을 변경
[user@k8s-master manifest]$ sudo vi etcd.yaml
- hostPath:
		path: /var/lib/etcd-new

:wq

static pod이기 때문에 재시작 없이 수정사항이 반영된다.

#반영됬는지 확인
[user@k8s-master manifest]$ sudo docker ps -a | grep etcd
-> Up상태가 2개인거 확인

```

## 기출문제 (21번)

First, create a snapshot of the existing etcd instance running at [https://127.0.0.1:2379](https://127.0.0.1:2379), saving the snapshot to /srv/data/etcd-snapshot.db.

[![image.png](http://138.2.116.150/uploads/images/gallery/2023-05/scaled-1680-/Ef0image.png)](http://138.2.116.150/uploads/images/gallery/2023-05/Ef0image.png)

Next, restore an existing, previous snapshot located at /var/lib/backup/etcd-snapshot-previo us.db

[![image.png](http://138.2.116.150/uploads/images/gallery/2023-05/scaled-1680-/eTsimage.png)](http://138.2.116.150/uploads/images/gallery/2023-05/eTsimage.png)

- 답안
    
    ```jsx
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \\
    --cacert=/opt/KUIN00601/ca.crt \\
    --cert=/opt/KUIN00601/etcd-client.crt \\
    --key=/opt/KUIN00601/etcd-client.key \\
    snapshot save /srv/data/etcd-snapshot.db
    
    ```
    
    ```jsx
    ETCDCTL_API=3 etcdctl --data-dir /var/lib/etcd-new \\
    snapshot restore /var/lib/backup/etcd-snapshot-previous.db
    
    ```
    
    ```jsx
    cd /etc/kubernetes/manifests
    
    ```
    
    ```jsx
    vi etcd.yaml
    
    hostPath: 부분을 /var/lib/etcd-new로 수정
    
    :wq
    
    ```
    
    ```jsx
    # 확인
    docker ps -a | grep etcd
    
    2개가 Up 상태여야 함
    
    ```

## 💡Tip!!

1. 스냅샷 생성
    
    
    1. 인증서 파일 있으면 해당 명령어 사용해서 생성
2. 스냅샷 복구
    
    
    1. 이전 스냅샷이 있는지 확인 후 복구
    2. 복구할 때 /var/lib/etcd가 아닌 다른 디렉토리에 복구
3. 다른 디렉토리를 바라보게 config파일 수정
    
    
    1. /etc/kubernetes/manifests/etcd.yaml파일에서 hostPath부분 수정
4. 확인
    
    
    1. docker ps -a | grep etcd했을 때 2개가 Up상태이면 끝
    
    검색 : backup → Snapshot using etcdctl options
 



