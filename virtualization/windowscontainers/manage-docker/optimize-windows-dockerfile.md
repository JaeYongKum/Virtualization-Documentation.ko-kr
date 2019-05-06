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
ms.openlocfilehash: d897560061fae23fda6f88ebdad6dd804da9a8f1
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610343"
---
# <a name="optimize-windows-dockerfiles"></a>Windows Dockerfile 최적화

여러 가지 방법으로 Docker 빌드 프로세스와 결과 Docker 이미지를 모두 최적화할 수 있습니다. 이 문서에서는 Docker 빌드 프로세스의 작동 방식 및 최적으로 Windows 컨테이너에 대 한 이미지를 만드는 방법을 설명 합니다.

## <a name="image-layers-in-docker-build"></a>Docker 빌드에 이미지 계층

Docker 빌드 최적화 하려면, 먼저 Docker 빌드의 작동 방식을 확인 해야 합니다. Docker 빌드 프로세스 중에는 Dockerfile이 사용되며, 실행 가능한 각 명령이 고유한 임시 컨테이너에서 하나씩 실행됩니다. 결과로 실행 가능한 각 명령에 대한 새로운 이미지 계층이 생성됩니다.

예를 들어 다음 샘플에서는 Dockerfile은 `windowsservercore` 기본 OS 이미지를 IIS를 설치 하 고 다음 간단한 웹 사이트를 만듭니다.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

이 Dockerfile는 컨테이너 OS 이미지와 IIS 및 웹 사이트를 포함 하는 두 번째 이미지에 대해 하나씩 두 개의 계층을 사용 하 여 이미지를 생성을 예상할 수 있습니다. 그러나 실제 이미지에는 많은 계층으로 하며, 각 계층은 이전에 따라 달라 집니다.

명확 하 게 만드는, 실행 하겠습니다 합니다 `docker history` 이미지에 대해 만든 Dockerfile 샘플 명령 합니다.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

출력을 보여 줍니다이 이미지는 네 개의 계층: 계층 및 Dockerfile의 각 명령에 매핑되는 세 개의 추가 계층입니다. 맨 아래 계층(이 예제의 `6801d964fda5`)은 기본 OS 이미지를 나타냅니다. 한 계층은 IIS 설치 됩니다. 다음 계층에는 새 웹 사이트가 포함되는 등입니다.

Dockerfile은 이미지 계층을 최소화, 빌드 성능을 최적화 하며 가독성을 통해 접근성 최적화를 작성할 수 있습니다. 결국 동일한 이미지 빌드 작업을 여러 가지 방법으로 완료할 수 있습니다. Dockerfile의 형식이 빌드 시간에 영향을 미칩니다 및 자동화 환경을 개선 하는 이미지를 만드는 방법을 이해 합니다.

## <a name="optimize-image-size"></a>이미지 크기 최적화

공간 요구 사항에 따라 이미지 크기 Docker 컨테이너 이미지를 작성할 때 중요 한 요인이 될 수 있습니다. 컨테이너 이미지를 레지스트리와 호스트 간에 이동하고 내보내고 가져오며 궁극적으로 공간을 사용합니다. 이 섹션에는 Windows 컨테이너에 대 한 Docker 빌드 프로세스 중 이미지 크기를 최소화 하는 방법을 알려줍니다.

Dockerfile 모범 사례에 대 한 자세한 내용은 [Docker.com의 Dockerfile 작성에 대 한 모범 사례](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)를 참조 하세요.

### <a name="group-related-actions"></a>그룹 관련 작업

때문에 각 `RUN` 명령은 컨테이너 이미지를 가지 그룹화 새 레이어를 만듭니다 `RUN` 명령 계층은 Dockerfile의 수를 줄일 수 있습니다. 계층을 최소화해도 이미지 크기에 많은 영향을 주지 않지만 그룹화 관련 작업은 영향을 줄 수 있으며, 이 점은 다음 예제에서 확인할 수 있습니다.

이 섹션에서는 두 예제에서는 동일한 작업을 수행 하는 Dockerfile을 비교 합니다. 그러나 한 Dockerfile는 그룹화 관련된 작업에는 다른 작업을 당 하나의 명령만.

다음 그룹화 되지 않은 예제에서는 Dockerfile Windows 용 Python을 다운로드 하 고 설치, 설치 작업이 완료 되 면 다운로드 한 설치 파일을 제거 합니다. 이 dockerfile에서는 각 작업은 지정 자체 `RUN` 명령입니다.

