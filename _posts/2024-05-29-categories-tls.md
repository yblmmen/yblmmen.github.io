---
title: "[Project]-PAS-K5600 SSL/TLS POC"
excerpt: "SSL/TLS 가속 "

categories:
  - project
tags:
  - [ssl, tls, PAS-K]

permalink: /project/tls/

toc: true
toc_sticky: true

date: 2024-05-29
last_modified_at: 2024-05-29
---


## [파이오링크] PAS-K5600 SSL가속 POC

### 1. POC 일정
>
- 작업절차 : 1차 - 장비설치, 2차 - POC 진행
- 장비모델 : PAS-K5600
- 구성방안 : One-arm 이중화, DNAT 
- 사전준비 : 작업계획서, POC 시나리오(Offload 성능항목 포함), 체크리스트
 
### 2. 성능 비교
 - 시트릭스 MPS 5905
![thumbnail_image003](https://github.com/yblmmen/gatsby.github.io/assets/161982180/a22a549e-6c62-48da-9b6b-0ff756e14ff5)

 - PAS-K5600
![222](https://github.com/yblmmen/gatsby.github.io/assets/161982180/8fef2f8f-1b11-400d-9f39-892b6d907edc)

### 3. POC 구성도
![thumbnail_image002](https://github.com/yblmmen/gatsby.github.io/assets/161982180/3954ad8d-9905-47c1-a793-861b6d4e3324)

### 4. 네트워크 설정
 
#### 1) LB-DC1-BIZ

```
# Hostname 설정
hostname LB-DC1-BIZ

# VLAN 생성
vlan wan vid x
vlan hot vid x

# VLAN 포트 할당
vlan wan port x untagged
vlan hot port x untagged

# Interface ip 할당
Interface wan ip x.x.x.x/x
Interface hot ip 1.1.1.1/29

# Routing 설정
route default-gateway x.x.x.x
Port-boundary 설정	port-boundary 3
port x-x
apply

# Failover 설정	failover
vrrp x
interface wan vip x.x.x.x
interface hot vip 1.1.1.3
track single-port 5 port x
apply
exit

# management 활성화	management-access
ssh status enable
https status enable
apply

# Key 삽입
부하분산 -> SSL/TLS 가속 -> 키 관리 -> key import

# 인증서 삽입
부하분산 -> SSL/TLS 가속 -> 인증서 -> 인증서 import



# 프로필 생성	 
- 부하분산 -> SSL/TLS 가속 -> 프로필 -> 프로필 생성

# Layer7 pattern 생성
layer7 pattern 1
source-ip x.x.x.0/24
type source-ip
apply

# Real 서버 생성	real x
name [description]
rip x.x.x.x
apply
exit

# health-check 생성	health-check x
type tcp
port x
half-open enable
apply
exit

# advl7slb 정책 생성	advl7slb [name]
vip x.x.x.x protocol https vport x
ssl x
health-check x
backend-timeout 1
response-timeout 600
apply

group ext
lb-method type x
persist type x
real x,x
apply

group int
lb-method type x
persist type x
real x,x
apply

rule 1
group int
action group
pattern 1
snat-addr-type vip-addr
request-scheme http
backend-timeout 1
apply

rule 2
group ext
action group
request-scheme http
backend-timeout 1
apply
apply
exit

```


#### 2) LB-DC2-BIZ**

```
# 네트워크 설정
Hostname 설정	hostname LB-DC2-BIZ
VLAN 생성	vlan wan vid x
vlan hot vid x

# VLAN 포트 할당
vlan wan port x untagged
vlan hot port x untagged

# Interface ip 할당
Interface wan ip x.x.x.x/x
Interface hot ip 1.1.1.2/29

# Routing 설정
route default-gateway x.x.x.x

# Port-boundary 설정
port-boundary 3
port x-x
apply

# Failover 설정	failover
vrrp x
interface wan vip x.x.x.x
interface hot vip 1.1.1.3
track single-port 5 port x
apply
exit

# management 활성화
management-access
ssh status enable
https status enable
apply

# 인증서 삽입	Key 삽입	 
부하분산 -> SSL/TLS 가속 -> 키 관리 -> key import

# 인증서 삽입	 
부하분산 -> SSL/TLS 가속 -> 인증서 -> 인증서 import

# 프로필 생성	 
부하분산 -> SSL/TLS 가속 -> 프로필 -> 프로필 생성

# 부하분산 정책 생성
Layer7 pattern 생성	layer7 pattern 1
source-ip x.x.x.0/24
type source-ip
apply

# Real 서버 생성	real x
name [description]
rip x.x.x.x
apply
exit

# health-check 생성	health-check x
type tcp
port x
half-open enable
apply
exit

# advl7slb 정책 생성	advl7slb [name]
vip x.x.x.x protocol https vport x
ssl x
health-check x
backend-timeout 1
response-timeout 600
apply

group ext
lb-method type x
persist type x
real x,x
apply

group int
lb-method type x
persist type x
real x,x
apply

rule 1
group int
action group
pattern 1
snat-addr-type vip-addr
request-scheme http
backend-timeout 1
apply

rule 2
group ext
action group
request-scheme http
backend-timeout 1
apply
apply
exit

```

### 5. 성능측정

#### wrk 설치

```
# RHEL/CentOS
sudo yum install make gcc git openssl-devel  
sudo yum -y install openssl-devel git

# Ubuntu
sudo apt install make gcc unzip git libssl-dev

# wrk git 저장소를 복제
git clone https://github.com/wg/wrk.git 
cd wrk

# 소스를 컴파일
make WITH_OPENSSL=/usr/

#### wrk 사용

>
- -c, --connections <N>  Connections to keep open
- -d, --duration    <T>  Duration of test
- -t, --threads     <N>  Number of threads to use


```
// 12개 스레드를 400개 연결 후 30초 동안 요청을 전송 
$ wrk -t12 -c400 -d30s http://127.0.0.1:8080/index.php

Running 30s test @ http://127.0.0.1:8080/index.php
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    44.36ms   82.08ms   1.70s    98.07%
    Req/Sec   176.84    119.42     1.04k    69.73%
  59915 requests in 30.11s, 3.60GB read
  Socket errors: connect 0, read 59915, write 0, timeout 0
Requests/sec:   1990.17
Transfer/sec:    122.33MB
```








 
