---
layout: post
title: Docker Container 다루기
tags: [Docker, Dev, Linux]
categories:
    - Docker
permalink: docker-container.html
createat: 2020-05-16
thumbnail: /assets/images/thumbnails/docker-logo-1200x628.png
---

## 첫 번째 컨테이너 띄우기

첫번째 컨테이너를 띄워보겠습니다.

```bash
docker run --rm ubuntu:18.04
```

위와 같이 입력하면 다음과 같은 메시지를 볼 수 있습니다.

```
Unable to find image 'ubuntu:18.04' locally
16.04: Pulling from library/ubuntu
e92ed755c008: Pull complete
... (중략)
Status: Downloaded newer image for ubuntu:18.04
```

그리고 실행 후 컨테이너가 종료됩니다.  
이 것은 도커 컨테이너를 생성할 때 필요한 이미지가 없기 때문에 docker hub로 부터 이미지를 pull 받은 후 컨테이너를 생성하고,  
컨테이너에서 아무런 명령을 하지 않았기 때문에  
작업이 완료된 컨테이너 프로세스가 자동으로 종료된 것입니다.

컨테이너가 실행된 상태에서 다른 명령을 수행하기 위해서 아래와 같이 커맨드를 입력합니다.

```bash
docker run --rm -it ubuntu:18.04 /bin/bash
```

이렇게 입력을 하면 커맨드 라인 앞쪽이 (root@6f0c1b86bca6) 이런 형식으로 변한 것을 볼 수 있는데,  
도커 컨테이너 안에서 커맨드 명령을 입력할 수 있는 상태가 된것을 의미합니다.  
--it 옵션은 이렇게 컨테이너 안에서 커맨드 라인을 이용해 입출력을 할 수 있도록 해줍니다.

자주 쓰는 옵션은 다음과 같습니다.

-   -it : 터미널로 입력을 하겠다.
-   --rm : 프로세스를 종료하자 마자 컨테이너 제거 (연습중에는 컨테이너가 겹쳐 실행이 안되는 것을 방지하기 위해 항상 붙여줍니다.)
-   -d : background
-   -p : port
-   -e : 환경변수 <key=value>

## Docker 와 친해지기

docker command 와 친해지기 위해서 간단히 웹서버, mysql, 워드프레스 컨테이너를 다뤄보겠습니다.

### nginx 컨테이너 실행시키기

아래와 같이 커맨드를 입력해서 nginx서버를 띄울 수 있습니다.

```bash
docker run --rm -d \
-p 8080:80 \
nginx:latest
```

-d 옵션으로 백그라운드에서 실행되도록 했고,  
로컬의 8080 을 호스트의 80 포트와 연결시켰습니다.  
\ 문자는 bash에서 한줄을 띄어 쓸 수 있도록 연결시켜 주어 여러줄 입력할 수 있게 합니다.  
브라우저에서 localhost:8080 으로 접속하면 컨테이너의 80포트와 연결된 웹서버에 접근 할 수 있습니다.

### mysql 컨테이너 실행시키기

아래와 같이 명령어를 입력하면 mysql 컨테이너를 실행시킬 수 있습니다.  
눈치 채셨는지 모르겠지만 run 명령어는 image를 받아오는 명령과 start 명령이 혼합된 명령어 입니다.  
그래서 이미지가 없으면 docker hub 로 부터 이미지를 pull 받은 후 실행이 됩니다.

```bash

docker run -d \
-p 3306:3306 \
-e MYSQL_ALLOW_EMPTY_PASSWORD=true \
--name mysql \
mysql:5.7
```

mysql 이미지를 백그라운드에서 실행시켰습니다.  
-e 옵션을 통해서 환경변수 MYSQL_ALLOW_EMPTY_PASSWORD 에 값을 true로 주었습니다.  
--name 옵션을 통해 도커 이미지에 이름을 설정하여 다음부터 이 이름으로 접근할 수 있습니다.  
mysql:5.7 태그가 붙은 이미지를 docker hub로 부터 다운받아 이용합니다.

### wordpress 컨테이너를 실행시키기

```bash
docker exec -it mysql mysql
create database wp CHARACTER SET utf8;
grant all privileges on wp.* to wp@'%' identified by 'wp';
flush privileges;
quit

docker run -d \
-p 80:80 \
--name wp \
--link mysql:mysql \
-e WORDPRESS_DB_USER=wp \
-e WORDPRESS_DB_PASSWORD=wp \
-e WORDPRESS_DB_NAME=wp wordpress
```

