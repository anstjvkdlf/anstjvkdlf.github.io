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

도커 이미지는 어떤 콘테이너에서도 실행 될 수 있는 이미지 입니다.

무슨뜻인가 하면 개발을 완료하고 도커 이미지를 굽게되면, 개발 환경이 우분투이던, RedHat이던 상관없이 쿠버네티스의 컨테이너에 띄울 수 있다는 겁니다. 이 실습은 리눅스에서 도커 이미지를 말아서 OKD(쿠버네티스) 환경에서 띄어보는 실습이고,
실습 후에 도커 이미지가 무엇이고, 쿠버네티스 환경의 컨테이너에서 어떻게 내가 만든 이미지가 어떤식으로 떠있는지 그림을 그릴 수 있으면 됩니다.

# 2. 리눅스 환경세팅
리눅스 환경에서 도커이미지를 말아볼 것이기 때문에, 가상 리눅스 Machine이 필요합니다.

Microsoft Store에서 제공하는 가상 우분투를 사용해 시험해 보도록 하겠습니다.



## 2.1 Microsoft Store 우분투 다운

### 2.1.1 우분투 설치
![Microsoft Store Ubuntu](https://user-images.githubusercontent.com/18244590/154407931-2a356987-6aaf-4c68-a808-ccaa02a09108.PNG)

Microsoft에서 제공하는 VM입니다. 설치 후 WLS관련하여 세팅이 필요합니다.

### 2.1.1. WSL 버전 업그레이드
MS에서 제공하는 WSL기반 우분투에서 도커를 사용하기 위해서는 WSL2를 사용하는게 마음 편합니다. 자세한 이유는 [여기](https://blog.naver.com/PostView.nhn?blogId=ilikebigmac&logNo=222007741507) 정리되어 있으니 궁금하면 읽어보면 좋을 듯 합니다.

powerShell에서 다음 2 줄을 입력해 줍니다.
```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
그리고
```bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
입력한 후 시스템을 재부팅.

재부팅 했다면 아래 명령으로 WSL2를 기본으로 설정.
```bash
wsl --set-version Ubuntu 2
```

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

~~Docker 삭제(잘못했으면 삭제 후 다시)~~
```bash
apt-get remove docker docker-engine docker.io
```
---
## 1.3. git으로 말아 볼 소스 가져오기
아래는 hello-world를 찍는 도커 오피셜 이미지 소스입니다.

git에서 땡겨 오도록 합니다.


```bash
git clone https://github.com/anstjvkdlf/hello-world.git
```

땡겨오는 중...
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
먼저 c파일을 실행파일로 컴파일 해줍니다.
```bash
gcc -o hello_docker hello.c
```

~~gcc가 없다면 gcc 설치~~
```bash
sudo apt-get install gcc
```

## 3.2. DockerFile 작성
도커 이미지는 DockerFile을 이용해서 생성할 수 있고, Dockerfile은 Makefile과 비슷하다고 생각하면 이해가 편합니다.

```bash
vi Dockerfile

FROM ubuntu:latest

COPY ./hello_docker /

CMD ["./hello_docker"]
```

## 3.3. 이미지 빌드
아래 명령어를 실행하면 Dockerfile에 명시된 내용에 따라 이미지가 빌드됩니다.
~~tag명 잘 설정해줄 것~~
```bash
docker build --tag les-hello-world .
```

## 3.4. 실행
docker run 명령어로 방금 빌드한 이미지로 부터 컨테이너를 생성, 실행합니다.
```bash
docker run new-hello-world
```

~~실행결과~~
```bash
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
Docker에서 생성한 이미지를 이제 OKD로 올려보겠습니다.

## 4.1. 이미지 Save
OKD에 올라갈 때는 tar파일로 올라가야 합니다.

docker save 명령어로 이미지를 tar파일로 묶어줍니다. 

```bash
docker save -o les-hello-world.tar les-hello-world:latest
```
~~단순히 이미지를 tar로 묶는 것 외에 다른 작업들도 포함되기 때문에 tar 스크립트 말고 docker save 명령어를 사용해 줍니다.~~

## 4.2 이미지 bastion 전송
묶은 이미지 파일을 scp, sftp등의 스크립트를 이용해서 bastion으로 전송해줍니다.

# 5. bastion -> OKD 이미지 loading
bastion에 접속해줍니다.

bastion 로그인.
```bash
oc login
```

namespace를 core1로 이동해줍니다.

~~다른 프로젝트라면 core1대신 다른 이름을 적어주면 됩니다.~~
```bash
oc project core1
```

아래 명령들을 쳐줍니다.
```bash
oc registry login

podman load -i ./les-hello-world.tar

podman tag localhost/les-hello-world:latest default-route-openshift-image-registry.apps.eluon.okd.com/core1/les-hello-world:latest

podman push default-route-openshift-image-registry.apps.eluon.okd.com/core1/new-hello-world:latest

podman images
```

* 결과
```bash
OKD4.5 [root@bastion01:/root/core1] $ podman load -i ./les-hello-world.tar
Getting image source signatures
Copying blob e54a4cabb64f [--------------------------------------] 0.0b / 0.0b
Copying config 00d17f0873 done
Writing manifest to image destination
Storing signatures
Loaded image(s): localhost/les-hello-world:latest

OKD4.5 [root@bastion01:/root/core1] $ podman images | grep 'les'
localhost/les-hello-world                                                                   latest                      00d17f08738d   3 hours ago     75.2 MB

OKD4.5 [root@bastion01:/root/core1] $ podman tag localhost/les-hello-world:latest default-route-openshift-image-registry.apps.eluon.okd.com/core1/les-hello-world:latest

OKD4.5 [root@bastion01:/root/core1] $ podman push default-route-openshift-image-registry.apps.eluon.okd.com/core1/les-hello-world:latest
Getting image source signatures
Copying blob e54a4cabb64f done
Copying blob 36ffdceb4c77 skipped: already exists
Copying config 00d17f0873 done
Writing manifest to image destination
Storing signatures

```

이미지가 image streams 탭에 올라가 있습니다.
![image_stream](https://user-images.githubusercontent.com/18244590/154412001-e4239607-33db-450d-98f1-dc3c9047c9d4.PNG)

* 이미지 스트림을 pod에 올릴 때 필요한 설정들을 yaml 파일을 통해서 해줍니다.

~~vi pod2.yaml~~
```bash
apiVersion: v1
kind: Pod
metadata:
  name: les-hello-world-pod
  namespace: core1
spec:
  restartPolicy: Always
  nodeSelector:
    kubernetes.io/hostname: vm-worker01.eluon.okd.com
  containers:
    - image: image-registry.openshift-image-registry.svc:5000/core1/les-hello-world:latest
      imagePullPolicy: Always
      name: my-hello-world-container
      command: ["/bin/bash", "-c", "sleep inf"]
      securityContext: { privileged: true }
      volumeMounts:
      - mountPath: /data
        name: test-volume-core1
  volumes:
  - name: test-volume-core1
    hostPath:
      path: /opt/volume/test-volume-core1
      type: DirectoryOrCreate
```

pod(컨테이너)를 생성해줍니다.
```bash
oc create -f pod3.yaml
```

대쉬보드에서 pod 상태를 확인 합니다.
![pods](https://user-images.githubusercontent.com/18244590/154412371-aab14453-5287-40ff-bcf7-5ca4c9be86db.PNG)
