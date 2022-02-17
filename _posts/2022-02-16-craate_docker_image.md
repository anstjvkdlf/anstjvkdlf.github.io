---
title:  "도커 이미지 말아보기"
excerpt: "WSL2 리눅스에서 도커이미지 말아보기"

categories:
  - Blog
tags:
  - [Docker, K8s, Ubuntu, DockerFile, Kubernetes]

toc: true
toc_sticky: true
 
date: 2022-02-16
last_modified_at: 2022-02-16
---

# 1. 도커 이미지란?

도커 이미지는 어떤 콘테이너에서도 실행 될 수 있습니다.
무슨뜻인가 하면 도커이미지를 구웠던 환경이 우분투에서 구웠든, RedHat에서 구웠든 상관없이 말기만 하면 쿠버네티스에서 띄울 수 있다는 겁니다. 이 실습은 리눅스에서 도커 이미지는 말고 OKD(쿠버네티스) 환경에서 띄어보는 실습입니다.

실습 후에 도커 이미지가 무엇이고, 쿠버네티스 환경의 컨테이너에서 어떻게 이미지가 떠있는지 그림을 그릴 수 있으면 됩니다.

# 2. 리눅스 환경세팅
리눅스 환경에서 도커이미지를 말아볼 것이기 때문에, 가상 리눅스 Machine이 필요하다.

Microsoft Store에서 제공하는 가상 우분투를 사용해 시험해 본다.


## 2.1 Microsoft Store 우분투 다운
### 2.1.1. 먼저 WLS을 WLS2로 업그레이드 해 주어야 함
자세한 이유는 [여기](https://blog.naver.com/PostView.nhn?blogId=ilikebigmac&logNo=222007741507) 정리되어 있으니 궁금하면 읽어보면 좋을 듯 하다.

powerShell에서
```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
그리고
```bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
입력한 후 시스템을 재부팅 해준다.

재부팅 했다면 WSL2를 기본으로 설정해준다.
```bash
wsl --set-version Ubuntu 2
```

### 2.1.2. 우분투 설치


## 2.2. Docker 설치
패키지 설치
```bash
sudo apt-get update && sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

GPG Key 추가
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
Repositary 추가
```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

패키지 정상설치 확인
```bash
sudo apt-get update && sudo apt-cache search docker-ce
```

도커 설치
```bash
sudo apt-get update && sudo apt-get install docker-ce
```

사용자 추가
```bash
sudo usermod -aG docker $USER
```

설치확인
```bash
sudo docker --version
```

도커데몬 실행
```bash
sudo service docker start
```

~~Docker 삭제~~
```bash
apt-get remove docker docker-engine docker.io
```
---
## 1.3. git으로 말아볼 소스 가져오기
아래는 hello-world를 찍는 도커 오피셜 이미지 소스이다.



```bash
git clone https://github.com/anstjvkdlf/hello-world.git
```

```bash
les@DESKTOP-1JDHF6J:~/les$ git clone https://github.com/anstjvkdlf/hello-world.git
Cloning into 'hello-world'...
remote: Enumerating objects: 733, done.
remote: Counting objects: 100% (75/75), done.
remote: Compressing objects: 100% (50/50), done.
remote: Total 733 (delta 24), reused 53 (delta 12), pack-reused 658
Receiving objects: 100% (733/733), 570.84 KiB | 815.00 KiB/s, done.
Resolving deltas: 100% (304/304), done.
les@DESKTOP-1JDHF6J:~/les$
les@DESKTOP-1JDHF6J:~/les$
```

# 3. Dockerfile을 이용해 이미지 말기

## 3.1 소스 컴파일

```bash
gcc -o hello_docker hello.c
```

## 3.2. DockerFile 작성
도커 이미지는 DockerFile을 이용해서 생성할 수 있고, Dockerfile은 Makefile과 비슷하다고 생각하면 이해가 편하다. 

```bash
vi Dockerfile

FROM ubuntu:latest

COPY ./hello_docker /

CMD ["./hello_docker"]
```

## 3.3. 이미지 빌드
아래 명령어를 실행해서 이미지를 빌드 해준다.

```bash
docker build --tag new-hello-world .
```

## 3.4. 실행
docker run 명령어로 방금 빌드한 이미지로 부터 컨테이너를 생성, 실행합니다.
```bash
docker run new-hello-world
```

~~실행결과~~
```bash
 les@DESKTOP-1JDHF6J:~/les/hello-world$ docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
new-hello-world   latest    00d17f08738d   24 minutes ago   72.8MB
ubuntu            latest    54c9d81cbb44   2 weeks ago      72.8MB
les@DESKTOP-1JDHF6J:~/les/hello-world$ docker run new-hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 ```

# 4. 이미지 OKD(쿠버네티스)에 올리기
## 4.1. 이미지 Save
OKD에 올라갈 때는 tar파일로 올라가야 한다.

docker save 명령어로 이미지를 묶어준다.
```bash
docker save -o new-hello-world.tar new-hello-world:latest
```

scp, sftp등 파일전송 스크립트를 이용해서 bastion으로 tar 파일을 전송해준다.