첫 번째 줄에서는 위에서 지정한 name(mysql)으로 컨테이너에 접근합니다.  
exec 는 컨테이너 안에서 명령을 실행시킨다는 것인데 여기서는 마지막 mysql(mysql 클라이언트 실행) 명령어를 실행했습니다.
이 후 wp 계정을 만들어 주 다음 wordpress 컨테이너도 run 해줍니다.  
link 를 이용해 네트워크를 연결시켰습니다.  
네트워크를 구성하는 방법은 여러가지 있지만 지금은 이렇게 연결하겠습니다.  
8080 으로 접속하면 간단하게 워드프레스가 구동되는 것을 확인 할 수 있습니다.

## 자주 쓰는 커맨드

### 프로세스 제어

```bash
docker ps
docker ps -a
docker stop <cotainer ID>
docker rm <contariner ID>
```

docker ps 는 현제 컨테이너 리스트를 보여줍니다.
컨테이너 이름과 포트 따로 이름을 생성하지 않으면 자동으로 만들어줍니다.  
-a 정지된 컨테이너를 볼 수 있습니다.
docker stop 으로 컨테이너를 종료시킬 수 있습니다.  
종료시킬때는 아이디 전체를 칠 필요없이 앞에 구분할 수 있을 정도로 몇글자만 입력해주면 됩니다.  
rm 명령어는 중지된 컨테이너를 삭제할 수 있습니다.

```bash
docker logs <containerID>
```

실제 컨테이너에서 생성된 로그를 출력할 수 있습니다.  
-f : 새로 생성되는 로그를 실시간으로 확인할 수 있습니다.

### 이미지 목록 확인

이미지 목록을 보려면 아래와 같이 입력합니다.

```bash
docker images
```

### 이미지 삭제

쓰지 않는 이미지는 주기적으로 삭제해야 용량을 효율적으로 관리할 수 있습니다.

```bash
docker rmi <image ID>
```

### 이미지 다운로드 받기

pull 명령어를 통해 docker hub에서 이미지를 다운로드 받을 수 있습니다.

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

## 네트워크

### 네트워크 만들기

도커컨테이너 끼리 통신을 할 수 있는 가상의 네트워크를 만들 수 있습니다.
네트워크에 방식에는 여러가지가 있지만 자세한 네트워크 관련 설명은 다음번 포스트에서 다루겠습니다.

```bash
docker network create [OPTIONS] NETWORK
```

```bash
docker network create app-network

//네크워크를 이용해서 컨테이너를 새로 생성하거나
docker run -d --name mysql \
-e MYSQL_ALLOW_EMPTY_PASSWORD=true \
--network app-network \
mysql

//혹은 기존에 실행중인 컨테이너에 네트워크에 추가
docker network connect app-network mysql

```

app-network 라는 네트워크를 만들고 mysql 컨테이너를 app-network를 이용하는 컨테이너로 실행시켰습니다.

### docker compose

도커 compose 를 이용하면 커맨드 입력으로 컨테이너를 띄우는 것이 아니고  
.yml 에 미리 설정된 옵션을 가지고 실행시킬 수 있습니다.

```bash
docker-compose up -d
docker-compose down
```

docker-compose 의 기본 실행 파일은 실행하는 디렉토리의 docker-compose.yml 파일입니다.
기본 파일을 이용하지 않고 지정하기 위해서는 아래와 같이 -f 옵션을 사용합니다.

```bash
docker-compose up -d -f my-docker-compose.yml
```

### docker-compose 예제

docker-compose.yml 파일은 아래와 같은 형식으로 작성합니다.  
예제에는 mysql 과 워드프레스 컨테이너를 각각 띄워주었습니다.

```
version: '2'
services:
   db:
    image: mysql:5.7
    volumes:
        - ./mysql:/var/lib/mysql
        restart: always
    environment:
        MYSQL_ROOT_PASSWORD: wordpress
        MYSQL_DATABASE: wordpress
        MYSQL_USER: wordpress
        MYSQL_PASSWORD: wordpress

    wordpress:
        image: wordpress:latest
        volumes:
            - ./wp:/var/www/html
        ports:
            - "8000:80" restart: always
        environment:
            WORDPRESS_DB_HOST: db:3306
            WORDPRESS_DB_PASSWORD: wordpress

```

volumes 를 보면 볼륨마운트를 해줄 수 있습니다.  
볼륨이란 호스트에 있는 특정 디렉토리를 컨테이너에 있는 디렉토리와 연결시키는 것을 의미합니다.  
컨테이너가 죽었을 때 저장되지 않은 파일들은 모두 사라지기 때문에  
볼륨마운트를 해주면 컨테이너가 종료된 후에도 데이터가 유실되는 것을 막을 수 있습니다.
