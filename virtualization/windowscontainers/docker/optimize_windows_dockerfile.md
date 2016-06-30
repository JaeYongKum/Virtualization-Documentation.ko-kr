---
title: "Windows Dockerfile 최적화"
description: "Windows 컨테이너용 Dockerfile을 최적화합니다."
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
translationtype: Human Translation
ms.sourcegitcommit: cc216f56acd5e547d05a48beea57450ba5fcb28b
ms.openlocfilehash: 4822ff2f0248b2d7752299ea55b08e3499e2e2f7

---
# Windows Dockerfile 최적화

**이 예비 콘텐츠는 변경될 수 있습니다.** 

여러 가지 방법을 사용하여 Docker 빌드 프로세스와 결과 Docker 이미지를 모두 최적화할 수 있습니다. 이 문서에서는 Docker 빌드 프로세스의 작동 방식에 대해 자세히 설명하고 Windows 컨테이너에서 최적의 이미지 만들기에 사용할 수 있는 여러 가지 방법을 보여 줍니다.

## Docker 빌드

### 이미지 계층

Docker 빌드 최적화를 검토하기 전에 Docker 빌드의 작동 방식을 이해해야 합니다. Docker 빌드 프로세스 중에는 Dockerfile이 사용되며, 실행 가능한 각 명령이 고유한 임시 컨테이너에서 하나씩 실행됩니다. 결과로 실행 가능한 각 명령에 대한 새로운 이미지 계층이 생성됩니다. 

다음 Dockerfile을 살펴보세요. 이 예제에서는 `windowsservercore` 기본 OS 이미지를 사용하고 IIS를 설치한 다음 간단한 웹 사이트를 만듭니다.

```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

이 Dockerfile에서는 결과 이미지가 컨테이너 OS 이미지와 IIS 및 웹 사이트를 포함하는 두 번째 이미지에 대해 하나씩 두 개의 계층으로 구성된다고 예상하지만 실제로는 그렇지 않습니다. 새 이미지는 많은 계층으로 생성되며, 각 계층은 이전 계층에 종속됩니다. 이를 시각화하려면 새 이미지에 대해 `docker history` 명령을 실행할 수 있습니다. 이렇게 하면 이미지가 네 개의 계층 즉, 기본에 대한 계층 하나와 Dockerfile의 각 명령에 대해 하나씩 세 개의 추가 계층으로 구성됩니다.

```none
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

이러한 각 계층은 Dockerfile의 명령에 매핑할 수 있습니다. 맨 아래 계층(이 예제의 `6801d964fda5`)은 기본 OS 이미지를 나타냅니다. 한 계층 위에서는 IIS 설치를 확인할 수 있습니다. 다음 계층에는 새 웹 사이트가 포함되는 등입니다.

Dockerfile은 이미지 계층을 최소화하고, 빌드 성능을 최적화하며, 가독성과 같은 표면적인 문제를 최적화하도록 작성할 수 있습니다. 결국 동일한 이미지 빌드 작업을 여러 가지 방법으로 완료할 수 있습니다. Dockerfile의 형식이 빌드 시간 및 결과 이미지에 어떻게 영향을 미치는지 이해하면 자동화 환경이 개선됩니다. 

## 이미지 크기 최적화

Docker 컨테이너 이미지를 작성할 때 이미지 크기가 중요한 요인이 될 수 있습니다. 컨테이너 이미지를 레지스트리와 호스트 간에 이동하고 내보내고 가져오며 궁극적으로 공간을 사용합니다. Docker 빌드 프로세스 중에 여러 가지 방법을 사용하여 이미지 크기를 최소화할 수 있습니다. 이 섹션에서는 Windows 컨테이너와 관련된 몇 가지 방법에 대해 자세히 설명합니다. 

Dockerfile 모범 사례에 대한 자세한 내용은 [Docker.com의 Best practices for writing Dockerfiles(Dockerfile 작성에 대한 모범 사례)]( https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)를 참조하세요.

### 그룹 관련 작업

각 `RUN` 명령은 컨테이너 이미지에 새 계층을 만들므로 작업을 하나의 `RUN` 명령으로 그룹화하면 계층 수를 줄일 수 있습니다. 계층을 최소화해도 이미지 크기에 많은 영향을 주지 않지만 그룹화 관련 작업은 영향을 줄 수 있으며, 이 점은 다음 예제에서 확인할 수 있습니다.

다음 두 예제에서는 동일한 기능의 컨테이너 이미지를 만드는 동일한 작업을 보여 주지만 두 Dockerfile은 다르게 생성되었습니다. 결과 이미지도 비교합니다.  

첫 번째 예제에서는 Visual Studio의 재배포 가능 패키지를 다운로드, 추출 및 정리합니다. 이러한 각 작업은 고유한 `RUN` 명령으로 실행됩니다.

```none
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

결과 이미지는 각 `RUN` 명령에 대해 하나씩 세 개의 추가 계층으로 구성됩니다.

```none
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

다음은 동일한 작업이지만 비교하기 위해 모든 단계가 동일한 `RUN` 명령을 사용하여 실행됩니다. `RUN` 명령의 각 단계는 Dockerfile의 새 줄에 있으므로 '\' 문자를 사용하여 줄 바꿈합니다. 

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

여기서 결과 이미지는 `RUN` 명령에 대한 하나의 추가 계층으로 구성됩니다.

```none
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB                
```

### 초과 파일 제거

