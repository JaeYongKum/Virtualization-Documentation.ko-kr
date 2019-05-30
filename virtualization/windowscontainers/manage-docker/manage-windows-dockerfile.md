---
title: Dockerfile 및 Windows 컨테이너
description: Windows 컨테이너용 Dockerfile을 만듭니다.
keywords: Docker, 컨테이너
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: c08fa4d0a89bddeddd0f0a918345c33a6e2ab893
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680993"
---
# <a name="dockerfile-on-windows"></a>Windows의 Dockerfile

Docker 엔진에는 컨테이너 이미지 생성을 자동화 하는 도구가 포함 됩니다. `docker commit` 명령을 실행 하 여 수동으로 컨테이너 이미지를 만들 수 있지만 자동화 된 이미지 만들기 프로세스를 도입 하면 다음과 같은 다양 한 이점이 있습니다.

- 컨테이너 이미지를 코드로 저장할 수 있습니다.
- 유지 관리 및 업그레이드를 위해 컨테이너 이미지를 신속하고 정확하게 다시 만들 수 있습니다.
- 컨테이너 이미지와 개발 주기 간의 연속 통합을 제공합니다.

이 자동화를 수행하는 Docker 구성 요소는 Dockerfile과 `docker build` 명령입니다.

Dockerfile은 새 컨테이너 이미지를 만드는 데 필요한 지침이 포함 된 텍스트 파일입니다. 이러한 명령에는 기반으로 사용할 기존 이미지의 ID, 이미지 만들기 프로세스 중 실행할 명령 및 컨테이너 이미지의 새 인스턴스가 배포될 때 실행될 명령이 포함됩니다.

Docker 빌드는 Dockerfile을 사용 하 고 이미지 생성 프로세스를 트리거하는 Docker 엔진 명령입니다.

이 항목에서는 Windows 컨테이너에서 Dockerfiles를 사용 하 고, 기본 구문을 이해 하 고, 가장 일반적인 Dockerfiles 명령에 대 한 설명을 보여 줍니다.

이 문서에서는 컨테이너 이미지와 컨테이너 이미지 레이어의 개념에 대해 설명 합니다. 이미지 및 이미지 계층화에 대해 자세히 알아보려면 [이미지에](../quick-start/quick-start-images.md)대 한 빠른 시작 가이드를 참조 하세요.

