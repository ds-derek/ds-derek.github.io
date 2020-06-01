---
layout: post
title: Gitlab 을 통한 도커 이미지 배포
tags: [Docker, Dev, Linux]
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

### 3. 수동으로 2.에서 만든 이미지를 docker hub 에 push 하고 서버에서 pull 받아 컨테이너 구동 시키기

### 4. Gitlab 프로젝트를 만들고 위 (1.-3.) 과정을 gitlab-ci.yml에 정의해서 자동화하기
