---
title: Dockerfile 및 Windows 컨테이너
description: Windows 컨테이너용 Dockerfile을 만듭니다.
keywords: Docker, 컨테이너
author: PatrickLang
ms.date: 05/03/2019
ms.topic: tutorial
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 6c708eec39eba971bd33077dec2ec10c3f964454
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87984717"
---
# <a name="dockerfile-on-windows"></a>Windows의 Dockerfile

Docker 엔진에는 컨테이너 이미지 만들기를 자동화하는 도구가 포함되어 있습니다. `docker commit` 명령을 사용하여 수동으로 컨테이너 이미지를 만들 수 있지만, 자동화된 이미지 만들기 프로세스를 채택하면 다음을 비롯한 많은 혜택이 있습니다.

- 컨테이너 이미지를 코드로 저장할 수 있습니다.
- 유지 관리 및 업그레이드를 위해 컨테이너 이미지를 신속하고 정확하게 다시 만들 수 있습니다.
- 컨테이너 이미지와 개발 주기 간의 연속 통합을 제공합니다.

이 자동화를 수행하는 Docker 구성 요소는 Dockerfile과 `docker build` 명령입니다.

Dockerfile은 새 컨테이너 이미지를 만드는 데 필요한 명령을 포함하고 있는 텍스트 파일입니다. 이러한 명령에는 기반으로 사용할 기존 이미지의 ID, 이미지 만들기 프로세스 중 실행할 명령 및 컨테이너 이미지의 새 인스턴스가 배포될 때 실행될 명령이 포함됩니다.

Docker 빌드는 Dockerfile을 사용하고 이미지 만들기 프로세스를 트리거하는 Docker 엔진 명령입니다.

이 토픽에서는 Windows 컨테이너에서 Dockerfile을 사용하는 방법을 보여주고, 기본 구문 및 가장 많이 사용되는 Dockerfile 명령을 알아봅니다.

이 문서에서는 컨테이너 이미지 및 컨테이너 이미지 레이어의 개념에 대해 설명합니다. 이미지 및 이미지 계층화에 대해 자세히 알아보려면 [컨테이너 기본 이미지](../manage-containers/container-base-images.md)를 참조하세요.

