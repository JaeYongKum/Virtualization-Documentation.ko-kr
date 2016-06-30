---
title: "Windows에서 Docker 배포"
description: "Windows에서 Docker 배포"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bdfa4545-2291-4827-8165-2d6c98d72d37
redirect_url: ../docker/configure_docker_daemon
translationtype: Human Translation
ms.sourcegitcommit: cb689563fac8479fe9012cf41b183c1d1ef76657
ms.openlocfilehash: 387789ee9169d2affb9b9ed8e4e0e607071ac29b

---

# Docker 및 Windows

**이 예비 콘텐츠는 변경될 수 있습니다.** 

Docker 엔진은 Windows에 포함되어 있지 않으므로 개별적으로 설치하고 구성해야 합니다. Windows에서 Docker 엔진을 실행하는 데 사용되는 단계는 Linux에서 실행하는 데 사용되는 단계와 다릅니다. 이 문서에서는 Windows Server 2016, Nano Server 및 Windows 클라이언트에 Docker 엔진을 설치하고 구성하는 과정을 단계별로 실행합니다. 또한 Docker 엔진 및 명령줄 인터페이스는 최근에 두 파일로 분할되었습니다. 이 문서에서는 둘 다를 설치하는 지침을 제공합니다.

Docker 및 Docker 도구 집합에 대한 자세한 내용은 [Docker.com](https://www.docker.com/)에서 확인하세요. 

> Docker를 사용하여 Windows 컨테이너를 만들고 관리하려면 먼저 Windows 컨테이너 기능을 사용하도록 설정해야 합니다. 이 기능을 사용하도록 설정하기 위한 지침은 [컨테이너 호스트 배포 가이드](./docker_windows.md)를 참조하세요.

## Windows Server 2016

### Docker 디먼 설치 <!--1-->

`https://aka.ms/tp5/dockerd`에서 dockerd.exe를 다운로드하여 컨테이너 호스트의 System32 디렉터리에 배치합니다.

```none
wget https://aka.ms/tp5/dockerd -OutFile $env:SystemRoot\system32\dockerd.exe
```

`c:\programdata\docker`라는 디렉터리를 만듭니다. 이 디렉터리에 `runDockerDaemon.cmd`라는 파일을 만듭니다.

```none
New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

다음 텍스트를 `runDockerDaemon.cmd` 파일에 복사합니다.

```none
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (if exist %ProgramData%\docker\tag.txt (goto :secure))

if not exist %systemroot%\system32\dockerd.exe (goto :legacy)

dockerd -H npipe:// 
goto :eof

:legacy
docker daemon -H npipe:// 
goto :eof

:secure
if not exist %systemroot%\system32\dockerd.exe (goto :legacysecure)
dockerd -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
goto :eof

:legacysecure
docker daemon -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```
[https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip)에서 nssm.exe를 다운로드합니다.

```none
wget https://nssm.cc/release/nssm-2.24.zip -OutFile $env:ALLUSERSPROFILE\nssm.zip
```

압축된 패키지의 압축을 풉니다.

```none
Expand-Archive -Path $env:ALLUSERSPROFILE\nssm.zip $env:ALLUSERSPROFILE
```

`nssm-2.24\win64\nssm.exe`를 `c:\windows\system32` 디렉터리에 복사합니다.

```none
Copy-Item $env:ALLUSERSPROFILE\nssm-2.24\win64\nssm.exe $env:SystemRoot\system32
```
`nssm install`을 실행하여 Docker 서비스를 구성합니다.

```none
start-process nssm install
```

NSSM 서비스 설치 관리자의 해당 필드에 다음 데이터를 입력합니다.

응용 프로그램 탭:

**경로:** C:\Windows\System32\cmd.exe

**시작 디렉터리:** C:\Windows\System32

**인수:** /s /c C:\ProgramData\docker\runDockerDaemon.cmd < nul

**서비스 이름** - Docker

![](media/nssm1.png)

세부 정보 탭:

**표시 이름:** Docker

**설명:** Docker 디먼은 Docker 클라이언트의 컨테이너에 대한 관리 기능을 제공합니다.

![](media/nssm2.png)

IO 탭:

**출력(stdout):** C:\ProgramData\docker\daemon.log

**오류(stderr):** C:\ProgramData\docker\daemon.log

![](media/nssm3.png)

완료되면 `Install Service` 단추를 클릭합니다.

이제 Docker 디먼이 Windows 서비스로 구성됩니다.

### 방화벽 <!--1-->

원격 Docker 관리를 사용하도록 설정하려면 TCP 포트 2376도 열어야 합니다.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

### Docker 제거 <!--1-->

다음 명령은 Docker 서비스를 제거합니다.

```none
sc.exe delete Docker
```

### Docker CLI 설치

`https://aka.ms/tp5/docker`에서 docker.exe를 다운로드하여 컨테이너 호스트 또는 Docker 명령을 실행할 다른 모든 시스템의 System32 디렉터리에 배치합니다.

```none
wget https://aka.ms/tp5/docker -OutFile $env:SystemRoot\system32\docker.exe
```

## Nano Server

### Docker 설치 <!--2-->

`https://aka.ms/tp5/dockerd`에서 dockerd.exe를 다운로드하고 Nano Server 컨테이너 호스트의 `windows\system32` 폴더에 복사합니다.

`c:\programdata\docker`라는 디렉터리를 만듭니다. 이 디렉터리에 `runDockerDaemon.cmd`라는 파일을 만듭니다.

```none
New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

다음 텍스트를 `runDockerDaemon.cmd` 파일에 복사합니다.

```none
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (if exist %ProgramData%\docker\tag.txt (goto :secure))

if not exist %systemroot%\system32\dockerd.exe (goto :legacy)

dockerd -H npipe:// 
goto :eof

:legacy
docker daemon -H npipe:// 
goto :eof

:secure
if not exist %systemroot%\system32\dockerd.exe (goto :legacysecure)
dockerd -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
goto :eof

:legacysecure
docker daemon -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```

다음 스크립트를 사용하여 Windows가 부팅될 때 Docker 디먼을 시작하는 예약된 작업을 만들 수 있습니다.

```none
# Creates a scheduled task to start docker.exe at computer start up.

$dockerData = "$($env:ProgramData)\docker"
$dockerDaemonScript = "$dockerData\runDockerDaemon.cmd"
$dockerLog = "$dockerData\daemon.log"
$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c $dockerDaemonScript > $dockerLog 2>&1" -WorkingDirectory $dockerData
$trigger = New-ScheduledTaskTrigger -AtStartup
$settings = New-ScheduledTaskSettingsSet -Priority 5
Register-ScheduledTask -TaskName Docker -Action $action -Trigger $trigger -Settings $settings -User SYSTEM -RunLevel Highest | Out-Null
Start-ScheduledTask -TaskName Docker 
```

### 방화벽 <!--2-->

원격 Docker 관리를 사용하도록 설정하려면 TCP 포트 2376도 열어야 합니다.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

### 대화형 Nano 세션

Nano Server는 원격 PowerShell 세션을 통해 관리됩니다. Nano Server를 원격으로 관리하는 방법에 대한 자세한 내용은 [Getting Started with Nano Server(Nano Server 시작)]( https://technet.microsoft.com/en-us/library/mt126167.aspx#bkmk_ManageRemote)를 참조하세요.

'docker attach'와 같은 일부 Docker 작업은 이 원격 PowerShell 세션을 통해 수행할 수 있습니다. 일반적인 모범 사례로 이 문제를 해결하려면 보안 TCP 연결을 통해 원격 클라이언트에서 Docker를 관리합니다.

이렇게 하려면 Docker 디먼이 TCP 포트에서 수신 대기하도록 구성되었는지와 Docker 명령줄 인터페이스를 원격 클라이언트 컴퓨터에서 사용할 수 있는지를 확인합니다. 구성되면 -H 매개 변수를 사용하여 호스트에서 Docker 명령을 실행할 수 있습니다. 원격 시스템에서 Docker 디먼에 액세스하는 방법에 대한 자세한 내용은 [Docker.com의 Daemon socket options(디먼 소켓 옵션)](https://docs.docker.com/engine/reference/commandline/daemon/#daemon-socket-option)를 참조하세요.

컨테이너를 원격으로 배포하고 대화형 세션을 시작하려면 다음 명령을 실행합니다.

```none
docker -H tcp://<ipaddress of server>:2376 run -it nanoserver cmd
```

환경 변수 DOCKER_HOST를 만들어 -H 매개 변수 요구 사항을 제거할 수 있습니다. 이 작업에는 다음 PowerShell 명령을 사용할 수 있습니다.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

이 변수를 설정하면 명령이 다음과 같이 표시됩니다.

```none
docker run -it nanoserver cmd
```

### Docker 제거 <!--2-->

Nano Server에서 Docker 디먼 및 cli를 제거하려면 Windows\system32 디렉터리에서 `docker.exe`를 삭제합니다.

```none
Remove-Item $env:SystemRoot\system32\docker.exe
``` 

다음을 실행하여 Docker 예약된 작업을 등록 취소합니다.

```none
Get-ScheduledTask -TaskName Docker | UnRegister-ScheduledTask
```

### Docker CLI 설치

`https://aka.ms/tp5/docker`에서 docker.exe를 다운로드하고 Nano Server 컨테이너 호스트의 windows\system32 폴더에 복사합니다.

```none
wget https://aka.ms/tp5/docker -OutFile $env:SystemRoot\system32\docker.exe
```

## Docker 시작 구성

여러 가지 시작 옵션을 Docker 디먼에 사용할 수 있습니다. 이 섹션에서는 Windows의 Docker 디먼과 관련된 몇 가지 옵션을 자세히 살펴봅니다. 전체 범위의 모든 디먼 옵션은 [docker.com의 Docker daemon documentation(Docker 디먼 설명서)]( https://docs.docker.com/engine/reference/commandline/daemon/)을 참조하세요.

### TCP 포트 수신

Docker 디먼은 들어오는 연결을 명명된 파이프를 통해 로컬로 또는 TCP 연결을 통해 원격으로 수신하도록 구성할 수 있습니다. 기본 시작 동작은 명명된 파이프에서만 수신 대기하여 원격 연결을 방지하는 것입니다.

```none
docker daemon -D
```

이 동작은 다음 시작 명령을 사용하여 들어오는 보안 연결을 수신하도록 수정할 수 있습니다. 연결에 보안을 설정하는 방법에 대한 자세한 내용은 [docker.com의 Security Configuration docs(보안 구성 문서)](https://docs.docker.com/engine/security/https/)를 참조하세요.

```none
docker daemon -D -H npipe:// -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
``` 

### 명명된 파이프 액세스

컨테이너 호스트에서 로컬로 실행되는 Docker 명령은 명명된 파이프를 통해 수신됩니다. 이러한 명령을 실행하려면 관리자 권한이 필요합니다. 또 다른 옵션은 명명된 파이프에 대한 액세스 권한이 있는 그룹을 지정하는 것입니다. 다음 예제에서는 `docker`라는 Windows 그룹에 이 액세스 권한이 제공됩니다.

```none
dockerd -H npipe:// -G docker
```  


### 기본 런타임

Windows 컨테이너에는 Windows Server 및 Hyper-V라는 두 가지 고유한 런타임 형식이 있습니다. Docker 디먼은 기본적으로 Windows Server 런타임을 사용하도록 구성되지만 변경할 수 있습니다. Hyper-V를 기본 런타임으로 설정하려면 Docker 디먼을 초기화할 때 ‘—exec-opt isolation=hyperv`를 지정합니다.

```none
docker daemon -D —exec-opt isolation=hyperv
```




<!--HONumber=Jun16_HO4-->


