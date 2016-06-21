---
title: Windows 컨테이너 이미지
description: Windows 컨테이너를 사용하여 컨테이너 이미지를 만들고 관리합니다.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
---

# Windows 컨테이너 이미지

**이 예비 콘텐츠는 변경될 수 있습니다.** 

컨테이너 이미지는 컨테이너를 배포하는 데 사용됩니다. 이러한 이미지는 운영 체제, 응용 프로그램 및 모든 응용 프로그램 종속성을 포함할 수 있습니다. 예를 들어 Nano Server, IIS 및 IIS를 실행 중인 응용 프로그램으로 사전 구성한 컨테이너 이미지를 개발할 수 있습니다. 그런 다음 이 컨테이너 이미지를 나중에 사용할 수 있도록 컨테이너 레지스트리에 저장하고, 모든 Windows 컨테이너 호스트(온-프레미스, 클라우드 또는 컨테이너 서비스)에 배포하며, 새 컨테이너 이미지를 위한 기반으로 사용할 수도 있습니다.

컨테이너 이미지에는 다음과 같은 두 가지 유형이 있습니다.

**기본 OS 이미지** - Microsoft에서 제공하고 핵심 OS 구성 요소를 포함합니다. 

**컨테이너 이미지** - 기본 OS 이미지에서 파생된 사용자 지정 컨테이너 이미지입니다.

## 기본 OS 이미지

### 이미지 설치

컨테이너 OS 이미지는 ContainerImag PowerShell 모듈을 사용하여 확인하고 설치할 수 있습니다. 이 모듈을 사용하기 전에 먼저 모듈을 설치해야 합니다. 다음 명령을 사용하여 모듈을 설치할 수 있습니다. 컨테이너 이미지 OneGet PowerShell 모듈 사용에 대한 자세한 내용은 [컨테이너 이미지 공급자](https://github.com/PowerShell/ContainerProvider)를 참조하세요. 

```none
Install-PackageProvider ContainerImage -Force
```

설치되면 `Find-ContainerImage`를 사용하여 기본 OS 이미지 목록을 반환할 수 있습니다.

```none
Find-ContainerImage

Name                 Version          Source           Summary
----                 -------          ------           -------
NanoServer           10.0.14300.1010  ContainerImag... Container OS Image of Windows Server 2016 Technical...
WindowsServerCore    10.0.14300.1000  ContainerImag... Container OS Image of Windows Server 2016 Technical...
```

Nano Server 기본 OS 이미지를 다운로드하여 설치하려면 다음을 실행합니다. `-version` 매개 변수는 선택 사항이며, 기본 OS 이미지 버전을 지정하지 않으면 최신 버전이 설치됩니다.

```none
Install-ContainerImage -Name NanoServer -Version 10.0.14300.1010
```

마찬가지로 이 명령은 Windows Server Core 기본 OS 이미지를 다운로드하여 설치합니다. `-version` 매개 변수는 선택 사항이며, 기본 OS 이미지 버전을 지정하지 않으면 최신 버전이 설치됩니다.

```none
Install-ContainerImage -Name WindowsServerCore -Version 10.0.14300.1000
```

`docker images` 명령을 사용하여 이미지가 설치되었는지 확인합니다. 

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     40356b90dc80        2 weeks ago         793.3 MB
windowsservercore   10.0.14304.1000     7837d9445187        2 weeks ago         9.176 GB
```  

설치되면 '최신' 태그를 사용하여 이미지에 태그를 지정할 수도 있습니다. 이러한 지침은 아래의 태그 섹션에서 자세히 설명합니다.

> 기본 OS 이미지가 다운로드되었지만 `docker images`를 실행할 때 표시되지 않으면 서비스 제어판 애플릿을 사용하거나, 'sc stop docker' 명령을 사용한 다음 'sc start docker' 명령을 사용하여 Docker 서비스를 다시 시작합니다.

### 이미지 태그 지정

컨테이너 이미지를 이름으로 참조할 경우 Docker 엔진은 이미지의 최신 버전을 검색합니다. 최신 버전을 확인할 수 없는 경우 다음과 같은 오류가 발생합니다.

```none
docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Windows Server Core 또는 Nano Server 기본 OS 이미지를 설치한 후 이러한 이미지에 'latest' 버전이라는 태그가 지정되어야 합니다. 이렇게 하려면 `docker tag` 명령을 사용합니다. 

`docker tag`에 대한 자세한 내용은 [docker.com의 Tag, push, and pull you images(이미지 태그 지정, 푸시 및 끌어오기)](https://docs.docker.com/mac/step_six/)를 참조하세요. 

```none
docker tag <image id> windowsservercore:latest
```

태그를 지정하면 `docker images`의 출력에 두 가지 버전의 동일한 이미지가 표시되는데, 하나는 이미지 버전의 태그가 지정되고 다른 하나는 ‘최신’ 태그가 지정되어 있습니다. 이제 이름으로 이미지를 참조할 수 있습니다.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14300.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### 오프라인 설치

인터넷 연결 없이 기본 OS 이미지를 설치할 수도 있습니다. 이렇게 하려면 인터넷에 연결된 컴퓨터에 이미지를 다운로드하고 대상 시스템에 복사한 다음 `Install-ContainerOSImages` 명령을 사용하여 이미지를 가져옵니다.

기본 OS 이미지를 다운로드하기 전에 다음 명령을 실행하여 컨테이너 이미지 공급자로 **인터넷에 연결된** 시스템을 준비합니다.

```none
Install-PackageProvider ContainerImage -Force
```

PowerShell OneGet 패키지 관리자로부터 이미지 목록을 반환합니다.

```none
Find-ContainerImage
```

출력:

```none
Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.14300.1010         Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.14300.1000         Container OS Image of Windows Server 2016 Techn...
```

이미지를 다운로드하려면 `Save-ContainerImage` 명령을 사용합니다.

```none
Save-ContainerImage -Name NanoServer -Path c:\container-image
```

이제 다운로드한 컨테이너 이미지를 **오프라인 컨테이너 호스트**에 복사하고 `Install-ContainerOSImage` 명령을 사용하여 설치할 수 있습니다.

```none
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### OS 이미지 제거

기본 OS 이미지는 `Uninstall-ContainerOSImage` 명령을 사용하여 제거할 수 있습니다. 다음 예제에서는 NanoServer 기본 OS 이미지를 제거합니다.

```none
Uninstall-ContainerOSImage -FullName CN=Microsoft_NanoServer_10.0.14304.1003
```

## 컨테이너 이미지

### 이미지 나열

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.14300.1000     6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.14300.1010     8572198a60f1        2 weeks ago          0 B
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

> "nano-"로 시작하는 이미지는 Nano Server 기본 OS 이미지에 대한 종속성이 있습니다.

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

Docker 허브에서 이미지를 다운로드하려면 `docker pull`을 사용합니다.

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



<!--HONumber=May16_HO4-->


