---
title: "[Kubernetes]-도커&쿠버네티스 2일차"
excerpt: "도커 컨테이너 설치 및 실행 실습"

categories:
  - Kubernetes
tags:
  - [kubernetes, docker]

permalink: /kubernetes/k-2/

toc: true
toc_sticky: true

date: 2024-05-21
last_modified_at: 2024-05-21
---






## 도커&컨테이너(2일차)

**1. centos7 docker 설치**

```
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
systemctl enable --now docker
yum install -y bash-completion git
```

**2. ubuntu20 docker 설치**

```
# Add Docker's official GPG key:
apt-get update
apt-get install ca-certificates curl -y
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
 

# 사용자 계정 docker 권한부여
sudo docker load -i test_commit.tar
sudo usermod -aG docker yeshua0605
sudo chmod 666 /var/run/docker.sock



## 원격 호스트로 이미지 전송
sudo scp -i [private.key] sale.tar yeshua0605@10.178.0.3:/home/yeshua0605
```

**3. Dockerfile 작성**

```
# mkdir test && cd $_
# vi Dockerfile
FROM ubuntu:18.04
MAINTAINER johnlee
LABEL "name"="webserver"
ENV aloha=date
ENV path=/var/www/html
RUN sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list
RUN apt-get update && apt-get install apache2 -y
RUN apt-get install apache2 -y
COPY nihao $path/nihao
COPY hello.html $path
ADD sales.tar $path
WORKDIR $path
RUN echo ohayo > ohayo.html
VOLUME $path
EXPOSE 80
ENTRYPOINT ["apachectl"]
CMD ["-D", "FOREGROUND"]

# docker build -t web-site:sales ./
```

**4. Docker Private registry**

```
# docker run -d -p 5000:5000 --restart=always --name private-docker-registry registry # 저장소 서버
# vi /etc/docker/daemon.json
{ "insecure-registries":["172.16.0.130:5000"] }

# systemctl restart docker

# docker tag web-site:sales 172.16.0.130:5000/web-site:sales
# docker tag web-site:foods 172.16.0.130:5000/web-site:foods
# docker push 172.16.0.130:5000/web-site:sales
# docker push 172.16.0.130:5000/web-site:foods

# docker run -d -p 80:80 --name sales 172.16.0.130:5000/web-site:sales
# docker run -d -p 80:80 --name foods 172.16.0.130:5000/web-site:foods
```

**5. Docker persistent volume**

- Bind Mount

```
# source="$(pwd)/my-web" && target='/usr/share/nginx/html'
# mkdir my-web
# echo "Hello World" > my-web/index.html
# docker run -d -p 8012:80 --name my-web --mount type=bind,source=$source,target=$target nginx
# curl localhost:8012
# echo "Aloha" > my-web/index.html
# curl localhost:8012
```

- Volume Mount

```
# docker volume create my-vol01
# docker volume list
# docker volume inspect my-vol01
"Mountpoint": "/var/lib/docker/volumes/my-vol01/_data"
# docker run -itd --name vol-test -v my-vol01:/mnt centos:7
# docker run -itd -p 801:80 --name vol-web -v my-vol01:/usr/local/apache2/htdocs:ro httpd:latest
# curl 192.168.0.151:801
<html><body><h1>It works!</h1></body></html>
# docker exec vol-test sh -c "echo "Nihao" > /mnt/index.html"
# curl 192.168.0.151:801
Nihao
```

**6. 워드프레스**

- dbserver

```
# docker volume create back-db
# docker run -d -p 3306:3306 --name dbserver \
-e MYSQL_DATABASE=wordpress \
-e MYSQL_USER=wpuser \
-e MYSQL_PASSWORD=test1234 \
-e MYSQL_ROOT_PASSWORD=test1234 \
--network test_bridge -v back-db:/var/lib/mysql mariadb
```

- webserver

```
# docker volume create back-web
# docker run -d -p 80:80 --name webserver \
-e WORDPRESS_DB_HOST=dbserver \
-e WORDPRESS_DB_NAME=wordpress \
-e WORDPRESS_DB_USER=wpuser \
-e WORDPRESS_DB_PASSWORD=test1234 \
--network test_bridge -v back-web:/var/www/html wordpress
```

- Docker compose