```dockerfile
FROM windowsservercore

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

두 번째 예제는 정확히 동일한 작업을 수행 하는 Dockerfile. 그러나 모든 관련 작업은 단일에서 그룹화 된 `RUN` 명령입니다. 각 단계는 `RUN` Dockerfile의 새 줄에 동안는 ' \\' 문자 줄 바꿈 데 사용 됩니다.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

결과 이미지에 대 한 하나의 추가 계층에는 `RUN` 명령입니다.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>초과 파일 제거

파일 후 필요 없는 설치 관리자 같은 프로그램 Dockerfile에는, 이미지 크기를 줄이는 것을 제거할 수 없는 경우. 이 작업은 이미지 계층에 파일을 복사하는 것과 동일한 단계에서 수행되어야 합니다. 이렇게 파일이 하위 수준의 이미지 계층에서 유지 되지 않도록 할 수 있습니다.

다음 예제에서는 Dockerfile, Python 패키지는 다운로드를 실행 한 다음 제거 합니다. 모든 작업이 하나의 `RUN` 작업에서 완료되고 단일 이미지 계층이 생성됩니다.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>빌드 속도 최적화

### <a name="multiple-lines"></a>여러 줄

Docker 빌드 속도 최적화 하기 위해 여러 개의 개별 명령으로 작업을 나눌 수 있습니다. 여러 `RUN` 작업 각각에 대해 개별 계층이 만들어지므로 있으면 캐싱 유효성이 증가 `RUN` 명령 합니다. 동일한 명령 다른 Docker 빌드 작업에서 이미 실행 되었을 경우이 캐시 된 작업 (이미지 계층) 다시 사용 감소 Docker 빌드 런타임이 발생 합니다.

다음 예제에서는 Apache와 Visual Studio 재배포 가능 패키지를 모두 다운로드, 설치 및 더 이상 필요 없는 파일을 제거 하 여 다음을 정리 합니다. 이 작업을 모두 완료 된 하나의 `RUN` 명령입니다. 이러한 작업이 업데이트 되 면 모든 작업을 다시 실행 합니다.

```dockerfile
FROM windowsservercore

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

결과 이미지에는 두 개의 계층, 기본 OS 이미지 및 모든 작업을 단일 포함 된 `RUN` 명령입니다.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

3 개으로 동일한 동작 분할 비교 하 여, 다음은 `RUN` 지침. 이 경우 각 `RUN` 명령은 컨테이너 이미지 계층에서 캐시 및 명령만 후속 Dockerfile 다시 실행 하 여 변경 된 필요가 빌드에서 합니다.

```dockerfile
FROM windowsservercore

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

결과 이미지는 네 개의 계층; 구성 기본 OS 이미지 및 세 계층 `RUN` 지침. 때문에 각 `RUN` 명령이 고유한 계층에서 실행,이 Dockerfile의 모든 후속 실행 또는 다른 dockerfile에서 명령의 동일한 집합 빌드 시간을 단축 하 고 캐시 된 이미지 계층으로 사용 됩니다.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

지침을 요청 하는 방법 이기도 이미지 캐시로 작업할 때 다음 섹션에서 확인할 수 있습니다.

### <a name="ordering-instructions"></a>명령 순서 지정

Dockerfile은 위에서 아래로 처리되며, 각 명령은 캐시된 계층과 비교됩니다. 캐시된 계층이 없는 명령이 있으면 이 명령과 모든 후속 명령은 새 컨테이너 이미지 계층에서 처리됩니다. 따라서 명령이 배치되는 순서가 중요합니다. 일정하게 유지될 명령은 Dockerfile의 위쪽에 배치합니다. 변경될 수 있는 명령은 Dockerfile의 아래쪽에 배치합니다. 이렇게 하면 기존 캐시를 부정할 가능성이 줄어듭니다.

다음 예제에서는 Dockerfile 명령 순서가 영향 캐싱 유효성 보여 줍니다. 이 간단한 예제 Dockerfile 번호가 매겨진된 4 개의 폴더에 있습니다.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

결과 이미지와 각 기본 OS 이미지에 대해 하나씩 5 개의 계층이는 `RUN` 지침.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

다음 Dockerfile이 약간 수정 되었습니다, 세 번째를 사용 하 여 `RUN` 명령이 새 파일을 변경 합니다. 이 Dockerfile에 대해 Docker 빌드가 실행되면 마지막 예제의 명령과 동일한 처음 세 명령은 캐시된 이미지 계층을 사용합니다. 그러나 때문에 변경 된 `RUN` 명령이 캐시 되지, 변경 된 명령과 모든 후속 명령에 대 한 새 계층에 대해 만들어집니다.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

이 섹션의 첫 번째 예제는 새 이미지의 이미지 Id를 비교할 경우 맨 아래에서 처음 세 계층은 공유 되지만 네 번째와 다섯 번째는 고유한 알 수 있습니다.

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

Dockerfile 명령은 대/소문자를 구분 하지 않지만 규칙은 대문자를 사용 하는 것입니다. 그러면 명령 호출과 명령 작업을 구별 하 여 가독성이 향상 됩니다. 다음 두 예제는 시작 및 애플리케이션의 Dockerfile을 비교합니다.

다음은 시작된 된 Dockerfile입니다.

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

다음은 대문자를 사용 하 여 동일한 Dockerfile입니다.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>줄 바꿈

길고 복잡 한 작업은 백슬래시도 여러 줄으로 구분할 수 있습니다 `\` 문자입니다. 다음 Dockerfile에서는 Visual Studio 재배포 가능 패키지를 설치하고 설치 관리자 파일을 제거한 다음 구성 파일을 만듭니다. 이러한 세 작업을 모두 한 줄에 지정합니다.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

명령은 나눌 수 있습니다 백슬래시를 사용 하 여 하므로 각 작업에서에서 `RUN` 명령이 고유한 줄에 지정 됩니다.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>추가 정보 및 참조

[Windows의 Dockerfile](manage-windows-dockerfile.md)

[Docker.com의 Best practices for writing Dockerfile(Dockerfile 작성에 대한 모범 사례)](https://docs.docker.com/engine/reference/builder/)
