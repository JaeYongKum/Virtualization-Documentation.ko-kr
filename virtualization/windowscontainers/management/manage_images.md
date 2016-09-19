---
title: "Windows 컨테이너 이미지"
description: "Windows 컨테이너를 사용하여 컨테이너 이미지를 만들고 관리합니다."
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
redirect_url: https://docs.docker.com/v1.8/userguide/dockerimages/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 49e29949fc91533cf1d3ed0aef47e829276772f4

---

# Windows 컨테이너 이미지

**이 예비 콘텐츠는 변경될 수 있습니다.** 

>Windows 컨테이너는 Docker를 통해 관리됩니다. Windows 컨테이너 설명서는 [docs.docker.com](https://docs.docker.com/)설명서를 보완합니다.

컨테이너 이미지는 컨테이너를 배포하는 데 사용됩니다. 이러한 이미지는 응용 프로그램 및 모든 응용 프로그램 종속성을 포함할 수 있습니다. 예를 들어 Nano Server, IIS 및 IIS를 실행 중인 응용 프로그램으로 사전 구성한 컨테이너 이미지를 개발할 수 있습니다. 그런 다음 이 컨테이너 이미지를 나중에 사용할 수 있도록 컨테이너 레지스트리에 저장하고, 모든 Windows 컨테이너 호스트(온-프레미스, 클라우드 또는 컨테이너 서비스)에 배포하며, 새 컨테이너 이미지를 위한 기반으로 사용할 수도 있습니다.

### 이미지 설치

Windows 컨테이너를 사용하기 전에 기본 이미지를 설치해야 합니다. 기본 이미지는 기본 운영 체제로 Windows Server Core 또는 Nano Server에서 사용할 수 있습니다. 지원되는 구성에 대한 자세한 내용은 [Windows 컨테이너 시스템 요구 사항](../deployment/system_requirements.md)을 참조하세요.

Windows Server Core 기본 이미지를 설치하려면 다음을 실행합니다.

```none
docker pull microsoft/windowsservercore
```

Nano Server 기본 이미지를 설치하려면 다음을 실행합니다.

```none
docker pull microsoft/nanoserver
```

### 이미지 나열

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        9 weeks ago         7.764 GB
microsoft/nanoserver          latest              3a703c6e97a2        9 weeks ago         969.8 MB
```

### 새 이미지 만들기

모든 기존 컨테이너에서 새 컨테이너 이미지를 만들 수 있습니다. 이렇게 하려면 `docker commit` 명령을 사용합니다. 다음 예제에서는 이름이 'windowsservercoreiis'인 새 컨테이너 이미지를 만듭니다.

```none
docker commit 475059caef8f windowsservercoreiis
```

### 이미지 제거

중지 상태에서라도 컨테이너에 해당 이미지에 대한 종속성이 있으면 컨테이너 이미지를 제거할 수 없습니다.

Docke가 있는 이미지를 제거할 때는 이미지 이름 또는 ID로 이미지를 참조할 수 있습니다.

```none
docker rmi windowsservercoreiis
```

### 이미지 종속성

Docker를 사용하여 이미지 종속성을 확인하려면 `docker history` 명령을 사용할 수 있습니다.

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker 허브

Docker 허브 레지스트리는 컨테이너 호스트에 다운로드할 수 있는 미리 빌드된 이미지를 포함합니다. 이러한 이미지를 다운로드한 후에는 Windows 컨테이너 응용 프로그램의 기반으로 사용할 수 있습니다.

Docker 허브에서 사용 가능한 이미지 목록을 확인하려면 `docker search` 명령을 사용합니다. 참고 - Docker 허브에서 이러한 종속 컨테이너 이미지를 끌어오려면 먼저 Windows Serve Core 또는 Nano Server 기본 OS 이미지를 설치해야 합니다.

이러한 이미지 대부분에는 Windows Server Core 및 Nano Server 버전이 있습니다. 특정 버전을 가져오려면 ":windowsservercore" 또는 ":nanoserver" 태그를 추가하면 됩니다. Nano Server 버전만 사용할 수 있는 경우가 아니라면 "최신" 태그는 기본적으로 Windows Server Core 버전을 반환합니다.


```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

### Docker Pull

Docker 허브에서 이미지를 다운로드하려면 `docker pull`을 사용합니다. 자세한 내용은 [Docker Pull on Docker.com(Docker.com의 Docker Pull)](https://docs.docker.com/engine/reference/commandline/pull/)을 참조하세요.

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

이제 `docker images`를 실행하면 이미지가 표시됩니다.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

> Docker Pull이 실패하면 최신 누적 업데이트를 Docker 컨테이너 호스트에 적용했는지 확인합니다. TP5 업데이트는 [KB3157663]( https://support.microsoft.com/en-us/kb/3157663)에 있습니다.

### Docker Push

Docker 허브 또는 Docker Trusted Registry로 컨테이너 이미지를 업로드할 수도 있습니다. 업로드한 후 다른 Windows 컨테이너 환경에서 이러한 이미지를 다운로드하여 다시 사용할 수 있습니다.

컨테이너 이미지를 Docker 허브로 업로드하려면 먼저 레지스트리에 로그인합니다. 자세한 내용은 [Docker Login on Docker.com(Docker.com의 Docker Login)]( https://docs.docker.com/engine/reference/commandline/login/)을 참조하세요.

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password:

Login Succeeded
```

Docker 허브 또는 Docker Trusted Registry에 로그인한 후 `docker push`를 사용하여 컨테이너 이미지를 업로드합니다. 컨테이너 이미지는 이름 또는 ID로 참조할 수 있습니다. 자세한 내용은 [Docker Push on Docker.com(Docker.com의 Docker Push)]( https://docs.docker.com/engine/reference/commandline/push/)을 참조하세요.

```none
docker push username/containername

The push refers to a repository [docker.io/username/containername]
b567cea5d325: Pushed
00f57025c723: Pushed
2e05e94480e9: Pushed
63f3aa135163: Pushed
469f4bf35316: Pushed
2946c9dcfc7d: Pushed
7bfd967a5e43: Pushed
f64ea92aaebc: Pushed
4341be770beb: Pushed
fed398573696: Pushed
latest: digest: sha256:ae3a2971628c04d5df32c3bbbfc87c477bb814d5e73e2787900da13228676c4f size: 2410
```

이제 컨테이너 이미지를 사용할 수 있고 `docker pull`로 액세스할 수 있습니다.






<!--HONumber=Sep16_HO2-->