도커 컴포즈 파일은 다음고 같은 세 개의 최상위 구문(statement)으로 구성된다.

1) version은 이 파일에 사용된 도커 컴포즈 파일 형식의 버전을 가리킨다.                                                              여러 번에 걸쳐 문법과 표현 가능한 요소에 많은 변화가 있었으므로 먼저                                                                        정의가 따르는 형식 버전을 지정할 필요가 있다.
2) services는 애플리케이션을 구성하는 모든 컴포넌트를 열거하는 부분이다.                                                                       도커 컴포즈에서는 실제 컨테이너 대신 서비스 개념을 단위로 삼는다.                                                                  하나의 서비스를 같은 이미지로 여러 컨테이너에서 실행할 수 있기 때문이다.
3) networks는 서비스 컨테이너가 연결될 모든 도커 네트워크를 열거하는 부분이다.

```
# mkdir my_wordpress && cd $_
# vi docker-compose.yml
services:
  dbserver:
    image: mariadb
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment: # -e
      MYSQL_ROOT_PASSWORD: test1234
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: test1234
  wordpress:
    depends_on:
      - dbserver # wordpress 컨테이너 실행에 앞서서 dbserver을 실행
    image: wordpress
    volumes:
      - wordpress_data:/var/www/html
    ports: # -p 80:80
      - "80:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: dbserver:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: test1234
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}

# docker compose up -d
# docker compose ps
# docker compose pause
# docker compose unpause
# docker compose port wordpress 80
# docker compose config
# docker compose stop wordpress
# docker compose rm wordpress
# docker compose down
# docker compose down -v
# docker compose down --rmi all
```

​                                                                                                                                                                                              **7. Docker 스웜**

```
# yum install -y yum-utils
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
# systemctl enable --now docker

# cat <<EOF >> /etc/hosts
10.178.0.4 manager1
10.178.0.5 worker1
10.178.0.6 worker2
EOF

# docker swarm init --advertise-addr 10.178.0.4
# docker swarm join --token SWMTKN-1-0vkdjidcs2fmu7wn8nltkyidkpad4h0nrrknukqu0vymxlsv6z-dblxviuzbpet21peke80foobo 172.25.0.136:2377
# docker node ls
# docker service create --name my_web --replicas 3 --publish published=8080,target=80 nginx
# docker service ls
# docker service ps my_web
# docker service logs my_web
# docker service inspect --pretty my_web
# docker service scale my_web=5
# docker service ps my_web
# docker service update --image halilinux/web-site:v1.0 my_web
# docker service ps my_web
# docker node ls
# docker node update --availability drain worker1
# docker node inspect --pretty worker1
# docker service ps my_web
# docker node update --availability active worker1
# docker node inspect --pretty worker1
# docker node update --availability pause worker2
# docker service scale my_web=5
# docker node inspect --pretty worker2
# docker node update --availability active worker2
# docker service ps my_web
# docker node ls
```

**8. 도커 네트워크 구조**

- docker0 : 브리지 역할을 하여 컨테이너간 통신
  - ip a : docker0 인터페이스 확인
  - veth 5[인터페이스 번호] : container 인터페이스
    - 예) 25: vethda42ae8@if24: (25-도커외부)
    - 예) 24: eth0@if25: (24-컨테이너 외부, if25-컨테이너내부)
- 인터페이스 확인
  - docker inspect fe
  - docker network ls
- 도커환경에서 네트워크 추가 및 변경
  - 사용예시 : docker network create -d(=driver 브릿지옵션) bride –subnet(192.168.123.0/24)
  - dhcp 설정 : –ip-range 192.168.123.128/25 test_bridge

```
docker network create -d bridge --subnet 192.168.123.0/24 --ip-range 192.168.123.128/25 test_bridge
```

- 도커 컨테이너에 신규 네트워크 연결

  ```
  docker network connect test_bridge webserver
  ```

- 도커 컨테이너 네트워크 연결 끊기

  ```
  docker network disconnect test_bridge
  ```

