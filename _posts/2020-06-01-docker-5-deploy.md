---
layout: post
title: Gitlab 을 통한 도커 이미지 배포
tags: [Docker, Dev, Linux, Gitlab]
categories:
    - Docker
    - Gitlab
    - Deploy
permalink: docker-gitlab-deploy.html
createat: 2020-06-01
thumbnail: /assets/images/thumbnails/docker-logo-1200x628.png
---

# Gitlab 을 통해 Docker 이미지 배포

프로젝트를 서버에 배포 할 때 Gitlab을 이용하면 이미지 빌드와 실행 과정을 자동화 할 수 있습니다.  
예제로 React 프로젝트를 nginx 컨테이너를 이용하여 서버에 배포해 보겠습니다.

단계적으로 진행하기 위해 다음과 같이 진행하겠습니다.

1. React 프로젝트 빌드 이미지 만들기
2. Nginx 이미지를 1.에서 빌드된 React 소스와 함께 이미지로 빌드하기
3. 수동으로 2.에서 만든 이미지를 docker hub 에 push 하고 서버에서 pull 받아 컨테이너 구동 시키기
4. Gitlab 프로젝트를 만들고 위 (1.-3.) 과정을 gitlab-ci.yml에 정의해서 자동화하기

### 1. React 프로젝트 빌드 이미지 만들기

배포할 리액트 프로젝트를 만듭니다.  
node 와 npm은 설치되어있는 것으로 가정하겠습니다.

```bash
$ npm create-react-app my-app
```

꼭 create-react-app 을 이용하지 않고 webpack 등을 이용해서 프로젝트를 구성해도 상관없습니다.

### 2. Nginx 이미지를 1.에서 빌드된 React 소스와 함께 이미지로 빌드하기

nginx 로 서비스 할 것이기 때문에 Dockerfile 에서 Base Image 를 nginx 를 이용하면 됩니다.  
그런데 먼저 nginx 컨테이너 안에 서비스할 react project 가 빌드되어 있어야 합니다.  
이 것을 먼저 수행하는 방식은 몇가지 있겠지만 이 예제에서는 multi stage 를 이용하겠습니다.   

node 를 Base 이미지로 한 컨테이너에서 node 와 npm 으로 build 를 먼저 수행하고  
결과물을 nginx 이미지가 만들어질 때 연결해주는 방식 입니다.

```dockerfile
# Base Image
FROM node:12.2.0-alpine as builder

# root 에 app 폴더를 생성
RUN mkdir /app

# work dir 고정
WORKDIR /app

# app dependencies
COPY package.json /app/package.json
RUN npm install --silent

# 호스트 작업파일을 이미지 안으로 복사
COPY . /app

# 작업 빌드
RUN npm run build

# nginx 이미지를 사용
FROM nginx

# work dir 에 build 폴더 생성 /app/dist
RUN mkdir ./dist

# build 이미지의 dist 폴더를 workdir 의 dist 폴더로 복사
COPY --from=builder /app/dist ./dist

COPY ./src/assets/images /dist/assets/images

# nginx 의 default.conf 를 삭제
RUN rm /etc/nginx/conf.d/default.conf

# host pc 의 nginx.conf 를 아래 경로에 복사
COPY ./nginx.conf /etc/nginx/conf.d

# 80 포트 오픈
EXPOSE 80

# container 실행 시 자동으로 실행할 command. nginx 시작함
CMD ["nginx", "-g", "daemon off;"]
```  
builder 로 수행된 컨테이너는 이미지가 만들어지지 않고 nginx 이미지로 결과물을 연결시켜주는 기능을 합니다.  
최종적으로는 build 된 프로젝트를 가지고 있는 nginx 이미지가 생성이 됩니다.  

그리고 nginx 컨테이너에 설정을 외부에서 주입하기 위해 nginx.conf 파일을 예제와 같이 연결 시켜 줍니다.
react 를 서비스 하기 위한 nginx.conf 파일은 다음과 같습니다.

```nginx.conf
server {
    listen 80;
    location / {
        root    /dist;
        index   index.html;
        try_files $uri $uri/ /index.html;
    }
}
``` 
도커 이미지 빌드시에 빌드된 프로젝트를 nginx 컨테이너의 dist에 넣어 주었기 떄문에 nginx.conf 에서 root 를 /dist 디렉토리로 설정해 주었습니다.  
또 try_files 에는 우선 적으로 요청한 url에 대한 파일이 존재한다면 연결하고  
그렇지 않다면 index.html에서 react router를 통해 서비스 되도록 설정해 줍니다.  

### 3. 수동으로 2.에서 만든 이미지를 docker hub 에 push 하고 서버에서 pull 받아 컨테이너 구동 시키기
자 여기까지 작업이 되었다면 도커 이미지를 수동으로 생성 할 수 있게 되었습니다.  
프로젝트 경로에서 다음과 같이 빌드 명령을 하면 준비가 완료됩니다.  
```bash
$ docker build -t myproject:latest .
```

-t 옵션을 이용해서 태그와 함께 이미지를 생성해 줍니다.    
docker images 명령으로 이미지 목록을 확인하면 방금 입력했던 태그네임으로 생성된 이미지를 목록에서 볼 수 있습니다.
이 이미지를 가지고 컨테이너를 실행하면 로컬호스트에서 동작되는 것을 확인 할 수 있습니다.

```bash
$ docker run -d --name=runningproejct --restart=always myproject:latest 
```
-d 옵션은 백그라운드 실행을 의미합니다.  
--name 옵션으로 컨테이너에 이름을 지정해 주었습니다.  
--restart=always 옵션은 서버가 재부팅 될 때 컨테이너도 실행 시켜줍니다.  

