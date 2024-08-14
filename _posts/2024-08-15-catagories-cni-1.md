---
title: "[쿠버네티스]-CNI calico"
excerpt: "CNI calico"

categories:
  - Kubernetes
tags:
  - [kubernetes, cni, calico]

permalink: /kubernetes/cni/

toc: true
toc_sticky: true

date: 2024-08-15
last_modified_at: 2024-08-15
---

# Kubernetes – Calico getting start

## Install

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

```
kubectl get pod -n kube-system
NAME                                      READY   STATUS    RESTARTS      AGE
calico-kube-controllers-7bdbfc669-mfdbp   1/1     Running   0             55m
calico-node-4rml2                         1/1     Running   0             51m
calico-node-kv8z8                         1/1     Running   0             50m
calico-node-lg7ch                         1/1     Running   0             51m
calico-node-sfr5f                         1/1     Running   0             50m
calico-node-vmd5s                         1/1     Running   0             51m
```

## calicoctl install

```
root@master1:/home/yeshua/cka/study/calico# curl -L https://github.com/projectcalico/calico/releases/download/v3.28.1/calicoctl-linux-amd64 -o calicoctl
chmod 700 calicoctl
sudo mv calicoctl /usr/local/bin

// calico node 상태
root@master1:/home/yeshua/cka/study/calico# sudo calicoctl node status
Calico process is running.

IPv4 BGP status
+----------------+-------------------+-------+------------+-------------+
|  PEER ADDRESS  |     PEER TYPE     | STATE |   SINCE    |    INFO     |
+----------------+-------------------+-------+------------+-------------+
| 192.168.100.71 | node-to-node mesh | up    | 2024-08-11 | Established |
| 192.168.100.72 | node-to-node mesh | up    | 2024-08-11 | Established |
+----------------+-------------------+-------+------------+-------------+

IPv6 BGP status
No IPv6 peers found.
```

## calicoctl version

```
root@master1:/home/yeshua/cka/study/calico# calicoctl version
Client Version:    v3.28.1
Git commit:        601856343
Cluster Version:   v3.29.0-0.dev-257-g2f3485fe0204
Cluster Type:      k8s,bgp,kubeadm,kdd
```










