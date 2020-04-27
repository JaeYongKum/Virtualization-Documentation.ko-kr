---
title: Windows Dockerfile 최적화
description: Windows 컨테이너용 Dockerfile을 최적화합니다.
keywords: Docker, 컨테이너
author: cwilhit
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: ae633c7ba5d9672335addcc582988fc47c13ed79
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/24/2020
ms.locfileid: "74910153"
---
# <a name="optimize-windows-dockerfiles"></a>Windows Dockerfile 최적화

Docker 빌드 프로세스와 그 결과로 얻는 Docker 이미지를 모두 최적화하는 여러 가지 방법이 있습니다. 이 문서에서는 Docker 빌드 프로세스의 작동 방식과 최적의 Windows 컨테이너용 이미지를 만드는 방법을 설명합니다.

## <a name="image-layers-in-docker-build"></a>Docker 빌드의 이미지 레이어

Docker 빌드를 최적화하려면 먼저 Docker 빌드의 작동 방식을 알아야 합니다. Docker 빌드 프로세스 중에는 Dockerfile이 사용되며, 실행 가능한 각 명령이 고유한 임시 컨테이너에서 하나씩 실행됩니다. 결과로 실행 가능한 각 명령에 대한 새로운 이미지 계층이 생성됩니다.

예를 들어 다음 샘플 Dockerfile은 `mcr.microsoft.com/windows/servercore:ltsc2019` 기본 OS 이미지를 사용하여 IIS를 설치한 다음, 간단한 웹 사이트를 만듭니다.

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

이 Dockerfile이 생성하는 이미지에 두 개의 레이어, 즉, 컨테이너 OS 이미지용 레이어와 IIS 및 웹 사이트를 포함하는 레이어가 포함될 것으로 예상하시는 분들도 있을 것입니다. 그러나 실제 이미지에는 여러 레이어가 있으며, 각 레이어는 이전 계층에 종속됩니다.

이 점을 보다 명확하게 이해하기 위해, 샘플 Dockerfile에서 생성한 이미지에 대해 `docker history` 명령을 실행하겠습니다.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

출력을 보면 이 이미지에는 기본 레이어 하나와 Dockerfile의 각 명령에 매핑되는 세 개의 추가 레이어를 더해서 총 4개 레이어가 있습니다. 맨 아래 계층(이 예제의 `6801d964fda5`)은 기본 OS 이미지를 나타냅니다. 하나 위에 있는 레이어에는 IIS가 설치됩니다 다음 계층에는 새 웹 사이트가 포함되는 등입니다.

Dockerfile은 이미지 레이어를 최소화하고, 빌드 성능을 최적화하고, 가독성을 통해 접근성을 최적화하도록 작성할 수 있습니다. 결국 동일한 이미지 빌드 작업을 여러 가지 방법으로 완료할 수 있습니다. Dockerfile의 형식이 빌드 시간 및 이미지에 미치는 영향을 이해하면 자동화 환경이 개선됩니다.

## <a name="optimize-image-size"></a>이미지 크기 최적화

공간 요구 사항에 따라 Docker 컨테이너 이미지를 빌드할 때 이미지 크기가 중요한 요인이 될 수 있습니다. 컨테이너 이미지를 레지스트리와 호스트 간에 이동하고 내보내고 가져오며 궁극적으로 공간을 사용합니다. 이 섹션에서는 Windows 컨테이너의 Docker 빌드 프로세스 중에 이미지 크기를 최소화하는 방법을 설명합니다.

Dockerfile 모범 사례에 대한 자세한 내용은 [Docker.com의 Best practices for writing Dockerfiles(Dockerfile 작성에 대한 모범 사례)](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)를 참조하세요.

### <a name="group-related-actions"></a>그룹 관련 작업

각 `RUN` 명령은 컨테이너 이미지에 새 레이어를 생성하므로 작업을 하나의 `RUN` 명령으로 그룹화하면 Dockerfile의 레이어 수를 줄일 수 있습니다. 계층을 최소화해도 이미지 크기에 많은 영향을 주지 않지만 그룹화 관련 작업은 영향을 줄 수 있으며, 이 점은 다음 예제에서 확인할 수 있습니다.

이 섹션에서는 동일한 작업을 수행하는 두 개의 예제 Dockerfiles을 비교합니다. 그러나 한 Dockerfile은 작업당 하나의 명령만 있고, 다른 Dockerfile은 관련 작업이 함께 그룹화되어 있습니다.

그룹화되지 않은 다음 예제 Dockerfile은 Windows용 Python을 다운로드하여 설치하고, 설치가 완료되면 다운로드한 설치 파일을 제거합니다. 이 Dockerfile의 각 작업에는 고유한 `RUN` 명령이 제공됩니다.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

결과 이미지는 각 `RUN` 명령에 대해 하나씩 세 개의 추가 계층으로 구성됩니다.

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

두 번째 예제는 정확히 동일한 작업을 수행하는 Dockerfile입니다. 그러나 모든 관련 작업이 단일 `RUN` 명령으로 그룹화되었습니다. `RUN` 명령의 각 단계는 Dockerfile의 새 줄에 있고, '\\' 문자는 줄 바꿈에 사용됩니다.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

결과 이미지에는 `RUN` 명령에 대한 추가 레이어 하나만 있습니다.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>초과 파일 제거

설치 관리자처럼 사용 후에는 더 이상 필요 없는 파일이 Dockerfile에 있는 경우 해당 파일을 제거하여 이미지 크기를 줄일 수 있습니다. 이 작업은 이미지 계층에 파일을 복사하는 것과 동일한 단계에서 수행되어야 합니다. 이렇게 하면 파일이 하위 수준의 이미지 레이어에 유지되지 않도록 할 수 있습니다.

