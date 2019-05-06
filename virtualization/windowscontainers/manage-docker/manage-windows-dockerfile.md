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
ms.openlocfilehash: 9ff6256ab9708533f72e9b3210f8a5fd32f4048a
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610273"
---
# <a name="dockerfile-on-windows"></a>Windows의 Dockerfile

Docker 엔진은 컨테이너 이미지 생성 자동화 하는 도구가 포함 되어 있습니다. 실행 하 여 컨테이너 이미지를 수동으로 만들 수 있지만 합니다 `docker commit` 명령을 비롯 한 여러 혜택에는 자동화 된 이미지 만들기 프로세스를 채택 합니다.

- 컨테이너 이미지를 코드로 저장할 수 있습니다.
- 유지 관리 및 업그레이드를 위해 컨테이너 이미지를 신속하고 정확하게 다시 만들 수 있습니다.
- 컨테이너 이미지와 개발 주기 간의 연속 통합을 제공합니다.

이 자동화를 수행하는 Docker 구성 요소는 Dockerfile과 `docker build` 명령입니다.

Dockerfile 새 컨테이너 이미지를 만드는 데 필요한 명령을 포함 하는 텍스트 파일입니다. 이러한 명령에는 기반으로 사용할 기존 이미지의 ID, 이미지 만들기 프로세스 중 실행할 명령 및 컨테이너 이미지의 새 인스턴스가 배포될 때 실행될 명령이 포함됩니다.

Docker 빌드 Dockerfile을 사용 하 고 이미지 만들기 프로세스를 트리거하는 Docker 엔진 명령입니다.

이 항목에서는 Dockerfile을 사용 하 여 Windows 컨테이너를 사용 하 여, 기본 구문을 이해 하는 방법과 가장 일반적인 Dockerfile 명령 이란 보여줍니다.

이 문서에서는 컨테이너 이미지와 컨테이너 이미지 계층의 개념을 설명 합니다. 이미지와 이미지 계층화에 대 한 자세한 정보를 원하는 경우 [이미지 빠른 시작 가이드를](../quick-start/quick-start-images.md)참조 하세요.