- 도커 컨테이너에 특정 네트워크 연결하여 서비스 올리기

  ```
  docker run -d -p 8888:80 --name webserver1 --network test_bridge test_commit:v1.0
  docker run -d -p 8080:80 --cpus 0.5 --memory 128m -v /html:/usr/share/nginx/html --network test_bridge --name test_br_web test_commit:v2.0
  ```

  ​                                                                                                                                                                                         **-  도커파일 작성 및 이미지 생성**

  - Iac(Infra_) : 서버 템플릿 도구 1) 서버템플릿 도구 : 도커(Dockerfile) 2) 오케스트레이션 도구 : kubernetes(manifest) 1) bash script 도구 : ad-hoc 4) 구성관리도구 : ansible (playbook) 5) 프로비전도구 : 생성, 구성, 실행하는 도구 (terraform)
    - teraaform : HCL 로 스크립트 작성
  - 도커이미지는 도커 파일을 작성
  - 도커파일 구성요소
    - FROM : 도커 베이스 이미지(골든이미지)를 지정하고 가져오는 역할
      - 예) FROM ubuntu:20.04
    - RIUN : 베이스 이미지에 특정 어플리케이션을 설치, 실행을 정의
    - CMD : 컨테이너 실행을 위한 기본 명령어 설정
    - EXPOSE : 컨테이너가 리스닝할 포트 지정
      - 예) -P의 역할과 동일, 8080:80(=EXPOSE에 해당되는 부분)
    - ENV : 환경변수 설정
      - 예) mariadb - username, password, hostname 등의 변수 설정
    - ADD와 COPY
      - 파일과 디렉토리를 이미지로 복사
      - COPY : 로컬 파일 -> 이미지로 복사
      - ADD : 로컬에 tar 파일을 압축을 풀어서 복사
        - tar -xvf 기능탑재
    - ENTRYPOINT
      - 실행파일과 기본 파라미터를 분리하는 기능
    - WORKDIR : 작업폴더이면서 CD의 역할
    - VOLUME :
    - MAINTAINER : 작성자 정보(필수아님)
    - LABEL : 이미지 특징, 용도 설명
    - USER : 서버 계정 지정(기본: root)
    - ONBUILD : 빌드 후 실행명령(가이드 이미지), 필수아님

- 도커파일 사전준비

```
// index 파일 미리 생성
docker exec -it amazing_raman /bin/bash echo “hello” > index.html echo “hello” > index.htm

// 현재 폴더에 복사
cp /home/yeshua0605/web-site-v1.tar ./
```

```
// 도커파일 복사
mkdir test && cd $_ (바로 직전에 만든 폴더로 이동)
vi Dockerfile  
FROM ubuntu:18.04 MAINTAINER yeshua0605 LABEL “name”=”webserver”
```

```
// 이미지 내 환경변수 지정
ENV aloha=date ENV path=/var/www/html
RUN sed -i ‘s/archive.ubuntu.com/ftp.daumkakao.com/g’ /etc/apt/sources.list RUN apt-get update && apt-get install apache2 -y  //아파치 레포지토리를 kakao로 변경해서 빠르게 다운하기 위함

RUN sed -i ‘s/archive.ubuntu.com/ftp.daumkakao.com/g’ /etc/apt/sources.list RUN apt-get update && apt-get install apache2 -y

// $path 는 path값 변수
COPY nihao $path/nihao COPY hello.html $path

// tar 파일을 압축을 풀어서 옮길 때 사용
ADD web-site-v1.tar $path  --> cd /var/www/html 과 동일한 기능

WORKDIR $path RUN echo “<H1>ohayo</H1>” > ohayo.html

// -v 옵션과 동일한 기능
VOLUME $path

// 컨테이너 영역에서 사용되는 포트번호 
EXPOSE 80

// 변경불가한 무조건 실행해야할 내용 
ENTRYPOINT [“apachectl”]

// 옵션과 인자 등의 변수 
CMD [“-D”, “FOREGROUND”]

// 도커 빌드 ‘./’ or ‘.’ -> 도커파일위치 지정 
docker build -t web-site:v1.0 ./ ```

// 도커 정리
docker container prune ```

// 중지되어 있는 컨테이너 삭제 
[root@docker-centos7 test]# docker container prune 
WARNING! This will remove all stopped containers. Are you sure you want to continue? [y/N] y 
Deleted Containers: 84e8de51a3eb061b88c938d784d8f0df6f3ccd10d527e7c1da0146c9bc36ce82
```

\# 모든 컨테이너 삭제 docker rm -f $(docker ps -a -q)

