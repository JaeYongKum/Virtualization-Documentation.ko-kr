---
title: "컨테이너 배포 빠른 시작 - 이미지"
description: "컨테이너 배포 빠른 시작"
keywords: "Docker, 컨테이너"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
translationtype: Human Translation
ms.sourcegitcommit: 5d746be15856aad683bf684d2eef573d732ab457
ms.openlocfilehash: 6add396bea629d5438cde5892458f6c8405bf644

---

# Windows Server의 컨테이너 이미지

이전 Windows Server 빠른 시작에서는 미리 만든 .Net Core 샘플에서 Windows 컨테이너를 만들었습니다. 이 연습에서는 수동으로 사용자 지정 컨테이너 이미지 만들기, Dockerfile을 사용하여 컨테이너 이미지 생성 자동화, Docker 허브 공개 레지스트리에 컨테이너 이미지 저장에 대해 자세히 알아봅니다.

이 빠른 시작은 Windows Server 2016의 Windows Server 컨테이너 전용이며 Windows Server Core 컨테이너 기본 이미지를 사용합니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

**필수 구성 요소:**

- Windows Server 2016이 실행되는 컴퓨터 시스템(물리적 또는 가상) 1대.
- Windows 컨테이너 기능 및 Docker로 이 시스템을 구성합니다. 이러한 단계에 대한 연습은 [Windows Server의 Windows 컨테이너](./quick-start-windows-server.md)를 참조하세요.
- Docker ID는 Docker 허브에 컨테이너 이미지를 푸시하기 위해 사용됩니다. Docker ID가 없는 경우 [Docker Cloud](https://cloud.docker.com/)(Docker 클라우드)에서 하나 등록하세요.

## 1. 컨테이너 이미지 - 수동

최상의 환경을 위해 Windows 명령 셸(cmd.exe)에서 이 연습 과정을 안내합니다.

수동으로 컨테이너 이미지를 만드는 첫 번째 단계는 컨테이너를 배포하는 것입니다. 이 예제에서는 미리 만들어진 IIS 이미지에서 IIS 컨테이너를 배포합니다. 컨테이너가 배포된 다음에는 컨테이너 내의 셸 세션에서 작업합니다. 대화형 세션은 `-it` 플래그로 시작됩니다. Docker Run 명령에 대한 자세한 내용은 [Docker.com의 Docker Run Reference(Docker Run 참조)](https://docs.docker.com/engine/reference/run/)를 참조하세요. 

> 이 단계는 Windows Server Core 기본 이미지의 크기 때문에 다소 시간이 걸릴 수 있습니다.

```none
docker run -d --name myIIS -p 80:80 microsoft/iis
```

이제 컨테이너가 백그라운드에서 실행됩니다. 컴퓨터에 포함된 기본 명령인 `ServiceMonitor.exe`는 IIS 진행 상황을 모니터링하고 IIS가 중지되면 컨테이너를 자동으로 중지합니다. 이 이미지를 어떻게 만드는지 자세히 알아보려면 GitHub에서 [Microsoft/docker-iis](https://github.com/Microsoft/iis-docker)를 참조하세요.

다음, 컨테이너에서 대화식 cmd를 시작합니다. 이렇게 하면 IIS 또는 ServiceMonitor를 중지하지 않고 실행 중인 컨테이너에서 명령을 실행할 수 있습니다.

```none
docker exec -i myIIS cmd 
```

그런 다음 실행 중인 컨테이너를 변경할 수 있습니다. 다음 명령을 실행하여 IIS 시작 화면을 제거합니다.

```none
del C:\inetpub\wwwroot\iisstart.htm
```

다음 명령을 실행하여 기본 IIS 사이트를 새 정적 사이트로 바꿉니다.

```none
echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

다른 시스템에서 컨테이너 호스트의 IP 주소를 찾습니다. 이제 ‘Hello World’ 응용 프로그램이 표시됩니다.

**참고:** Azure에서 작업하는 경우 포트 80을 통한 트래픽을 허용하는 네트워크 보안 그룹 규칙이 있어야 합니다. 자세한 내용은 [네트워크 보안 그룹에서 규칙 만들기](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)를 참조하세요.

![](media/hello.png)

다시 컨테이너에서 대화형 컨테이너 세션을 종료합니다.

```none
exit
```

이제 수정된 컨테이너를 새 컨테이너 이미지로 캡처할 수 있습니다. 이렇게 하려면 컨테이너 이름이 필요합니다. `docker ps -a` 명령으로 찾을 수 있습니다.

```none
docker ps -a

CONTAINER ID     IMAGE                             COMMAND   CREATED             STATUS   PORTS   NAMES
489b0b447949     microsoft/iis   "cmd"     About an hour ago   Exited           pedantic_lichterman
```

새 컨테이너 이미지를 만들려면 `docker commit` 명령을 사용합니다. Docker 커밋은 “docker commit container-name new-image-name” 형식을 사용합니다. 참고 - 이 예제에 있는 컨테이너의 이름을 실제 컨테이너 이름으로 바꿉니다.

```none
docker commit pedantic_lichterman modified-iis
```

새 이미지가 만들어졌는지 확인하려면 `docker images` 명령을 사용합니다.  

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
modified-iis        latest              3e4fdb6ed3bc        About a minute ago   10.17 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago          9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago          9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago          9.344 GB
```

이제 이 이미지를 배포할 수 있습니다. 결과 컨테이너에 캡처된 모든 수정 내용이 포함됩니다.

## 2. 컨테이너 이미지 - Dockerfile

마지막 연습에서 컨테이너를 수동으로 만들어 수정한 다음 새 컨테이너 이미지에 캡처했습니다. Docker에는 Dockerfile을 사용하여 이 프로세스를 자동화하는 방법이 포함되어 있습니다. 이 연습의 결과는 지난 연습과 거의 동일하지만 이번에는 프로세스가 자동화됩니다. 이 연습에서는 Docker ID가 필요합니다. Docker ID가 없는 경우 [Docker Cloud]( https://cloud.docker.com/)(Docker 클라우드)에서 하나 등록하세요.

컨테이너 호스트에서 `c:\build` 디렉터리를 만들고, 이 디렉터리에 `Dockerfile`라는 파일을 만듭니다. 참고 – 파일에는 파일 확장명이 없어야 합니다.

```none
powershell new-item c:\build\Dockerfile -Force
```

메모장에서 Dockerfile을 엽니다.

```none
notepad c:\build\Dockerfile
```

Dockerfile에 다음 텍스트를 복사하고 파일을 저장합니다. 이러한 명령은 `microsoft/iis`를 기반으로 사용하여 새 이미지를 만들도록 Docker에 지시합니다. 그런 다음 dockerfile은 `RUN` 명령에 지정된 명령을 실행합니다. 이 경우 index.html 파일이 새 내용으로 업데이트됩니다. 

Dockerfile에 대한 자세한 내용은 [Windows의 Dockerfile](../manage-docker/manage-windows-dockerfile.md)을 참조하세요.

```none
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

`docker build` 명령은 이미지 빌드 프로세스를 시작합니다. `-t` 매개 변수는 새 이미지의 이름을 `iis-dockerfile`로 지정하도록 빌드 프로세스에 지시합니다. **Docker 계정의 사용자 이름으로 '사용자'를 대체**합니다. Docker 계정이 없는 경우 [Docker Cloud](https://cloud.docker.com/)(Docker 클라우드)에서 하나 등록하세요.

```none
docker build -t <user>/iis-dockerfile c:\Build
```

완료되면 `docker images` 명령을 사용하여 이미지가 만들어졌는지 확인할 수 있습니다.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

이제 다음 명령을 사용하여 컨테이너를 배포하고 Docker ID로 사용자를 다시 대체합니다.

```none
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

컨테이너를 만든 후 컨테이너 호스트의 IP 주소로 이동합니다. hello world 응용 프로그램에 표시됩니다.

![](media/dockerfile2.png)

다시 컨테이너 호스트에서 `docker ps`를 사용하여 컨테이너의 이름을 가져오고 `docker rm`을 사용하여 컨테이너를 제거합니다. 참고 - 이 예제에 있는 컨테이너의 이름을 실제 컨테이너 이름으로 바꿉니다.

컨테이너 이름을 가져옵니다.

```none
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

컨테이너를 제거합니다.

```none
docker rm -f <container name>
```

## 3. Docker Push

Docker 컨테이너 이미지는 컨테이너 레지스트리에 저장할 수 있습니다. 이미지를 레지스트리에 저장하면 나중에 사용하기 위해 여러 컨테이너 호스트에서 검색할 수 있습니다. Docker는 [Docker Hub](https://hub.docker.com/)(Docker 허브)에 컨테이너 이미지를 저장하기 위한 공개 레지스트리를 제공합니다.

이 연습에서는 사용자 지정 Hello World 이미지가 Docker 허브의 고유한 계정으로 푸시됩니다.

먼저 `docker login command`를 사용하여 Docker 계정에 로그인합니다.

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

로그인하면 컨테이너 이미지를 Docker 허브에 푸시할 수 있습니다. 이렇게 하려면 `docker push` 명령을 사용합니다. **'사용자'를 Docker ID로 대체합니다**. 

```none
docker push <user>/iis-dockerfile
```

이제 `docker pull`을 사용하여 Docker 허브의 컨테이너 이미지를 Windows 컨테이너 호스트에 다운로드할 수 있습니다. 이 자습서에서는 기존 이미지를 삭제한 다음 Docker 허브에서 끌어옵니다. 

```none
docker rmi <user>/iis-dockerfile
```

`docker images`를 실행하면 이미지가 제거된 것으로 표시됩니다.

```none
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

마지막으로 docker pull을 사용하여 이미지를 다시 컨테이너 호스트로 끌어올 수 있습니다. Docker 계정의 사용자 이름으로 사용자를 대체합니다. 

```none
docker pull <user>/iis-dockerfile
```

## 다음 단계

[Windows 10의 Windows 컨테이너](./quick-start-windows-10.md)



<!--HONumber=Jan17_HO4-->