Dockerfile에 대 한 전체 [Dockerfile 참조](https://docs.docker.com/engine/reference/builder/)를 참조 하세요.

## <a name="basic-syntax"></a>기본 구문

가장 기본적인 형태의 Dockerfile은 매우 간단할 수 있습니다. 다음 예제에는 IIS 및 'hello world' 사이트를 포함하는 새 이미지를 만듭니다. 이 예제에는 각 단계를 설명하는 주석(`#`으로 표시됨)이 포함되어 있습니다. 이 문서의 다음 섹션에서는 Dockerfile 구문 규칙 및 Dockerfile 명령에 대해 더 자세히 설명합니다.

>[!NOTE]
>Dockerfile은 확장명 없이 만들어야 합니다. Windows에서 이렇게 하려면 원하는, 편집기를 사용 하 여 파일을 만든 후 표기법 "Dockerfile" (따옴표 포함)를 사용 하 여 저장 합니다.

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Windows 용 Dockerfile의 추가 예제를 [Windows 용 Dockerfile 리포지토리](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)를 참조 하세요.

## <a name="instructions"></a>지침

Dockerfile 명령은 Docker 엔진은 컨테이너 이미지를 만드는 데 필요한 지침을 제공 합니다. 이러한 명령은-하나씩 및 순서 대로 수행 됩니다. 다음 예제는 Dockerfile의 가장 자주 사용 되는 지침입니다. Dockerfile 명령의 전체 목록은, [Dockerfile 참조](https://docs.docker.com/engine/reference/builder/)를 참조 하세요.

### <a name="from"></a>FROM

`FROM` 명령은 새 이미지 만들기 프로세스 중에 사용될 컨테이너 이미지를 설정합니다. 예를 들어 `FROM microsoft/windowsservercore` 명령을 사용하면 결과 이미지가 Windows Server Core 기본 OS 이미지에서 파생되며 이 이미지에 대해 종속성이 있습니다. 지정된 이미지가 Docker 빌드 프로세스가 실행되는 시스템에 없는 경우 Docker 엔진은 공용 또는 개인 이미지 레지스트리에서 이미지를 다운로드하려고 합니다.

FROM 명령 형식을 다음과 같이 진행 됩니다.

```dockerfile
FROM <image>
```

다음은 FROM 명령의 예입니다.

```dockerfile
FROM microsoft/windowsservercore
```

자세한 내용은 [FROM 참조](https://docs.docker.com/engine/reference/builder/#from)를 참조 하세요.

### <a name="run"></a>RUN

`RUN` 명령은 실행되고 새 컨테이너 이미지로 캡처될 명령을 지정합니다. 이러한 명령에는 소프트웨어 설치, 파일 및 디렉터리 만들기, 환경 구성 만들기와 같은 항목이 포함될 수 있습니다.

RUN 명령은 다음과 같이 진행 됩니다.

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec 및 shell 형식 간의 차이 방법에 `RUN` 명령이 실행 됩니다. exec 형식을 사용하면 지정된 프로그램이 명시적으로 실행됩니다.

Exec의 예는 다음과 같습니다.

```dockerfile
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

결과 이미지는 실행 되는 `powershell New-Item c:/test` 명령:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

반대로 다음 예제에서는 셸 형태로 동일한 작업을 실행 합니다.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

결과 이미지는의 run 명령이 생성 `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>실행을 사용 하 여 windows에 대 한 고려 사항

Windows에서 exec 형식으로 `RUN` 명령을 사용하는 경우 백슬래시를 이스케이프해야 합니다.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

통해 설치 프로그램을 추출 해야 대상 프로그램이 Windows installer 이면 합니다 `/x:<directory>` 플래그는 실제 (자동) 설치 절차를 시작할 수 있습니다. 또한 아무 작업도 수행 하기 전에 종료 하려면 명령에 대 한 기다려야 합니다. 그렇지 않으면 아무것도 설치 하지 않고 프로세스 조기에 종료 됩니다. 자세한 내용은 아래 예제를 참조하세요.

#### <a name="examples-of-using-run-with-windows"></a>실행을 사용 하 여 windows의 예

다음 예제에서는 Dockerfile DISM을 사용 하 여 컨테이너 이미지에 IIS를 설치 합니다.

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

다음 예제에서는 Visual Studio의 재배포 가능 패키지를 설치합니다. `Start-Process` 및 `-Wait` 설치 관리자를 실행 하려면 매개 변수 사용 됩니다. 이렇게 하면 Dockerfile에서 다음 명령으로 넘어가기 전에 설치가 완료 되는.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

RUN 명령에 대 한 자세한 내용은 [참조 실행](https://docs.docker.com/engine/reference/builder/#run)을 참조 하세요.

### <a name="copy"></a>복사

`COPY` 명령은 컨테이너의 파일 시스템에 파일 및 디렉터리에 복사 합니다. 파일 및 디렉터리는 Dockerfile의 상대 경로에 있어야 합니다.

`COPY` 명령 형식은 다음과 같이 이루어집니다.

```dockerfile
COPY <source> <destination>
```

원본 또는 대상에 공백을 포함 하는 경우에 다음 예제에 표시 된 대로 경로를 대괄호와 큰따옴표를 묶습니다.

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>복사본을 사용 하 여 windows에 대 한 고려 사항

Windows에서는 대상 형식이 슬래시를 사용해야 합니다. 예를 들어 다음은 유효한 `COPY` 지침:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

한편, 백슬래시를 사용 하 여 다음 형식을 작동 하지 않습니다.

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>복사본을 사용 하 여 windows의 예

다음 예제에서는 원본 디렉터리의 내용을 디렉터리에 추가 `sqllite` 컨테이너 이미지에서:

```dockerfile
COPY source /sqlite/
```

다음 예제에서는 됩니다를 config로 시작 하는 모든 파일을 추가 합니다 `c:\temp` 컨테이너 이미지의 디렉터리:

```dockerfile
COPY config* c:/temp/
```

에 대 한 세부 정보는 `COPY` 명령 [COPY 참조](https://docs.docker.com/engine/reference/builder/#copy)를 참조 하세요.

### <a name="add"></a>ADD

ADD 명령은 COPY 명령과 같은 하지만 훨씬 더 많은 기능입니다. `ADD` 명령은 호스트에서 컨테이너 이미지로 파일을 복사할 뿐만 아니라 URL이 지정된 원격 위치에서 파일을 복사할 수도 있습니다.

`ADD` 명령 형식은 다음과 같이 이루어집니다.

```dockerfile
ADD <source> <destination>
```

원본 또는 대상 공백을 포함 하는 경우 경로 대괄호와 큰따옴표를 묶습니다.

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>추가 Windows와 함께 실행 하기 위한 고려 사항

Windows에서는 대상 형식이 슬래시를 사용해야 합니다. 예를 들어 다음은 유효한 `ADD` 지침:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

한편, 백슬래시를 사용 하 여 다음 형식을 작동 하지 않습니다.

```dockerfile
ADD test1.txt c:\temp\
```

또한 Linux에서 `ADD` 명령은 복사할 때 압축된 패키지를 확장합니다. Windows에서는 이 기능을 사용할 수 없습니다.

#### <a name="examples-of-using-add-with-windows"></a>Windows와 함께 사용 하 여 몇 가지 추가

다음 예제에서는 원본 디렉터리의 내용을 디렉터리에 추가 `sqllite` 컨테이너 이미지에서:

```dockerfile
ADD source /sqlite/
```

다음 예제는 "구성"로 시작 하는 모든 파일을 추가할 합니다 `c:\temp` 컨테이너 이미지의 디렉터리입니다.

```dockerfile
ADD config* c:/temp/
```

다음 예제에서는 Windows 용 Python을 다운로드 하 고 `c:\temp` 컨테이너 이미지의 디렉터리입니다.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

에 대 한 세부 정보는 `ADD` 명령 [참조 추가](https://docs.docker.com/engine/reference/builder/#add)참조 하세요.

### <a name="workdir"></a>WORKDIR

`WORKDIR` 명령은 `RUN`, `CMD` 등 다른 Dockerfile 명령에 대한 작업 디렉터리를 설정하고 컨테이너 이미지의 실행 중인 인스턴스에 대한 작업 디렉터리도 설정합니다.

`WORKDIR` 명령 형식은 다음과 같이 이루어집니다.

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Windows WORKDIR 사용에 대 한 고려 사항

Windows에서 작업 디렉터리에 백슬래시가 포함된 경우 이스케이프해야 합니다.

```dockerfile
WORKDIR c:\\windows
```

**예**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

대 한 자세한 내용은 합니다 `WORKDIR` 명령 [WORKDIR 참조](https://docs.docker.com/engine/reference/builder/#workdir)를 참조 하세요.

### <a name="cmd"></a>CMD

`CMD` 명령은 컨테이너 이미지의 인스턴스를 배포할 때 실행할 기본 명령을 설정합니다. 예를 들어 컨테이너에서 NGINX 웹 서버를 호스트 하는 경우는 `CMD` 와 같은 명령을 사용 하 여 웹 서버를 시작 하는 지침을 포함 될 수 있습니다 `nginx.exe`. Dockerfile에서 `CMD` 명령을 여러 개 지정하는 경우 마지막 명령만 평가됩니다.

`CMD` 명령 형식은 다음과 같이 이루어집니다.

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>CMD를 사용 하 여 windows에 대 한 고려 사항

Windows에서는 `CMD` 명령에 지정된 파일 경로에서 슬래시나 이스케이프된 백슬래시 `\\`를 사용해야 합니다. 다음은 유효한 `CMD` 지침:

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

그러나 다음 형식을 적절 한 슬래시 없이 작동 하지 않습니다.

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

에 대 한 세부 정보는 `CMD` 명령 [CMD 참조](https://docs.docker.com/engine/reference/builder/#cmd)를 참조 하세요.

## <a name="escape-character"></a>이스케이프 문자

대부분의 경우에서 Dockerfile 명령이 여러 줄에 걸쳐 해야 합니다. 이렇게 하려면 이스케이프 문자를 사용할 수 있습니다. 기본 Dockerfile 이스케이프 문자는 백슬래시 `\`입니다. 그러나 백슬래시 Windows의 파일 경로 구분 기호 이기도 하므로 여러 줄에 걸쳐를 사용 하 여 문제가 발생할 수 있습니다. 이 문제를 가져오려면 기본 이스케이프 문자를 변경 하려면 파서 지시문을 사용할 수 있습니다. 파서 지시문에 대 한 자세한 내용은 [파서 지시문](https://docs.docker.com/engine/reference/builder/#parser-directives)을 참조 하세요.

다음 예제에서는 기본 이스케이프 문자를 사용 하 여 여러 줄에서 계속 되는 단일 RUN 명령을 보여 줍니다.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

이스케이프 문자를 수정하려면 Dockerfile의 첫 번째 줄에 이스케이프 파서 지시문을 놓습니다. 다음 예제에서이 볼 수 있습니다.

>[!NOTE]
>이스케이프 문자로 두 개의 값을 사용할 수 있습니다: `\` 및 `` ` ``.

```dockerfile
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

이스케이프 파서 지시문에 대 한 자세한 내용은 [이스케이프 파서 지시문](https://docs.docker.com/engine/reference/builder/#escape)을 참조 하세요.

## <a name="powershell-in-dockerfile"></a>Dockerfile의 PowerShell

### <a name="powershell-cmdlets"></a>PowerShell cmdlet

PowerShell cmdlet를 사용 하 여 Dockerfile에서 실행 될 수는 `RUN` 작업 합니다.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST 호출

PowerShell의 `Invoke-WebRequest` cmdlet 웹 서비스에서 정보나 파일을 수집할 때 유용할 수 있습니다. 예를 들어 Python을 포함 하는 이미지를 빌드하는 경우 설정할 수 있습니다 `$ProgressPreference` 를 `SilentlyContinue` 여 다음 예와 같이 더 빠르게 다운로드 합니다.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` 또한 Nano 서버에서 작동합니다.

이미지를 만드는 프로세스 중 PowerShell을 사용하여 파일을 다운로드하는 또 다른 옵션은 .NET WebClient 라이브러리를 사용하는 것입니다. 이렇게 하면 다운로드 성능이 향상될 수 있습니다. 다음 예제에서는 WebClient 라이브러리를 사용하여 Python 소프트웨어를 다운로드합니다.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano 서버 WebClient 현재 지원 되지 않습니다.

### <a name="powershell-scripts"></a>PowerShell 스크립트

경우에 따라 스크립트를 이미지 만들기 프로세스 중 사용할 컨테이너에 복사한 다음 컨테이너 내에서 스크립트를 실행 하는 것이 유용할 수 있습니다.

>[!NOTE]
>이 작업은 이미지 계층 캐싱이 제한와 Dockerfile의 가독성이 저하 됩니다.

다음 예제에서는 `ADD` 명령을 사용하여 빌드 컴퓨터에서 컨테이너로 스크립트를 복사합니다. 이 스크립트는 RUN 명령을 사용하여 실행됩니다.

```dockerfile
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker 빌드

Dockerfile을 만들고 디스크에 저장을 실행할 수 있습니다 `docker build` 새 이미지를 만듭니다. `docker build` 명령은 여러 가지 옵션 매개 변수와 Dockerfile의 경로를 사용합니다. Docker 빌드 대 한 전체 설명서에 대 한 빌드 옵션 모든 목록을 포함 하 여 [빌드 참조](https://docs.docker.com/engine/reference/commandline/build/#build)를 참조 하십시오.

형식의 `docker build` 다음과 같은 명령을 전송 됩니다.

```dockerfile
docker build [OPTIONS] PATH
```

예를 들어 다음 명령은 "iis." 라는 이미지로 만듭니다.

```dockerfile
docker build -t iis .
```

빌드 프로세스에서 시작 되 면 출력 되며 상태를 나타내는 throw 된 오류를 반환 합니다.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM micrsoft/windowsservercore
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

결과이 예제는 "iis." 라는 새 컨테이너 이미지

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>추가 정보 및 참조

- [Windows 용 Dockerfiles 및 Docker 빌드 최적화](optimize-windows-dockerfile.md)
- [Dockerfile 참조](https://docs.docker.com/engine/reference/builder/)
