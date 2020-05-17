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

도커 컨테이너에는 Read Only 영역과 Writable layer 영역으로 구성되며 대부분 읽기 전용입니다.  
컨테이너를 실행 시킨 후 컨테이너 안에서 새로운 프로그램을 설치하게 되면 이미지의 Writable layer 영역에 쓰여지게 됩니다.  
이 상태에서 컨테이너를 commit 하면 설치한 Writable layer 까지 포함된 새로운 이미지를 만들 수 있습니다.

<div class="columns">
    <div class="column is-2">
        <table>
            <thead>
                <tr>
                    <th>Base Image</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>rootfs</td>
                </tr>
                <tr>
                    <td>Base Image (linux kernel...)</td>
                </tr>
            </tbody>
        </table>
    </div>
    <div class="column is-2">
        <table class="table">
            <tr>
                <td> run -> </td>
            </tr>
        </table>
    </div>
    <div class="column is-2">
        <table>
            <thead>
                <tr>
                    <th>Container</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>rootfs</td>
                </tr>
                <tr>
                    <td>Base Image</td>
                </tr>
                <tr>
                    <td>Install Some Program (Writable) </td>
                </tr>
            </tbody>
        </table>
    </div>
    <div class="column is-2">
        <table>
            <tr>
                <td> commit -> </td>
            </tr>
        </table>
    </div>
    <div class="column is-2">
        <table>
            <thead>
                <tr>
                    <th>Custom Image</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>rootfs</td>
                </tr>
                <tr>
                    <td>Base Image</td>
                </tr>
                <tr>
                    <td>Installed Some Program</td>
                </tr>
            </tbody>
        </table>
    </div>
</div>

컨테이너 안에서 git 설치 예제

```bash
$ docker run -it ubuntu:18.04 bash
$ apt-get update
$ apt-get install -y git
```

### 컨테이너 변경사항 확인

위 예제에서 ubuntu:18.04 를 베이스 이미지로 실행시킨 컨테이너에 git을 설치하였습니다.  
이 상태에서 컨테이너를 종료하게 되면 설치했던 git은 사라지게 됩니다.  
bash 를 빠져나오면 컨테이너가 종료되기 때문에 아래 커맨드는 새로운 터미널에서 실행해 줍니다.

```bash
$ docker ps
$ docker diff <컨테이너_ID>
$ docker commit <컨테이너_ID> ubuntu:git
```

docker ps 명령어를 통해 현재 실행 중인 ubuntu:18.04 컨테이너 아이디를 확인합니다.  
docker diff 명령어를 통해서 컨테이너에 어떤 추가적인 작업이 있었는지 확인할 수 있습니다.  
docker commit 명령어와 적절한 태그 이름(여기서는 ubuntu:git)을 입력하면  
지정한 태그 이름으로 지금까지 작업내역을 가지고 있는 새로운 이미지가 생성됩니다.

### Dockerfile

docker commit 명령어를 이용하는 것 보다 Dockerfile을 이용하면 일련의 작업을 미리 지정하여 실행할 수 있기 때문에 더 편리합니다.  
Dockerfile 이란 이미지 생성 과정을 단계별로 기술한 Docker 전용 DSL 입니다.

```Dockerfile
FROM ubnutu:18.04

RUN apt-get update
RUN apt-get instll -y git
```

```bash
$ docker build -t ubuntu:git02 .
```

위 Dockerfile을 확인하면 ubuntu:18.04를 베이스로 하여 git까지 설치 한 것을 볼 수 있습니다.  
command 를 이용한 상단의 예제와 동일한 동작을 하고  
이 것을 docker build 명령어를 입력하면 이미지가 생성됩니다.  
이 때 명령어 마지막에 경로를 반드시 명시해야 하는데, 입력한 경로 아래서 Dockerfile을 찾기 때문입니다.

### FROM

베이스 이미지 지정

```Dockerfile
FROM ubuntu:16.04
```

### ADD

파일 추가  
현재 디렉토리에 있는 파일만 추가 할 수 있다.

```Dockerfile
ADD <추가할 파일> <파일이 추가될 컨테이너상 경로>
ADD data.txt /tmp/data.txt
```

### RUN

컨테이너 생성한 후 그 안에서 명령어 실행

```Dockerfile
RUN <명령어>
RUN apt-get install -y git
```

### WORKDIR

작업 디렉터리 변경

```Dockerfile
WORKDIR <디렉토리명>
```

### ENV

환경변수 기본값 지정

```Dockerfile
ENV foo bar
```

### EXPOSE

컨테이너 실행 시 노출시킬 포트를 지정합니다.
도커가 자동으로 포트를 열어줍니다.
이 옵션을 사용하더라도 -p 옵션은 지정해 주어야 합니다.

```Dockerfile
EXPOSE <포트> : -p 옵션 함께 써야 함.
```

### CMD

컨테이너가 생성될 때 기본으로 실행될 명령어를 지정할 때 사용합니다.
