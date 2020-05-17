---
layout: post
title: Docker Images 다루기
tags: [Docker, Dev, Linux]
categories:
    - Docker
permalink: docker-exec.html
createat: 2020-05-17
thumbnail: /assets/images/thumbnails/docker-logo-1200x628.png
---

### Docker Image

도커 이미지란 특정 프로세스를 실행하기 위한 환경을 말합니다.  
프로세스가 실행되는 환경이란 계층화된 파일 시스템으로 도커 이미지는 파일의 집합과 같습니다.  
프로세스가 실행되는 환경(도커 이미지)도 결국 파일의 집합이라고 할 수 있습니다.

### 컨테이너를 이미지로 만들기

도커 컨테이너에는 Read Only 영역과 Writable layer 영역으로 구성됩니다.
대부분 읽기만 가능합니다.

Base Image
rootfs
Base Images

Container
rootfs
Base Image
Install Git -- Writable

Custom Image

docker run -it ubuntu:latest bash
git

### 컨테이너 변경사항 확인

```bash
docker diff <컨테이너\_ID>

docker commit <컨테이너\_ID>

docker build -t ubuntu:git2 .
-t : 이미지의 이름애찯ㄱ 지정
```

Dockerfile  
FROM
베이스 이미지 지정
FROM ubuntu:latest
ADD
파일 추가
ADD <추가할 파일> <파일이 추가될 컨테이너상 경로> : 현재 디렉토리에 있는 파일만 추가 할 수 있다.
Run
컨테이너 안에서 명령어 실행
RUN <명령어> | RUN apt-get install -y git

WORKDIR
작업 디렉터리 변경
WORKDIR <디렉토리명>  
ENV
환경벼수 기본값 지정
ENV foo bar
EXPOSE
컨테이너 실행 시 노출시킬 포트
EXPOSE <포트> : -p 옵션 함께 써야 함.
CMD
이미지 기본 실행 명령어 지정

```bash
docker run -p 3306:3306 --name mysql mysql:lastest
```