\# 실행된 컨테이너 bash 진입 docker exec -it web-site-v1 bash

\# 실행된 컨테이너 변수 확인 env

\# 실행명령어 확인 $aloha

\# 워크디렉토리 echo $path

  

- 도커허브를 통해 이미지 업로드

```
// 도커허브 로그인
docker login

// 도커 이미지 tag 작업Permalink
docker tag web-site:v1.0 hwlim78/web-site:v1.0

// 도커 이미지 push
docker push hwlim78/web-site:v1.0

// 도커 이미지 다운
docker push hwlim78/web-site:v1.0
```

- 도커 프라이빗 레지스트리 만들기

```
# 저장소 서버
docker run -d -p 5000:5000 –restart=always –name private-docker-registry registry 
–restart=always : 서버가 재부팅 되더라도 자동으로 컨테이너가 실행되도록 함
 


레지스트리 자격증명 vi /etc/docker/daemon.json { “insecure-registries”:[“10.178.0.3:5000”] } systemctl restart docker <– 꼭 해줘야함

이미지 등록상태 확인 curl http://10.178.0.3:5000/v2/_catalog <– image 이름 확인 curl http://10.178.0.3:5000/v2/log/tags/list <– tag 정보 확인 curl http://10.178.0.7:5000/v2/web-site/tags/list {“name”:”web-site”,”tags”:[“v2.0”]}
```

- 사설 레지에 이미지 푸시하기

```
// tag 작업 
docker tag web-site:v2.0 10.178.0.3:5000/web-site:v2.0

// 레지스트리 자격증명 
vi /etc/docker/daemon.json 
{ “insecure-registries”:[“10.178.0.3:5000”] } 
systemctl restart docker <– 꼭 해줘야함

// 이미지 등록상태 확인 
curl http://10.178.0.3:5000/v2/_catalog  # image 이름 확인 curl http://10.178.0.3:5000/v2/log/tags/list  # tag 정보 확인 curl http://10.178.0.7:5000/v2/web-site/tags/list 
{“name”:”web-site”,”tags”:[“v2.0”]}

// harbor : 웹기반 레지스트리
```

- 사설 레지에 이미지 풀하기

```
; 레지서버 설정과 동일하게 클라이언트에서도 적용해야함 
vi /etc/docker/daemon.json { “insecure-registries”:[“10.178.0.3:5000”] }
systemctl restart docker <– 꼭 해줘야함

// 레지에서 다운받아 실행 
docker run -d -P 10.178.0.3:5000/web-site:blue 
docker cp ./index.html 8609b93e07cc:/var/www/html/ 
docker commit -a “ubuntu20” -m “green” 8609b93e07cc 10.178.0.7:5000/web-site:green 
docker push 10.178.0.7:5000/web-site:green

// 미사용 도커 이미지 삭제
docker image prune

// 모든 이미지 삭제
docker image prune -a
```

- 도커 퍼시스턴트 볼륨 구성

​    -  OS 수준의 볼륨을 설정

​    -  단점 : 볼륨 명령어로 관리가 안됨

```
# Bind Mount 
source=”$(pwd)/my-web” && target=’/usr/share/nginx/html’ 
mkdir my-web 
echo “Hello World” > my-web/index.html 
docker run -d -p 8012:80 –name my-web –mount type=bind,source=$source,target=$target nginx 
curl localhost:8012 

echo “Aloha” > my-web/index.html 
curl localhost:8012
```

- 볼륨 마운트
  - 도커가 직접 관리
  - docker volume 을 통해 관리
  - 도커 볼륨 명령어로 관리가 가능
    - docker volume list

```
Volume docker 
volume create my-vol01 
docker volume list 
docker volume inspect my-vol01 
“Mountpoint”: “/var/lib/docker/volumes/my-vol01/_data” 
docker run -itd –name vol-test -v my-vol01:/mnt centos:7 
docker run -itd -p 801:80 –name vol-web -v my-vol01:/usr/local/apache2/htdocs:ro httpd:latest 

curl 192.168.0.151:801
<html><body><h1>It works!</h1></body></html>

docker exec vol-test sh -c "echo "Nihao" > /mnt/index.html"
curl 192.168.0.151:801 
Nihao
```


      
