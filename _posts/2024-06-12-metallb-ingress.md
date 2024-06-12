---
title: "[Kubernetes]-Nginx ingress controller with MetalLB"
excerpt: "메탈LB를 활용한 쿠버네티스 Ingress Controller 구성"

categories:
  - Kubernetes
tags:
  - [kubernetes, metallb, ingress]

permalink: /kubernetes/metallb-ingress/

toc: true
toc_sticky: true

date: 2024-06-12
last_modified_at: 2024-06-12
---
## kubernetes MetalLB 와 Ingress-Nginx

If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.

kubectl edit configmap -n kube-system kube-proxy

```yml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true #false를 true로 변경
```

또는 자동화

```bash
# 보기
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# 변경
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

이제 설치

Installation By Manifest
공식사이트 [metallb.io](https://metallb.io/installation/)

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.0/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.0/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

이제 설정

vi metallb-configmap.yml

```yml
# 사용 apiversion
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  #해당 IPAddressPool 명과 사용하는 namespace
  name: ip-pool
  namespace: metallb-system
spec:
  addresses:
  # 사용할 ip address pool
  - 192.168.100.62-192.168.100.67
  autoAssign: true

---
apiVersion: metallb.io/v1beta1
# metalib의 l2모드를 사용한다.
kind: L2Advertisement
metadata:
  name: l2-network
  # L2Advertisement이 사용할 namespace명
  namespace: metallb-system
spec:
   # 사용할 ipAddressPools을 추가해주는 데 위에 정의한 ipAddressPools을 사용하도록 한다.
  ipAddressPools:
    - ip-pool
  nodeSelectors:        # first-pool IP를 통해서 들어오는 노드 선택
  - matchLabels:
      kubernetes.io/hostname: worker1.innerinfo.io
  - matchLabels:
      kubernetes.io/hostname: worker2.innerinfo.io

```

```bash
kubectlapply -f metallb-configmap.yml

```

## Ingress-Nginx 

현재 ingress 서비스 확인
> kubectl get svc -n ingress-nginx
```
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                     PORT(S)                      AGE
ingress-nginx-controller             NodePort       10.109.163.218   <none>                          80:30100/TCP,443:30200/TCP   7d17h
ingress-nginx-controller-admission   ClusterIP      10.96.142.146    <none>                          443/TCP                      7d17h
```

## ingress 서비스 추가

멀티 ingress 테스트를 위하여 2개의 yaml을 작성하였음

```yml

# ingress-nginx-controller2.yaml

apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller2
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: ingress-nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30120
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
      nodePort: 30220

  externalIPs:
    - 192.168.100.66


# ingress-nginx-controller3.yaml

apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller3
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: ingress-nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30130
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
      nodePort: 30230

  externalIPs:
    - 192.168.100.67

# 적용
kubectl apply -f ingress-nginx-controller2.yaml
kubectl apply -f ingress-nginx-controller3.yaml

# domain 주소와 external IP가 정확히 일치하는지 반드시 확인하여야 함
```

## ingress rule 설정

deploymet와 서비스는 ssl-test와 ssl80를 사전에 기동시켜 운영중임

```bash
root@k8smaster:/home/master/ingress/service# kubectl get svc,pod -o wide
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE     SELECTOR
service/alertmanager-operated   ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   7d17h   app.kubernetes.io/name=alertmanager
service/kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP                      7d17h   <none>
service/prometheus-operated     ClusterIP   None            <none>        9090/TCP                     7d17h   app.kubernetes.io/name=prometheus
service/ssl-test                ClusterIP   10.101.144.73   <none>        80/TCP                       78m     app=ssl-test
service/ssl80                   ClusterIP   10.98.61.66     <none>        80/TCP                       78m     app=ssl80

NAME                                                         READY   STATUS    RESTARTS   AGE    IP               NODE                     NOMINATED NODE   READINESS GATES
pod/alertmanager-kube-prometheus-stack-1717-alertmanager-0   2/2     Running   0          110m   172.16.127.89    k8sworker1.example.net   <none>           <none>
pod/prometheus-kube-prometheus-stack-1717-prometheus-0       2/2     Running   0          6d3h   172.16.175.238   k8sworker2.example.net   <none>           <none>
pod/ssl-test-5bf4868d6-9bcbm                                 1/1     Running   0          103m   172.16.127.93    k8sworker1.example.net   <none>           <none>
pod/ssl-test-5bf4868d6-gtm8x                                 1/1     Running   0          103m   172.16.175.249   k8sworker2.example.net   <none>           <none>
pod/ssl-test-5bf4868d6-pxzvm                                 1/1     Running   0          103m   172.16.127.91    k8sworker1.example.net   <none>           <none>
pod/ssl80-74976789cd-q4lnc                                   1/1     Running   0          6d3h   172.16.175.255   k8sworker2.example.net   <none>           <none>
pod/ssl80-74976789cd-q4qpq                                   1/1     Running   0          6d3h   172.16.175.245   k8sworker2.example.net   <none>           <none>
pod/ssl80-74976789cd-q6gp7                                   1/1     Running   0          6d3h   172.16.175.207   k8sworker2.example.net   <none>           <none>
```


## Ingress rule 추가

> ssl-test 서비스에 대한 도메인 worker1.innerinfo.io 을 path 라우팅 방식 ingress rule추가

```yml
## 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ssl-test-ingress
spec:
  tls:
  - secretName: ssl-secret
  ingressClassName: nginx
  rules:
  - host: worker1.innerinfo.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ssl-test
            port:
              number: 80
```

> ssl80 서비스에 대한 도메인 worker1.innerinfo.io 을 path 라우팅 방식 ingress rule추가

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ssl80-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: worker2.innerinfo.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ssl80
            port:
              number: 80
```

## 서비스 확인

```bash
root@k8smaster:/home/master/ingress/service# kubectl get svc,ingress
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-operated   ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   7d17h
service/kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP                      7d18h
service/prometheus-operated     ClusterIP   None            <none>        9090/TCP                     7d17h
service/ssl-test                ClusterIP   10.101.144.73   <none>        80/TCP                       87m
service/ssl80                   ClusterIP   10.98.61.66     <none>        80/TCP                       86m

NAME                                         CLASS   HOSTS                  ADDRESS          PORTS     AGE
ingress.networking.k8s.io/ssl-test-ingress   nginx   worker1.innerinfo.io   192.168.100.66   80, 443   112m
ingress.networking.k8s.io/ssl80-ingress      nginx   worker2.innerinfo.io   192.168.100.66   80        7d19h
root@k8smaster:/home/master/ingress/service#
root@k8smaster:/home/master/ingress/service#
root@k8smaster:/home/master/ingress/service# kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                     PORT(S)                      AGE
ingress-nginx-controller             NodePort       10.109.163.218   <none>                          80:30100/TCP,443:30200/TCP   7d17h
ingress-nginx-controller-admission   ClusterIP      10.96.142.146    <none>                          443/TCP                      7d17h
ingress-nginx-controller1            LoadBalancer   10.103.192.81    192.168.100.62,192.168.100.62   80:30110/TCP,443:30210/TCP   91m
ingress-nginx-controller2            LoadBalancer   10.97.169.90     192.168.100.63,192.168.100.66   80:30120/TCP,443:30220/TCP   91m
ingress-nginx-controller3            LoadBalancer   10.99.110.188    192.168.100.64,192.168.100.67   80:30130/TCP,443:30230/TCP   63m
```