Dockerfile을 전체적으로 살펴보려면 [Dockerfile 참조](https://docs.docker.com/engine/reference/builder/)를 확인하세요.

## <a name="basic-syntax"></a>기본 구문

가장 기본적인 형태의 Dockerfile은 매우 간단할 수 있습니다. 다음 예제에는 IIS 및 'hello world' 사이트를 포함하는 새 이미지를 만듭니다. 이 예제에는 각 단계를 설명하는 주석(`#`으로 표시됨)이 포함되어 있습니다. 이 문서의 다음 섹션에서는 Dockerfile 구문 규칙 및 Dockerfile 명령에 대해 더 자세히 설명합니다.

>[!NOTE]
>Dockerfile은 확장명 없이 만들어야 합니다. Windows에서 이렇게 하려면 원하는 편집기로 파일을 만들고, 따옴표를 포함하여 "Dockerfile"로 저장하면 됩니다.

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

Windows용 Dockerfile에 대한 추가 예제는 [Windows용 Dockerfile 리포지토리](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)를 참조하세요.

## <a name="instructions"></a>지침

Dockerfile 명령은 컨테이너 이미지를 만드는 데 필요한 명령을 Docker 엔진에 제공합니다. 이러한 명령은 순서대로 하나씩 수행됩니다. 다음 예제는 Dockerfile에서 가장 많이 사용되는 명령입니다. Dockerfile 전체 명령 목록은 [Dockerfile 참조](https://docs.docker.com/engine/reference/builder/)에서 확인할 수 있습니다.

### <a name="from"></a>FROM

`FROM` 명령은 새 이미지 만들기 프로세스 중에 사용될 컨테이너 이미지를 설정합니다. 예를 들어 `FROM mcr.microsoft.com/windows/servercore` 명령을 사용하면 결과 이미지가 Windows Server Core 기본 OS 이미지에서 파생되며 이 이미지에 대해 종속성이 있습니다. 지정된 이미지가 Docker 빌드 프로세스가 실행되는 시스템에 없는 경우 Docker 엔진은 공용 또는 프라이빗 이미지 레지스트리에서 이미지를 다운로드하려고 합니다.

FROM 명령의 형식은 다음과 같습니다.

```dockerfile
FROM <image>
```

다음은 FROM 명령의 예입니다.

MCR(Microsoft Container Registry)에서 ltsc2019 버전 Windows Server Core를 다운로드하려면 다음을 수행합니다.
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

자세한 내용은 [FROM 참조](https://docs.docker.com/engine/reference/builder/#from)에서 확인할 수 있습니다.

### <a name="run"></a>RUN

`RUN` 명령은 실행되고 새 컨테이너 이미지로 캡처될 명령을 지정합니다. 이러한 명령에는 소프트웨어 설치, 파일 및 디렉터리 만들기, 환경 구성 만들기와 같은 항목이 포함될 수 있습니다.

RUN 명령은 다음과 같습니다.

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

exec 및 shell 형식 간의 차이는 `RUN` 명령이 실행되는 방법에 있습니다. exec 형식을 사용하면 지정된 프로그램이 명시적으로 실행됩니다.

다음은 exec 형식의 예입니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

결과 이미지는 다음과 같이 `powershell New-Item c:/test` 명령을 실행합니다.

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

반면, 다음 예제는 동일한 작업을 shell 형식으로 실행합니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

결과 이미지에는 `cmd /S /C powershell New-Item c:\test`의 실행 명령이 있습니다.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Windows에서 RUN 명령을 사용할 때 고려 사항

Windows에서 exec 형식으로 `RUN` 명령을 사용하는 경우 백슬래시를 이스케이프해야 합니다.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

대상 프로그램이 Windows 설치 프로그램인 경우 `/x:<directory>` 플래그를 통해 설치 프로그램을 추출해야만 실제(자동) 설치 절차를 시작할 수 있습니다. 또한 다른 작업을 수행하려면 명령이 종료될 때까지 기다려야 합니다. 그렇지 않으면 아무것도 설치되지 않고 프로세스가 종료됩니다. 자세한 내용은 아래 예제를 참조하세요.

#### <a name="examples-of-using-run-with-windows"></a>Windows에서 RUN 사용 예

다음 예제 Dockerfile은 DISM을 사용하여 컨테이너 이미지에 IIS를 설치합니다.

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

다음 예제에서는 Visual Studio의 재배포 가능 패키지를 설치합니다. 설치 관리자를 실행할 때 `Start-Process` 및 `-Wait` 매개 변수를 사용합니다. 이렇게 하면 설치가 완료된 후 Dockerfile의 다음 단계로 이동됩니다.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

RUN 명령에 대한 자세한 내용은 [RUN 참조](https://docs.docker.com/engine/reference/builder/#run)에서 확인할 수 있습니다.

### <a name="copy"></a>복사

`COPY` 명령은 파일 및 디렉터리를 컨테이너의 파일 시스템에 복사합니다. 파일 및 디렉터리는 Dockerfile의 상대 경로에 있어야 합니다.

`COPY` 명령의 형식은 다음과 같습니다.

```dockerfile
COPY <source> <destination>
```

원본 또는 대상에 공백이 포함된 경우 다음 예제처럼 경로를 대괄호와 큰따옴표를 묶습니다.

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Windows에서 COPY 명령을 사용할 때 고려 사항

Windows에서는 대상 형식이 슬래시를 사용해야 합니다. 예를 들어 다음은 유효한 `COPY` 명령입니다.

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

반면, 백슬래시를 사용하는 다음 형식은 작동하지 않습니다.

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Windows에서 COPY 사용 예

다음 예제에서는 원본 디렉터리의 내용을 컨테이너 이미지의 `sqllite` 디렉터리에 추가합니다.

```dockerfile
COPY source /sqlite/
```

다음 예제에서는 config로 시작하는 모든 파일을 컨테이너 이미지의 `c:\temp` 디렉터리에 추가합니다.

```dockerfile
COPY config* c:/temp/
```

`COPY` 명령에 대한 자세한 내용은 [COPY 참조](https://docs.docker.com/engine/reference/builder/#copy)에서 확인할 수 있습니다.

### <a name="add"></a>추가

ADD 명령은 COPY 명령과 비슷하지만 훨씬 많은 기능이 있습니다. `ADD` 명령은 호스트에서 컨테이너 이미지로 파일을 복사할 뿐만 아니라 URL이 지정된 원격 위치에서 파일을 복사할 수도 있습니다.

`ADD` 명령의 형식은 다음과 같습니다.

```dockerfile
ADD <source> <destination>
```

원본 또는 대상에 공백이 포함된 경우 다음과 같이 경로를 대괄호와 큰따옴표를 묶습니다.

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Windows에서 ADD 명령을 사용할 때 고려 사항

Windows에서는 대상 형식이 슬래시를 사용해야 합니다. 예를 들어 다음은 유효한 `ADD` 명령입니다.

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

반면, 백슬래시를 사용하는 다음 형식은 작동하지 않습니다.

```dockerfile
ADD test1.txt c:\temp\
```

또한 Linux에서 `ADD` 명령은 복사할 때 압축된 패키지를 확장합니다. Windows에서는 이 기능을 사용할 수 없습니다.

#### <a name="examples-of-using-add-with-windows"></a>Windows에서 ADD 사용 예

다음 예제에서는 원본 디렉터리의 내용을 컨테이너 이미지의 `sqllite` 디렉터리에 추가합니다.

```dockerfile
ADD source /sqlite/
```

다음 예제에서는 "config"로 시작하는 모든 파일을 컨테이너 이미지의 `c:\temp` 디렉터리에 추가합니다.

```dockerfile
ADD config* c:/temp/
```

다음 예제에서는 Windows용 Python을 컨테이너 이미지의 `c:\temp` 디렉터리에 다운로드합니다.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

`ADD` 명령에 대한 자세한 내용은 [ADD 참조](https://docs.docker.com/engine/reference/builder/#add)에서 확인할 수 있습니다.

### <a name="workdir"></a>WORKDIR

`WORKDIR` 명령은 `RUN`, `CMD` 등 다른 Dockerfile 명령에 대한 작업 디렉터리를 설정하고 컨테이너 이미지의 실행 중인 인스턴스에 대한 작업 디렉터리도 설정합니다.

`WORKDIR` 명령의 형식은 다음과 같습니다.

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Windows에서 WORKDIR 명령을 사용할 때 고려 사항

Windows에서 작업 디렉터리에 백슬래시가 포함된 경우 이스케이프해야 합니다.

```dockerfile
WORKDIR c:\\windows
```

**예제**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

`WORKDIR` 명령에 대한 자세한 내용은 [WORKDIR 참조](https://docs.docker.com/engine/reference/builder/#workdir)에서 확인할 수 있습니다.

### <a name="cmd"></a>CMD

`CMD` 명령은 컨테이너 이미지의 인스턴스를 배포할 때 실행할 기본 명령을 설정합니다. 예를 들어 컨테이너에서 NGINX 웹 서버를 호스팅하는 경우 `CMD`에는 `nginx.exe` 같은 명령을 사용하여 웹 서버를 시작하는 명령이 포함될 수 있습니다. Dockerfile에서 `CMD` 명령을 여러 개 지정하는 경우 마지막 명령만 평가됩니다.

`CMD` 명령의 형식은 다음과 같습니다.

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Windows에서 CMD 명령을 사용할 때 고려 사항

Windows에서는 `CMD` 명령에 지정된 파일 경로에서 슬래시나 이스케이프된 백슬래시 `\\`를 사용해야 합니다. 다음은 유효한 `CMD` 명령입니다.

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

그러나 적절한 슬래시가 없는 다음 형식은 작동하지 않습니다.

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

`CMD` 명령에 대한 자세한 내용은 [CMD 참조](https://docs.docker.com/engine/reference/builder/#cmd)에서 확인할 수 있습니다.

## <a name="escape-character"></a>이스케이프 문자

Dockerfile 명령이 여러 줄에 걸쳐 계속되어야 하는 경우가 많습니다. 이 경우 이스케이프 문자를 사용하면 됩니다. 기본 Dockerfile 이스케이프 문자는 백슬래시 `\`입니다. 그러나 백슬래시는 Windows에서도 파일 경로 구분 기호이므로, 백슬래시를 사용하여 여러 줄로 확장할 때 문제가 발생할 수 있습니다. 이 문제를 피하려면 기본 이스케이프 문자를 변경하는 파서 지시문을 사용하면 됩니다. 파서 지시문에 대한 자세한 내용은 [파서 지시문](https://docs.docker.com/engine/reference/builder/#parser-directives)을 참조하세요.

다음 예제에서는 기본 이스케이프 문자를 사용하여 여러 줄에서 계속되는 단일 RUN 명령을 보여줍니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

이스케이프 문자를 수정하려면 Dockerfile의 첫 번째 줄에 이스케이프 파서 지시문을 놓습니다. 이 내용은 다음 예제에서 확인할 수 있습니다.

>[!NOTE]
>`\` 및 `` ` `` 값만 이스케이프 문자로 사용할 수 있습니다.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

이스케이프 파서 지시문에 대한 자세한 내용은 [이스케이프 파서 지시문](https://docs.docker.com/engine/reference/builder/#escape)을 참조하세요.

## <a name="powershell-in-dockerfile"></a>Dockerfile의 PowerShell

### <a name="powershell-cmdlets"></a>PowerShell cmdlet

PowerShell cmdlet은 Dockerfile에서 `RUN` 작업을 사용하여 실행할 수 있습니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST 호출

PowerShell의 `Invoke-WebRequest` cmdlet은 웹 서비스에서 정보나 파일을 수집할 때 유용하게 사용할 수 있습니다. 예를 들어 Python을 포함하는 이미지를 빌드할 때 다음 예제처럼 `$ProgressPreference`를 `SilentlyContinue`로 설정하여 다운로드 속도를 높일 수 있습니다.

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
>`Invoke-WebRequest`는 Nano Server에서도 작동합니다.

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
>Nano Server는 현재 WebClient를 지원하지 않습니다.

### <a name="powershell-scripts"></a>PowerShell 스크립트

경우에 따라 스크립트를 이미지 만들기 프로세스 중에 사용하는 컨테이너에 복사한 다음, 컨테이너 내에서 스크립트를 실행하는 것이 좋을 수 있습니다.

>[!NOTE]
>이렇게 하면 이미지 레이어 캐싱이 제한되고 Dockerfile의 가독성이 저하됩니다.

다음 예제에서는 `ADD` 명령을 사용하여 빌드 컴퓨터에서 컨테이너로 스크립트를 복사합니다. 이 스크립트는 RUN 명령을 사용하여 실행됩니다.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker 빌드

Dockerfile을 만들고 디스크에 저장한 후에는 `docker build`를 실행하여 새 이미지를 만들 수 있습니다. `docker build` 명령은 여러 가지 옵션 매개 변수와 Dockerfile의 경로를 사용합니다. 모든 빌드 옵션 목록을 포함하여 Docker Build에 대한 전체 설명서는 [빌드 참조](https://docs.docker.com/engine/reference/commandline/build/#build)에서 확인할 수 있습니다.

`docker build` 명령의 형식은 다음과 같습니다.

```dockerfile
docker build [OPTIONS] PATH
```

예를 들어 다음 명령은 'iis'라는 이미지를 만듭니다.

```dockerfile
docker build -t iis .
```

빌드 프로세스가 시작되면 출력은 상태를 나타내고 throw된 오류를 반환합니다.

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

그 결과 새 컨테이너 이미지가 생성되는데, 이 예제에서는 "iis"라는 이미지입니다.

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>추가 참고 자료

- [Windows용 Dockerfile 및 Docker 빌드 최적화](optimize-windows-dockerfile.md)
- [Dockerfile 참조](https://docs.docker.com/engine/reference/builder/)