여기 까지 작업이 완료되었다면 docker hub 혹은 docker registry 을 이용해서 이미지를 배포할 수 있을 것입니다.  
하지만 깃랩을 이용하면 이 부분도 자동화 할 수 있습니다. 

### 4. Gitlab 프로젝트를 만들고 위 (1.-3.) gitlab-runner 등록 (register) 하기
깃랩에 특정 브런치에 push 혹은 merge 될 때마다 도커 빌드와 배포를 하기 위해서는 gitlab runner 와 gitlab-ci.yml파일이 필요합니다.  
gitlab runner 의 경우 배포할 서버에 설치해야 하는데 만약 서버가 개발서버, 운영서버 2대 있다면 2대 모두 설치해 주어야 합니다.  
gitlab-ci.yml 은 프로젝트 루트경로에 Dockerfile과 함께 생성합니다.  

우선 gitlab-runner 를 설치하기 위해서 서버에 ssh 등을 이용해 접속합니다.  
gitlab-runner 를 지원하는 executor는 몇 가지가 있는데 다음과 같습니다.  
* SSH
* Shell
* Parallels
* VirtualBox
* Docker
* Docker Machine (auto-scaling)
* Kubernetes
* Custom

이번 예제에서는 간단히 Shell 을 이용해 진행 해보겠습니다.  
docker를 이용해 docker in docker 로 서비스 하는 예제 포스팅도 많았으나 
저의 경우는 여러 까다로운 설정(접근 권한 등..)들 때문에 Shell 을 이용했습니다.    

1. 리눅스 버전에 따라 설치 파일을 다운로드

```bash
# Linux x86-64
$ sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Linux x86
$ sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-386

# Linux arm
$ sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-arm

# Linux arm64
$ sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-arm64
```

2. 디렉토리 실행 권한 설정
```bash
$ sudo chmod +x /usr/local/bin/gitlab-runner
```

3. Gitlab CI 프로세스를 실행시킬 user 생성
```bash
$ sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
```

4. 인스톨 및 프로세스 실행
```bash
$ sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
$ sudo gitlab-runner start
```

설치가 되었다면 이제 gitlab 러너를 gitlab 저장소에 등록해야 합니다.  
등록을 위해서는 자신의 프로젝트 저장소에서 토큰을 발급받아야 하는데 gitlab 접속 후 프로젝트로 이동하여  
Setting > CI/CD 메뉴로 들어갑니다.  
그리고 항목들 중 Runners 항목의 오른쪽 Expand 버튼을 클릭하면 디테일이 보이는데, 
Specific Runners 에 Set up a specific Runner manually 부분에 보면 3번째 항목이 이 토큰입니다.  
토큰을 확인했다면 다시 서버로 돌아와 토큰으로 러너를 등록해 줍니다. 

1. 아래와 같이 입력하면 등록이 진행 됩니다. 
```bash
$ gitlab-runner register
```
2. 그리고 gitlab url 을 입력하라고 나오는데 공식 gitlab을 이용하기 때문에 gitlab url 을 입력해 줍니다.  
만약 외부의 다른 깃랩서버를 이용하신다면 그 주소를 입력해 주시면 됩니다. 
```bash
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com  <-- url 입력
```
3. 토큰을 입력하라는 메시지가 나오면 프로젝트 저장소에서 확인한 토큰을 입력합니다.  
```bash
Please enter the gitlab-ci token for this runner
프로젝트에서 copy한 토큰 입력
``` 
4. 추가로 description 을 입력해 줍니다. (이 부분은 추가정보이기 떄문에 아무거나 입력해도 상관없습니다.)
```bash
Please enter the gitlab-ci description for this runner
[hostname] my-runner
```
5. 그 뒤에 tags 를 입력하라고 나오는데 gitlab-ci.yml 파일에서 정확히 일치해야 하니 주의해서 입력해서 줍니다.  
하나만 입력해도 되고, 콤마를 이용해 1개 이상 입력할 수도 있습니다.
```bash
Please enter the gitlab-ci tags for this runner (comma separated):
my-tag,another-tag
```

6.executor 를 선택하라고 하는데 Shell 을 입력해 줍니다. 
```bash
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
Shell
```

여기 까지 진행했다면 gitlab-runner 가 프로젝트와 연동이 된 것을 확인할 수 있습니다.  
gitlab 프로젝트의 runners 토큰 부분을 확인하면 태그별로 리스트가 보이는 녹색불이 보인다면 성공한 것입니다.  
간혹 러너가 여기서 확인이 안되는 경우가 있는데 다음 명령을 통해 러너가 정상적으로 떠있는지 확인하거나 재시작하면 gitlab 에서 확인이 완료 됩니다. 
```bash
$ sudo gitlab-runner status // 상태확인 
$ sudo gitlab-runner restart // 재시작
$ sudo gitlab-runner verify // 등록된 러너 확인
```
 
 이제 연동된 gitlab-runner 를 gitlab-ci.yml 파일로 동작하도록 해주면 자동배포 구축이 완료됩니다.  
 
 ### 5.gitlab-ci.yml 작성
 
```yaml
stages:
    - test
    - dockernize
    - deploy
variables:
    IMAGE_NAME: @이미지명:태그

test:
    stage: test
    script:
        - echo "skip test."
        - docker ps -a
    tags :
        - release

dockernize:
    only:
        - develop
    stage: dockernize

    script:
        - docker container ls -a
        - docker build -f Dockerfile.dev -t $IMAGE_NAME .
        - docker image prune -f
    tags:
        - develop

deploy:
    only:
        - master
    stage: deploy

    script:
        - docker container ls -a
        - docker container rm -f gio-client
        - docker run -d -p 80:80 --name=gio-client --restart always $IMAGE_NAME
        - docker container ls -a

    tags:
        - release
```
 