Dockerfiles를 살펴보려면 [Dockerfiles 참조](https://docs.docker.com/engine/reference/builder/)를 참조 하세요.

## <a name="basic-syntax"></a>기본 구문

가장 기본적인 형태의 Dockerfile은 매우 간단할 수 있습니다. 다음 예제에는 IIS 및 'hello world' 사이트를 포함하는 새 이미지를 만듭니다. 이 예제에는 각 단계를 설명하는 주석(`#`으로 표시됨)이 포함되어 있습니다. 이 문서의 다음 섹션에서는 Dockerfile 구문 규칙 및 Dockerfile 명령에 대해 더 자세히 설명합니다.

>[!NOTE]
>Dockerfile은 확장명 없이 만들어야 합니다. Windows에서이 작업을 수행 하려면 원하는 편집기를 사용 하 여 파일을 만든 다음 "Dockerfile" 표기법 (따옴표 포함)을 사용 하 여 저장 합니다.

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Windows 용 Dockerfiles에 대 한 추가 예제는 [windows 리포지토리의 Dockerfiles](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)를 참조 하세요.

## <a name="instructions"></a>지침

Dockerfile 명령어는 컨테이너 이미지를 만드는 데 필요한 지침을 Docker 엔진에 제공 합니다. 이러한 지침은 하나씩 순서 대로 수행 됩니다. 다음 예제는 Dockerfiles에서 가장 일반적으로 사용 되는 명령입니다. Dockerfile 명령에 대 한 전체 목록은 [dockerfile 참조](https://docs.docker.com/engine/reference/builder/)를 참조 하세요.

### <a name="from"></a>FROM

`FROM` 명령은 새 이미지 만들기 프로세스 중에 사용될 컨테이너 이미지를 설정합니다. 예를 들어 `FROM microsoft/windowsservercore` 명령을 사용하면 결과 이미지가 Windows Server Core 기본 OS 이미지에서 파생되며 이 이미지에 대해 종속성이 있습니다. 지정된 이미지가 Docker 빌드 프로세스가 실행되는 시스템에 없는 경우 Docker 엔진은 공용 또는 개인 이미지 레지스트리에서 이미지를 다운로드하려고 합니다.

FROM 명령의 형식은 다음과 같습니다.

```dockerfile
FROM <image>
```

다음은 FROM 명령의 예입니다.

MCR (Microsoft 컨테이너 레지스트리)에서 ltsc2019 버전의 windows server core를 다운로드 하려면 다음을 수행 합니다.
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

자세한 내용은 [FROM 참조](https://docs.docker.com/engine/reference/builder/#from)를 참조 하세요.

### <a name="run"></a>RUN

`RUN` 명령은 실행되고 새 컨테이너 이미지로 캡처될 명령을 지정합니다. 이러한 명령에는 소프트웨어 설치, 파일 및 디렉터리 만들기, 환경 구성 만들기와 같은 항목이 포함될 수 있습니다.

실행 명령은 다음과 같습니다.

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec와 shell 형식 간의 차이는 `RUN` 명령이 실행 되는 방식에 있습니다. exec 형식을 사용하면 지정된 프로그램이 명시적으로 실행됩니다.

다음은 exec 폼의 예입니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

결과 이미지는 다음 `powershell New-Item c:/test` 명령을 실행 합니다.

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

반대로 다음 예제는 shell 형식에서 동일한 작업을 실행 합니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

결과 이미지에는의 `cmd /S /C powershell New-Item c:\test`실행 명령이 있습니다.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Windows에서 실행을 사용할 때의 고려 사항

Windows에서 exec 형식으로 `RUN` 명령을 사용하는 경우 백슬래시를 이스케이프해야 합니다.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

대상 프로그램이 Windows installer 인 경우 실제 (자동) 설치 절차를 시작 하기 전에 플래그를 `/x:<directory>` 통해 설치 프로그램을 추출 해야 합니다. 또한 다른 작업을 수행 하기 전에 명령이 종료 될 때까지 기다려야 합니다. 그렇지 않으면 아무런 작업도 설치 하지 않고 프로세스가 일찍 종료 됩니다. 자세한 내용은 아래 예제를 참조하세요.

#### <a name="examples-of-using-run-with-windows"></a>Windows에서 실행 하는 예제

다음 예제 Dockerfile은 DISM을 사용 하 여 컨테이너 이미지에 IIS를 설치 합니다.

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

다음 예제에서는 Visual Studio의 재배포 가능 패키지를 설치합니다. `Start-Process` `-Wait` 매개 변수는 설치 관리자를 실행 하는 데 사용 됩니다. 이렇게 하면 Dockerfile에서 다음 명령으로 이동 하기 전에 설치가 완료 됩니다.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

실행 명령에 대 한 자세한 내용은 [실행 참조](https://docs.docker.com/engine/reference/builder/#run)를 참조 하세요.

### <a name="copy"></a>복사

`COPY` 명령은 파일 및 디렉터리를 컨테이너의 파일 시스템에 복사 합니다. 파일 및 디렉터리는 Dockerfile에 상대적인 경로에 있어야 합니다.

`COPY` 명령의 형식은 다음과 같습니다.

```dockerfile
COPY <source> <destination>
```

원본 또는 대상에 공백이 포함 된 경우 다음 예제와 같이 대괄호 및 큰따옴표에 경로를 넣습니다.

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Windows에서 복사를 사용할 때 고려해 야 할 사항

Windows에서는 대상 형식이 슬래시를 사용해야 합니다. 예를 들어 다음 명령을 사용할 `COPY` 수 있습니다.

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

또한 백슬래시가 포함 된 다음 형식은 작동 하지 않습니다.

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Windows에서 복사를 사용 하는 예제

다음 예에서는 원본 디렉터리의 내용을 컨테이너 이미지에 명명 `sqllite` 된 디렉터리에 추가 합니다.

```dockerfile
COPY source /sqlite/
```

다음 예제에서는 구성으로 시작 하는 모든 파일을 컨테이너 이미지 `c:\temp` 의 디렉터리로 추가 합니다.

```dockerfile
COPY config* c:/temp/
```

`COPY` 명령에 대 한 자세한 내용은 [참조 복사](https://docs.docker.com/engine/reference/builder/#copy)를 참조 하세요.

### <a name="add"></a>ADD

추가 명령은 복사 명령과 비슷하지만 더욱 많은 기능을 제공 합니다. `ADD` 명령은 호스트에서 컨테이너 이미지로 파일을 복사할 뿐만 아니라 URL이 지정된 원격 위치에서 파일을 복사할 수도 있습니다.

`ADD` 명령의 형식은 다음과 같습니다.

```dockerfile
ADD <source> <destination>
```

원본 또는 대상에 공백이 포함 된 경우 대괄호 및 큰따옴표에 경로를 묶습니다.

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Windows에서 추가를 실행할 때의 고려 사항

Windows에서는 대상 형식이 슬래시를 사용해야 합니다. 예를 들어 다음 명령을 사용할 `ADD` 수 있습니다.

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

또한 백슬래시가 포함 된 다음 형식은 작동 하지 않습니다.

```dockerfile
ADD test1.txt c:\temp\
```

또한 Linux에서 `ADD` 명령은 복사할 때 압축된 패키지를 확장합니다. Windows에서는 이 기능을 사용할 수 없습니다.

#### <a name="examples-of-using-add-with-windows"></a>Windows에서 추가를 사용 하는 예제

다음 예에서는 원본 디렉터리의 내용을 컨테이너 이미지에 명명 `sqllite` 된 디렉터리에 추가 합니다.

```dockerfile
ADD source /sqlite/
```

다음 예제에서는 "config"로 시작 하는 모든 파일을 컨테이너 이미지 `c:\temp` 의 디렉터리에 추가 합니다.

```dockerfile
ADD config* c:/temp/
```

다음 예제에서는 Windows 용 Python을 컨테이너 이미지의 `c:\temp` 디렉터리로 다운로드 합니다.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

`ADD` 명령에 대 한 자세한 내용은 [참조 추가](https://docs.docker.com/engine/reference/builder/#add)를 참조 하세요.

### <a name="workdir"></a>WORKDIR

`WORKDIR` 명령은 `RUN`, `CMD` 등 다른 Dockerfile 명령에 대한 작업 디렉터리를 설정하고 컨테이너 이미지의 실행 중인 인스턴스에 대한 작업 디렉터리도 설정합니다.

`WORKDIR` 명령의 형식은 다음과 같습니다.

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Windows에서 사용할 때의 작업 디렉터리에 대 한 고려 사항

Windows에서 작업 디렉터리에 백슬래시가 포함된 경우 이스케이프해야 합니다.

```dockerfile
WORKDIR c:\\windows
```

**예**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

`WORKDIR` 지침에 대 한 자세한 내용은 [근무 dir 참조](https://docs.docker.com/engine/reference/builder/#workdir)를 참조 하세요.

### <a name="cmd"></a>CMD

`CMD` 명령은 컨테이너 이미지의 인스턴스를 배포할 때 실행할 기본 명령을 설정합니다. 예를 들어 컨테이너를 NGINX 웹 서버 호스팅 하는 경우 웹 서버를 `CMD` 시작 하는 지침과 같은 `nginx.exe`명령을 포함할 수 있습니다. Dockerfile에서 `CMD` 명령을 여러 개 지정하는 경우 마지막 명령만 평가됩니다.

`CMD` 명령의 형식은 다음과 같습니다.

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Windows에서 CMD를 사용할 때의 고려 사항

Windows에서는 `CMD` 명령에 지정된 파일 경로에서 슬래시나 이스케이프된 백슬래시 `\\`를 사용해야 합니다. 유효한 `CMD` 지침은 다음과 같습니다.

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

그러나 적절 한 슬래시가 없는 다음 형식은 작동 하지 않습니다.

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

`CMD` 명령에 대 한 자세한 내용은 [CMD 참조](https://docs.docker.com/engine/reference/builder/#cmd)를 참고 하세요.

## <a name="escape-character"></a>이스케이프 문자

대부분의 경우에는 Dockerfile 명령이 여러 줄에 걸쳐 있어야 합니다. 이렇게 하려면 이스케이프 문자를 사용할 수 있습니다. 기본 Dockerfile 이스케이프 문자는 백슬래시 `\`입니다. 그러나 백슬래시가 Windows의 파일 경로 구분 기호 이므로이를 사용 하 여 여러 줄에 걸쳐 문제가 발생할 수 있습니다. 이 문제를 해결 하기 위해 파서 지시문을 사용 하 여 기본 이스케이프 문자를 변경할 수 있습니다. 파서 지시문에 대 한 자세한 내용은 [parser 지시문](https://docs.docker.com/engine/reference/builder/#parser-directives)을 참조 하세요.

다음 예제에서는 기본 이스케이프 문자를 사용 하 여 여러 줄에 걸쳐 있는 단일 실행 명령을 보여 줍니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

이스케이프 문자를 수정하려면 Dockerfile의 첫 번째 줄에 이스케이프 파서 지시문을 놓습니다. 이는 다음 예제에 표시 될 수 있습니다.

>[!NOTE]
>두 개의 값만 이스케이프 문자: `\` 및 `` ` ``으로 사용할 수 있습니다.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Escape 파서 지시문에 대 한 자세한 내용은 [이스케이프 파서 지시문](https://docs.docker.com/engine/reference/builder/#escape)을 참조 하세요.

## <a name="powershell-in-dockerfile"></a>Dockerfile의 PowerShell

### <a name="powershell-cmdlets"></a>PowerShell cmdlet

PowerShell cmdlet은 `RUN` 작업을 통해 Dockerfile에서 실행할 수 있습니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST 통화

PowerShell `Invoke-WebRequest` cmdlet은 웹 서비스에서 정보나 파일을 수집할 때 유용할 수 있습니다. 예를 들어, Python을 포함 하는 이미지를 작성 하는 경우 `$ProgressPreference` 다음 `SilentlyContinue` 예제와 같이 빠른 다운로드를 수행 하도록 설정할 수 있습니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` Nano 서버 에서도 작동 합니다.

이미지를 만드는 프로세스 중 PowerShell을 사용하여 파일을 다운로드하는 또 다른 옵션은 .NET WebClient 라이브러리를 사용하는 것입니다. 이렇게 하면 다운로드 성능이 향상될 수 있습니다. 다음 예제에서는 WebClient 라이브러리를 사용하여 Python 소프트웨어를 다운로드합니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano 서버는 현재 WebClient를 지원 하지 않습니다.

### <a name="powershell-scripts"></a>PowerShell 스크립트

경우에 따라 이미지 만들기 프로세스 중 사용 하는 컨테이너에 스크립트를 복사한 다음 컨테이너 내에서 스크립트를 실행 하는 것이 유용할 수 있습니다.

>[!NOTE]
>이렇게 하면 모든 이미지 계층 캐싱이 제한 되 고 Dockerfile의 가독성이 낮아집니다.

다음 예제에서는 `ADD` 명령을 사용하여 빌드 컴퓨터에서 컨테이너로 스크립트를 복사합니다. 이 스크립트는 RUN 명령을 사용하여 실행됩니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker 빌드

Dockerfile을 만들어 디스크에 저장 한 후에는를 실행 `docker build` 하 여 새 이미지를 만들 수 있습니다. `docker build` 명령은 여러 가지 옵션 매개 변수와 Dockerfile의 경로를 사용합니다. 모든 빌드 옵션 목록을 비롯 한 Docker 빌드에 대 한 자세한 내용은 [빌드 참조](https://docs.docker.com/engine/reference/commandline/build/#build)를 참조 하세요.

`docker build` 명령의 형식은 다음과 같습니다.

```dockerfile
docker build [OPTIONS] PATH
```

예를 들어 다음 명령을 수행 하면 "iis" 라는 이미지가 만들어집니다.

```dockerfile
docker build -t iis .
```

빌드 프로세스가 시작 되 면 출력에 상태가 표시 되 고 발생 한 오류를 반환 합니다.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

결과는 새 컨테이너 이미지 이며,이 예제에서는 "iis" 라는 이름이 지정 됩니다.

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>추가 읽기 및 참조

- [Windows 용 Dockerfiles 및 Docker 빌드 최적화](optimize-windows-dockerfile.md)
- [Dockerfile 참조](https://docs.docker.com/engine/reference/builder/)
