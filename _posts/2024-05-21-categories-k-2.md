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

### 도커 설치 및 실행 명령어

```
--- centos7 docker 설치
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
systemctl enable --now docker
yum install -y bash-completion git

--- ubuntu20 docker 설치
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

--- Dockerfile 작성
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

--- Docker 프라이빗 레지스트리
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

--- Docker 퍼시스턴트 볼륨
- Bind Mount
# source="$(pwd)/my-web" && target='/usr/share/nginx/html'
# mkdir my-web
# echo "Hello World" > my-web/index.html
# docker run -d -p 8012:80 --name my-web --mount type=bind,source=$source,target=$target nginx
# curl localhost:8012
# echo "Aloha" > my-web/index.html
# curl localhost:8012

- Volume
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

--- 워드프레스
- dbserver
# docker volume create back-db
# docker run -d -p 3306:3306 --name dbserver \
-e MYSQL_DATABASE=wordpress \
-e MYSQL_USER=wpuser \
-e MYSQL_PASSWORD=test1234 \
-e MYSQL_ROOT_PASSWORD=test1234 \
--network test_bridge -v back-db:/var/lib/mysql mariadb

- webserver
# docker volume create back-web
# docker run -d -p 80:80 --name webserver \
-e WORDPRESS_DB_HOST=dbserver \
-e WORDPRESS_DB_NAME=wordpress \
-e WORDPRESS_DB_USER=wpuser \
-e WORDPRESS_DB_PASSWORD=test1234 \
--network test_bridge -v back-web:/var/www/html wordpress

--- Docker 컴포즈
도커 컴포즈 파일은 다음고 같은 세 개의 최상위 구문(statement)으로 구성된다.
1) version은 이 파일에 사용된 도커 컴포즈 파일 형식의 버전을 가리킨다. 여러 번에 걸쳐 문법과 표현 가능한 요소에 많은 변화가 있었으므로 먼저 정의가 따르는 형식 버전을 지정할 필요가 있다.
2) services는 애플리케이션을 구성하는 모든 컴포넌트를 열거하는 부분이다. 도커 컴포즈에서는 실제 컨테이너 대신 서비스 개념을 단위로 삼는다. 하나의 서비스를 같은 이미지로 여러 컨테이너에서 실행할 수 있기 때문이다.
3) networks는 서비스 컨테이너가 연결될 모든 도커 네트워크를 열거하는 부분이다.

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

--- Docker 모니터링
# docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:v0.49.1

--- Docker 스웜
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

--- Docker 로그
# docker info --format '{{.LoggingDriver}}'
# docker logs [container]
# docker logs --tail 10 [container]
# docker logs -f [container]
# docker logs -f -t [container]
# cat /var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log
```

### ssh kegen 생성방법
  - cmd > ssh-keygen -t rsa