다음 Dockerfile 예제에서는 Python 패키지를 다운로드하고, 실행하고, 제거합니다. 모든 작업이 하나의 `RUN` 작업에서 완료되고 단일 이미지 계층이 생성됩니다.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>빌드 속도 최적화

### <a name="multiple-lines"></a>여러 줄

작업을 여러 개별 명령으로 분할하여 Docker 빌드 속도를 최적화할 수 있습니다. 각 `RUN` 명령에 대한 개별 레이어가 만들어지므로 여러 `RUN` 작업을 수행하면 캐싱 효과가 증가합니다. 동일한 명령이 다른 Docker 빌드 작업에서 이미 실행된 경우 이 캐싱된 작업(이미지 레이어)이 다시 사용되므로 Docker 빌드 런타임이 감소합니다.

다음 예제에서는 Apache와 Visual Studio 재배포 가능 패키지를 다운로드하여 설치한 다음, 더 이상 필요 없는 파일을 제거하여 정리합니다. 이 모든 작업이 단일 `RUN` 명령을 통해 수행됩니다. 이러한 작업이 업데이트되면 모든 작업이 다시 실행됩니다.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \

  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \

  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

결과 이미지에는 기본 OS 이미지의 레이어와 단일 `RUN` 명령의 모든 작업을 포함하는 레이어가 있습니다.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

반면 여기서는 같은 작업이 3개의 `RUN` 명령으로 분할되었습니다. 이 경우 각 `RUN` 명령은 컨테이너 이미지 레이어에서 캐싱되며, 변경된 명령만 후속 Dockerfile 빌드에서 다시 실행해야 합니다.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

결과 이미지는 기본 OS 이미지의 레이어 하나와 3개의 각 `RUN` 명령에 대한 레이어 하나씩, 총 4개의 레이어로 구성됩니다. 각 `RUN` 명령이 고유한 레이어에서 실행되었기 때문에 이 Dockerfile의 모든 후속 실행 또는 다른 Dockerfile의 동일한 명령 세트에서는 캐시된 이미지 레이어를 사용하여 빌드 시간을 줄입니다.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

다음 섹션에서 볼 수 있듯이, 이미지 캐시를 사용할 때 명령 순서가 중요합니다.

### <a name="ordering-instructions"></a>명령 순서 지정

Dockerfile은 위에서 아래로 처리되며, 각 명령은 캐시된 계층과 비교됩니다. 캐시된 계층이 없는 명령이 있으면 이 명령과 모든 후속 명령은 새 컨테이너 이미지 계층에서 처리됩니다. 따라서 명령이 배치되는 순서가 중요합니다. 일정하게 유지될 명령은 Dockerfile의 위쪽에 배치합니다. 변경될 수 있는 명령은 Dockerfile의 아래쪽에 배치합니다. 이렇게 하면 기존 캐시를 부정할 가능성이 줄어듭니다.

다음 예제는 Dockerfile 명령 순서가 캐싱 효율성에 미치는 영향을 보여줍니다. 이 간단한 Dockerfile에는 번호가 매겨진 4개 폴더가 있습니다.  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

결과 이미지에는 기본 OS 이미지의 레이어 하나와 `RUN` 명령마다 하나씩, 총 5개의 레이어가 있습니다.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

이제 다음 Dockerfile이 약간 수정되었으며, 세 번째 `RUN` 명령이 새 파일로 변경되었습니다. 이 Dockerfile에 대해 Docker 빌드가 실행되면 마지막 예제의 명령과 동일한 처음 세 명령은 캐시된 이미지 계층을 사용합니다. 그러나 변경된 `RUN` 명령이 캐싱되지 않았기 때문에 변경된 명령과 모든 후속 명령에 대한 새 레이어가 만들어집니다.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

새 이미지의 이미지 ID를 이 섹션의 첫 번째 예제와 비교해 보면, 아래쪽에 있는 처음 3개 레이어는 공유되지만 네 번째 및 다섯 번째 레이어는 고유하다는 것을 알 수 있습니다.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>표면적 문제 최적화

### <a name="instruction-case"></a>명령 대/소문자

Dockerfile 명령은 대/소문자를 구분하지 않지만 규칙은 대문자를 사용합니다. 이렇게 하면 명령 호출과 명령 연산을 구별하여 가독성이 향상됩니다. 다음 두 예제에서는 대문자로 표시되는 Dockerfile과 그렇지 않은 Dockerfile을 비교합니다.

다음은 대문자로 표시되지 않는 Dockerfile입니다.

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

다음은 대문자를 사용하는 동일한 Dockerfile입니다.

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>줄 바꿈

길고 복잡한 작업은 백슬래시 `\` 문자를 사용하여 여러 줄로 구분할 수 있습니다. 다음 Dockerfile에서는 Visual Studio 재배포 가능 패키지를 설치하고 설치 관리자 파일을 제거한 다음 구성 파일을 만듭니다. 이러한 세 작업을 모두 한 줄에 지정합니다.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

명령을 백슬래시로 분할하면 한 `RUN` 명령의 각 작업을 각각 고유한 줄에 지정할 수 있습니다.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>추가 참고 자료

[Windows의 Dockerfile](manage-windows-dockerfile.md)

[Docker.com의 Best practices for writing Dockerfile(Dockerfile 작성에 대한 모범 사례)](https://docs.docker.com/engine/reference/builder/)
