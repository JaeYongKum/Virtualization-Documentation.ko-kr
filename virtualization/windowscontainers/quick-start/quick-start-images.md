---
title: 컨테이너 배포 빠른 시작 - 이미지
description: 컨테이너 배포 빠른 시작
keywords: Docker, 컨테이너
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 41fa89dcaba38d43d39681240a1a108c9250ba78
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575174"
---
# <a name="automating-builds-and-saving-images"></a>빌드 자동화 및 이미지 저장

이전 Windows Server 빠른 시작에서는 미리 만든 .Net Core 샘플에서 Windows 컨테이너를 만들었습니다. 이 연습에서는 Dockerfile에서 사용자 고유의 컨테이너 이미지를 작성 하 고 Docker 허브 공개 레지스트리에 컨테이너 이미지를 저장 하는 방법을 보여 줍니다.

이 빠른 시작은 Windows Server 2019 또는 Windows Server 2016에서 Windows Server 컨테이너와 관련이 및 Windows Server Core 컨테이너 기본 이미지를 사용 합니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

## <a name="prerequisites"></a>필수 구성 요소

다음 요구 사항을 충족 하는지 확인 하세요.

- 한 대의 컴퓨터 시스템 (물리적 또는 가상) 실행 중인 Windows Server 2019 또는 Windows Server 2016입니다.
- Windows 컨테이너 기능 및 Docker로이 시스템을 구성 합니다. 이러한 단계에 연습에서는 [Windows Server의 Windows 컨테이너](./quick-start-windows-server.md)를 참조 하세요.
- Docker ID는 Docker 허브에 컨테이너 이미지를 푸시하기 위해 사용됩니다. Docker ID가 없는 경우 [Docker Cloud](https://cloud.docker.com/)(Docker 클라우드)에서 하나 등록하세요.

## <a name="container-image---dockerfile"></a>컨테이너 이미지-Dockerfile

컨테이너를 수동으로 만들고, 수정한 다음 새 컨테이너 이미지에 캡처할 수는 있지만 Docker에는 Dockerfile을 사용하여 이 프로세스를 자동화하는 방법이 포합됩니다. 이 연습에서는 Docker ID가 필요합니다. Docker ID가 없는 경우 [Docker Cloud]( https://cloud.docker.com/)(Docker 클라우드)에서 하나 등록하세요.

컨테이너 호스트에서 `c:\build` 디렉터리를 만들고, 이 디렉터리에 `Dockerfile`라는 파일을 만듭니다. 참고 – 파일에는 파일 확장명이 없어야 합니다.

```console
powershell new-item c:\build\Dockerfile -Force
```

메모장에서 Dockerfile을 엽니다.

```console
notepad c:\build\Dockerfile
```

Dockerfile에 다음 텍스트를 복사하고 파일을 저장합니다. 이러한 명령은 `microsoft/iis`를 기반으로 사용하여 새 이미지를 만들도록 Docker에 지시합니다. 그런 다음 dockerfile은 `RUN` 명령에 지정된 명령을 실행합니다. 이 경우 index.html 파일이 새 내용으로 업데이트됩니다.

Dockerfile에 대한 자세한 내용은 [Windows의 Dockerfile](../manage-docker/manage-windows-dockerfile.md)을 참조하세요.

```dockerfile
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

`docker build` 명령은 이미지 빌드 프로세스를 시작합니다. `-t` 매개 변수는 새 이미지의 이름을 `iis-dockerfile`로 지정하도록 빌드 프로세스에 지시합니다. **Docker 계정의 사용자 이름으로 '사용자'를 대체**합니다. Docker 계정이 없는 경우 [Docker Cloud](https://cloud.docker.com/)(Docker 클라우드)에서 하나 등록하세요.

```console
docker build -t <user>/iis-dockerfile c:\Build
```

완료되면 `docker images` 명령을 사용하여 이미지가 만들어졌는지 확인할 수 있습니다.

```console
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

이제 다음 명령을 사용하여 컨테이너를 배포하고 Docker ID로 사용자를 다시 대체합니다.

```console
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

컨테이너를 만든 후 컨테이너 호스트의 IP 주소로 이동합니다. hello world 응용 프로그램에 표시됩니다.

![](media/dockerfile2.png)

다시 컨테이너 호스트에서 `docker ps`를 사용하여 컨테이너의 이름을 가져오고 `docker rm`을 사용하여 컨테이너를 제거합니다. 참고 - 이 예제에 있는 컨테이너의 이름을 실제 컨테이너 이름으로 바꿉니다.

컨테이너 이름을 가져옵니다.

```console
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

컨테이너를 중지 합니다.

```console
docker stop <container name>
```

컨테이너를 제거합니다.

```console
docker rm -f <container name>
```

## <a name="docker-push"></a>Docker Push

Docker 컨테이너 이미지는 컨테이너 레지스트리에 저장할 수 있습니다. 이미지를 레지스트리에 저장하면 나중에 사용하기 위해 여러 컨테이너 호스트에서 검색할 수 있습니다. Docker는 [Docker Hub](https://hub.docker.com/)(Docker 허브)에 컨테이너 이미지를 저장하기 위한 공개 레지스트리를 제공합니다.

이 연습에서는 사용자 지정 Hello World 이미지가 Docker 허브의 고유한 계정으로 푸시됩니다.

먼저 `docker login command`를 사용하여 Docker 계정에 로그인합니다.

```console
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

로그인하면 컨테이너 이미지를 Docker 허브에 푸시할 수 있습니다. 이렇게 하려면 `docker push` 명령을 사용합니다. **'사용자'를 Docker ID로 대체합니다**. 

```console
docker push <user>/iis-dockerfile
```

Docker 허브 최대 각 계층을 보이지 않게 하는 Docker를으로 docker 다른 레지스트리 (외부 계층) 또는 Docker 허브에 이미 존재 하는 계층을 건너뜁니다.  예를 들어 Microsoft 컨테이너 레지스트리 또는 개인 회사 레지스트리에서 계층에서 호스팅되는 최신 버전의 Windows Server Core, 및 Docker 허브로 푸시되 지 합니다.

이제 `docker pull`을 사용하여 Docker 허브의 컨테이너 이미지를 Windows 컨테이너 호스트에 다운로드할 수 있습니다. 이 자습서에서는 기존 이미지를 삭제한 다음 Docker 허브에서 끌어옵니다. 

```console
docker rmi <user>/iis-dockerfile
```

`docker images`를 실행하면 이미지가 제거된 것으로 표시됩니다.

```console
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

마지막으로 docker pull을 사용하여 이미지를 다시 컨테이너 호스트로 끌어올 수 있습니다. Docker 계정의 사용자 이름으로 사용자를 대체합니다. 

```
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>다음 단계

샘플 ASP.NET 응용 프로그램을 패키징하는 방법을 살펴보려면 아래에 링크된 Windows 10 자습서를 방문하세요.

> [!div class="nextstepaction"]
> [Windows 10의 컨테이너](./quick-start-windows-10.md)