설치 관리자 같은 파일이 사용된 후 필요하지 않으면 파일을 제거하여 이미지 크기를 줄입니다. 이 작업은 이미지 계층에 파일을 복사하는 것과 동일한 단계에서 수행되어야 합니다. 이렇게 하면 파일이 하위 수준의 이미지 계층에서 유지되지 않도록 할 수 있습니다.

이 예제에서는 Python 패키지를 다운로드하고 실행한 다음 실행 파일을 제거합니다. 모든 작업이 하나의 `RUN` 작업에서 완료되고 단일 이미지 계층이 생성됩니다.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## 빌드 속도 최적화

### 여러 줄

Docker 빌드 속도를 최적화하면 작업을 여러 개의 개별 명령으로 구분하는 것이 유용할 수 있습니다. `RUN` 작업이 여러 개 있으면 캐싱 유효성이 증가합니다. 각 `RUN` 명령에 대해 개별 계층이 만들어지므로 동일한 단계가 다른 Docker 빌드 작업에서 이미 실행된 경우 이 캐시된 작업(이미지 계층)이 다시 사용됩니다. 결과적으로 Docker 빌드 런타임이 줄어듭니다.

다음 예제에서는 Apache와 Visual Studio 재배포 가능 패키지를 모두 다운로드하고 설치한 다음 필요 없는 파일을 정리합니다. 이 모든 작업은 하나의 `RUN` 명령으로 수행됩니다. 이러한 작업이 업데이트되면 모든 작업이 다시 실행됩니다.

```none
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredistexe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force
```

결과 이미지는 기본 OS 이미지와 단일 `RUN` 명령의 모든 작업을 포함하는 두 번째 이미지에 대해 하나씩 두 개의 계층으로 구성됩니다.

```none
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

반대로 다음은 세 개의 `RUN` 명령으로 분할된 동일한 작업입니다. 이 경우 각 `RUN` 명령은 컨테이너 이미지 계층에서 캐시되며, 변경된 명령만 후속 Dockerfile 빌드에서 다시 실행해야 합니다.

```none
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

결과 이미지는 기본 OS 이미지와 각 `RUN` 명령에 대해 하나씩 네 개의 계층으로 구성됩니다. 각 `RUN` 명령이 고유한 계층에서 실행되었기 때문에 이 Dockerfile의 모든 후속 실행 또는 다른 Dockerfile의 동일한 명령 집합에서는 캐시된 이미지 계층을 사용하여 빌드 시간을 줄입니다. 이미지 캐시로 작업할 때는 명령 순서가 중요합니다. 자세한 내용은 이 문서의 다음 섹션을 참조하세요.

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

### 명령 순서 지정

Dockerfile은 위에서 아래로 처리되며, 각 명령은 캐시된 계층과 비교됩니다. 캐시된 계층이 없는 명령이 있으면 이 명령과 모든 후속 명령은 새 컨테이너 이미지 계층에서 처리됩니다. 따라서 명령이 배치되는 순서가 중요합니다. 일정하게 유지될 명령은 Dockerfile의 위쪽에 배치합니다. 변경될 수 있는 명령은 Dockerfile의 아래쪽에 배치합니다. 이렇게 하면 기존 캐시를 부정할 가능성이 줄어듭니다.

이 예제는 Dockerfile의 명령 순서가 캐싱 유효성에 어떻게 영향을 줄 수 있는지를 보여 주기 위한 것입니다. 이 간단한 Dockerfile에서는 번호가 매겨진 4개의 폴더를 만듭니다.  

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```
결과 이미지에는 기본 OS 이미지와 각 `RUN` 명령에 대해 하나씩 5개의 계층이 있습니다.

```none
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B    
```

Docker 파일이 약간 수정되었습니다. 세 번째 `RUN` 명령이 변경되었습니다. 이 Dockerfile에 대해 Docker 빌드가 실행되면 마지막 예제의 명령과 동일한 처음 세 명령은 캐시된 이미지 계층을 사용합니다. 그러나 변경된 `RUN` 명령이 캐시되지 않았기 때문에 새 계층이 해당 명령 자체와 모든 후속 명령에 대해 만들어집니다.

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

새 이미지의 이미지 ID를 마지막 예제와 비교하면 처음 세 계층(아래에서 위로)은 공유되지만 네 번째와 다섯 번째는 고유함을 확인할 수 있습니다.

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## 표면적 문제 최적화

### 명령 대/소문자

Dockerfile 명령은 대/소문자를 구분하지 않지만 규칙은 대문자를 사용하는 것입니다. 그러면 명령 호출과 명령 작업을 구별하여 가독성이 향상됩니다. 다음 두 예제에서는 이 개념을 보여 줍니다. 

소문자:
```none
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```
대문자: 
```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### 줄 바꿈

길고 복잡한 작업은 백슬래시 `\` 문자를 사용하여 여러 줄로 구분할 수 있습니다. 다음 Dockerfile에서는 Visual Studio 재배포 가능 패키지를 설치하고 설치 관리자 파일을 제거한 다음 구성 파일을 만듭니다. 이러한 세 작업을 모두 한 줄에 지정합니다.

```none
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```
`RUN` 명령 하나의 각 작업을 고유한 줄에 지정하도록 명령을 다시 작성할 수 있습니다. 

```none
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## 추가 참고 자료 및 참조

[Windows의 Dockerfile](./manage_windows_dockerfile.md)

[Docker.com의 Best practices for writing Dockerfile(Dockerfile 작성에 대한 모범 사례)](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Jun16_HO4-->


