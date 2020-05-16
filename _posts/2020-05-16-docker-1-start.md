---
layout: post
title: 내 PC 에서 Docker 시작하기
tags: [Docker, Dev, Linux]
categories:
    - Docker
permalink: docker-start.html
createat: 2020-05-16
---

# 내 PC에 도커를 설치해 보자

도커는 VM 과는 다른 리눅스 컨테이너 기반의 프로세스 격리 기술입니다.

도커는 리눅스만 지원하기 때문에 macOs와 windows에 설치되어 있는 도커는 실제로 가상머신에 설치 됩니다.  
결국 리눅스가 아니면 리눅스에 대한 에뮬레이션이 필요합니다.  
도커에서는 이것을 위해 도커 데스크탑 프로그램을 제공하고 있습니다.  
MacOs 는 xhyve windows 는 Hyper-V 가상화 기술을 사용합니다.  
그리고 그 에뮬레이션 된 리눅스 위에서 컨테이너(프로세스)들을 구동 시킬 수 있습니다.  
리눅스 사용자가 아니라면 사용하는 OS에 필요한 도커 데스크탑 프로그램을 다운로드 받아 설치해야 합니다.

### download link

[docker for windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows/)  
[docker for macOs](https://hub.docker.com/editions/community/docker-ce-desktop-mac)

# windows 필요 사항

윈도우의 경우 윈도우7 이상, professional 버전이 필요합니다.  
또한 OS 가상화를 위해 Hyper-V 기능을 On 해주어야 합니다.

1. 프로그램 및 기능
2. windows 기능 켜기/끄기
3. 항목 중 Hyper-V 기능 사용 체크

혹은 관리자 권한으로 cmd / powershell 을 열고 아래와 같이 커맨드를 입력합니다.

```bash
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

# wsl 에서 사용

wsl에 대한 대략적인 사용방법은 아래 포스트를 참고하실 수 있습니다.  
[윈도우에 리눅스 개발 환경 구축하기](wsl-start.html)

wsl 의 경우 완전한 리눅스 커널이 아니기 때문에 바로 사용은 안되고,  
윈도우용 도커 데스크탑을 먼저 설치하고, wsl 안에서도 도커를 설치하면 서로 연동이 되어 사용할 수 있습니다.  
windows & wsl 도커 연결을 위해서는 windows docker에 설정에서  
General > Expose daemon on tcp://localhost:2375 without TLS 옵션을 체크해야 합니다.

<div style="text-align:center;">
        <img src="/assets/images/posts/2020-05-16/running-docker-and-azure-cli-from-wsl-05.png">
</div>

wsl2의 경우 별도로 윈도우용 도커 데스크탑을 설치할 필요가 없습니다.

### get started

각 OS에 맞는 도커 데스크탑 설치를 완료했다면 다음과 같이 hello world 를 실행 할 수 있습니다.

```bash
docker run -it ubuntu:lastest echo "hello world"
```

짝짝짝! 축하합니다. 처음으로 도커를 이용하여 hello world를 실행시켰습니다.

아래와 같이 입력하면 도커안에서 실행된 bash 안으로 들어갈 수 있습니다.

```bash
docker run -it ubuntu:lastest bash
```

docker를 이용해서 ubuntu 이미지를 생성 후 컨테이너를 실행해서
-it 옵션으로 입출력이 가능하게 합니다.  
그 후 bash 를 실행하여 command를 도커 안에 ubuntu 프로세스에서 입력할 수 있도록 합니다.

더 자세한 도커 사용방법은 다음 포스트에서 소개하겠습니다.

감사합니다.
