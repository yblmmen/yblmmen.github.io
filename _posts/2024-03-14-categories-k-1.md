---
title: "[Kubernetes]-도커&쿠버네티스 1일차"
excerpt: "도커 컨테이너 설치 및 실행 실습"

categories:
  - Kubernetes
tags:
  - [kubernetes, docker]

permalink: /kubernetes/k-1/

toc: true
toc_sticky: true

date: 2024-03-14
last_modified_at: 2024-03-16
---

## Docker&kuber 1D

### GCP 설치

 - 강의자료
   - 임시자료실 : http://192.168.2.222/
   - 임시채팅방 : http://192.168.2.222:3000
      - 비밀번호 : Rapa5472!
   
 - 프로젝트 생성
 - VM 인스턴스 생성
    - centos7 or ubuntu
 - docker 설치
 
```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl enable --now docker
sudo yum install -y bash-completion git
sudo usermod -a -G docker johnlee
```
  - 시간설정변경
  ```
  timedatectl set-timezone Asia/Seoul
  ```
  
  - docker 이미지 검색
    - docker search nginx --limit 5
    - docker search ubuntu

  - docker 이미지 다운
    - docker image pull nginx
    - docker pull nginx 
  
  - docker 이미지 list
    - docker image ls
    - docker images
  
  - registrt(dockerhub:생략) > repogitory(directory 개념: nginx) > tag(버전)
    - 예) image 명: docker.io(생략)/nginx/latest   
  
  - docker 이미지 ID 검색
  
  ```  
  docker images -q
  ```
  
  - docker 이미지 상세검색
  ```
  docker inspect nginx:latest
  ```
  - 옵션(-format) : 특정 키워드 검색

  ```
  docker image inspect --format="{{.Os}}" nginx
  docker inspect nginx | grep Os
  ```
  
  - docker container 생성
  - docker create -p(port-foward 기능: publish)
    - docker create - p 80(외부 port):80(container port)
       
  ```    
  docker create -p 80:80 --name(컨테이너 이름설정) webserver nginx:latest
  ```
    
  - docker container 시작
    - docker container start [container name] or [container id]
    
  - docker container 조회
    - docker container ls = docker ps(현재 실행중인 리스트는 안보임)
    - docker container ls -a = docker ps -a (모든 컨테이너 조회)
    - ps = process 의 줄임말
    
  - 서버 TCP 세션 정보: ss -nat
  
  - 서버 인터페이스별 ip 현황
    - ip a
  
  - 서버 외부 ip 확인
    - curl ifconfig.me
    - curl ipconfig.io(에코서비스)
    
  - docker container 중지
    - docker stop [container id or name]
      - Exited (0) : stop 일때 상태정보
      - Exited (?) : 숫자정보별로 상태정보 다름
  
  - docker run 
    - container create + start
    - 컨테이너 생성과 시작을 모두 실행
    - docekr run -d(--detach) : 백그라운드 실행
    - docker run -i(interactive) : 표준입력을 연다. 포워그라운드 실행
    - docker run -t(tty) : 터미널 사용
    - 일반적으로 -i 와 -t 는 함께 사용
    ```
    docker run -it
    docker run -it ---name test_bash centos /bin/bash
    ```
         
    - docker run --name test_cal centos /bin/cal
      - Unable to find image 'centos:latest' locally  
      - 로컬상에 이미지가 없을경우 이미지 다운하여 실행
      - 임시 실행 후 자동으로 중지
      - 
    
    
  - docker container 삭제
    - 컨테이너가 중지상태일 때
      - docker rm
    
    - 컨테이너가 실행상태일 때
      - docker rm -f(=force) 
    
    - 
   
  - docker 백그라운드 실행
    - docker run -d
    
    ```
    docker run -d --name test_ping centos /bin/ping localhost
    확인 : docker logs -t test_ping2
    ```
    
  - docker container 상태 확인
    - docker logs -t : 중요함
    ```
    test : docker run -d -p 80:80 --name test_port_confict nginx
    test 2 : docker run -d -p 88:80 --name test_port_confict nginx          
     - port 충돌이후 또 실행하면 이번에는 container 명이 충돌이 발생함
     - 해결 : docker run -d -p 88:80 --name test_port_confict2 nginx
     - container 명을 변경후 해결
    ```
    
  - docker state (도커 상태 모니터링)
    - docker stats [container 이름]
      - 과부화 기준은 일반적으로 cpu를 메트릭 지표로 잡는다
      - cpu는 %, memory는 총량을 기준으로 표시
  
  - docker 리소스 지정방법
  ```  
  docker container run -d -p 8181:80 --cpu 1 --memory=256m --name test_resource nginx
  ```
     - nginx에서는 이미지 내부에 container 내 port를 80 port로 기본세팅되어 있음
       - 확인 : docker inspect --format="{{.Config.ExposedPorts}}" nginx

     - container에서 cpu는 소수점 지정 가능
        - cpu 0.1 or cpu 0.25
     - docker run -d -P(=expose) : 테스트의 경우 랜덤하게 포트지정하게 해줌
     
   - docker 디렉토리 공유  
     - docker run -v(볼륨공유) /tmp(호스트 경로) : /usr/share/nginx/html

   ```   
   docker run -d -p 8282:80 --cpus 0.5 --memory 128m -v /tmp:/usr/share/nginx/html --name volume-container nginx
   ```     
     - /tmp 폴더가 공유되었기 때문에 /tmp 폴더에 index.html을 생성해줘야 함
        - 해결방법 : echo "hello world" > index.html
     
   - docker 리스트 표시

   ```
   docker ps -a -f name=test_ : test_ 키값(글자)가 들어간 이름은 검색됨
   docker ps -a -f exited=0 : 컨테이너 상태에 따른 검색
   docker ps -a --format "tabe{{.Nmae}}"\
   - 예) docker ps -a --format "table{{.Names}}\t{{.Status}}"
   - docker ps -a --format "table{{.Names}}\t{{.Status}}" > table.txt   
    -> 특정 파일로 저장 및 출력
    -> 확인 : cat table.txt
   ``` 
   
   - 동작중인 컨테이너 실행(docker container exec)
     - attach
       - docker attach test_bash
       - ctrl+c를 눌르고 나오면 컨네이너가 중지
       - ctrl+p, ctrl+q 누르면 컨테이너 중지안됨
       
     - exec (아주 중요한 명령어)
       - run 명령어와 유사
       - 사용예시: docker exec -it test_bash /bin/echo "hello world"
       - 사용예시2 : docker exec -it test_bash /bin/cal
       - 이미 실행중인 컨테이너에서 실행할 때 사용
       - 쿠버네티스에서는 kubectl 명령어로 활용
       - exit 로 빠져나옴 > 컨테이너는 여전히 실행상태
       
   - 도커의 자원사용률
     - docker top test_port : 도커 프로세스 검색
     - docker port test_port : 도커에서 사용하는 port 정보
       - docker ps -f name=test_ 로도 확인가능
   
   - 도커 컨테이너 이름 변경
     - docker rename test_port webserver
     
   - 도커 복사
     - docker container cp(=copy)
       - scp와 유사한 명령어
       - docker cp webserver:/usr/share/nginx/html/index.html /root/index.html
         - container root 폴더 > 호스트 root 폴더로 복사
       - docker cp /root/index.html webserver:/usr/share/nginx/html/index.html
         - 호스트 root 폴더 > container root 폴더로 복사
         - 컨테이너에서 변경된 index.html 확인
         
       - 파일 압축 만들기
         - tar -cvf web-site-v1.tar index.html images
       - 압축 풀기
         - tar -xvf web-site-v1.tar -C /tmp/
         
   - docker diff 명령어
     - docker 변경사항 히스토리
     - docker diff webserver 
      - C : change
      - A : add
      - D : delete
   
   
   - 컨테이너를 이미지로 저장
     - docker commit -a(저작자, 필수아님) -m (메시지) container 이름 repo 이름:버전
       - 예) docker commit -a "hwlim<test@example.com>" -m "TEST" webserver test_commit:v1.0
     - 생성된 이미지는 inspect 를 통해 -a 정보와 -m 정보를 확인가능
       - docker inspect test_commit:v1.0
   
   - 컨테이너 이미지를 파일로 생성
     - docker save : 이미지를 tar 파일로 생성
     - docker save -o test_commit.tar test_commit:v1.0
     
   - 컨테이너 이미지를 도커 이미지로 변경
     - docker load : tar 파일을 도커 이미지로 변경
     - docker load -i test_commit.tar
     
     
     
