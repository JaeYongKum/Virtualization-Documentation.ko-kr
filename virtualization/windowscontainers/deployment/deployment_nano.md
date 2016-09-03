---
title: "Nano Server에 Windows 컨테이너 배포"
description: "Nano Server에 Windows 컨테이너 배포"
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 2319649d1dd39677e59a9431fbefaf82982492c6
ms.openlocfilehash: a79469987879656117812ff6f9563046584172c0

---

# 컨테이너 호스트 배포 - Nano Server

**이 예비 콘텐츠는 변경될 수 있습니다.** 

이 문서에서는 Windows 컨테이너 기능을 사용하여 기본적인 Nano Server를 배포하는 방법에 대해 단계별로 설명합니다. 고급 항목이므로 Windows 및 Windows에 컨테이너에 대한 기본적인 지식이 있다고 가정합니다. Windows 컨테이너에 대한 소개는 [Windows 컨테이너 빠른 시작](../quick_start/quick_start.md)을 참조하세요.

## Nano Server 준비

다음 섹션에서는 기본적인 Nano Server 구성 배포에 대해 자세히 설명합니다. Nano Server 배포 및 구성 옵션에 대한 자세한 내용은 [Getting Started with Nano Server(Nano Server 시작)](https://technet.microsoft.com/en-us/library/mt126167.aspx)를 참조하세요.

### Nano Server VM 만들기

먼저 [이 위치](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)에서 Nano Server 평가판 VHD를 다운로드합니다. 이 VHD에서 가상 컴퓨터를 만들고 가상 컴퓨터를 시작한 다음 Hyper-V 연결 옵션 또는 사용하는 가상화 플랫폼에 따라 해당 옵션을 사용하여 연결합니다.

### 원격 PowerShell 세션 만들기

Nano Server에는 대화형 로그온 기능이 없기 때문에 모든 관리는 PowerShell을 사용하는 원격 시스템에서 완료됩니다.

Nano Server 시스템을 원격 시스템의 신뢰할 수 있는 호스트에 추가합니다. IP 주소를 Nano Server의 IP 주소로 바꿉니다.

```none
Set-Item WSMan:\localhost\Client\TrustedHosts 192.168.1.50 -Force
```

원격 PowerShell 세션을 만듭니다.

```none
Enter-PSSession -ComputerName 192.168.1.50 -Credential ~\Administrator
```

이러한 단계를 완료하면 Nano Server 시스템과 함께 원격 PowerShell 세션을 사용하게 됩니다. 달리 언급되지 않는 한, 이 문서의 나머지 부분은 원격 세션에서 이루어집니다.


## 컨테이너 기능 설치

Nano Server 패키지 관리 공급자는 Nano Server에 역할 및 기능을 설치하도록 허용합니다. 이 명령을 사용하여 공급자를 설치합니다.

```none
Install-PackageProvider NanoServerPackage
```

패키지 공급자를 설치한 후 컨테이너 기능을 설치합니다.

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

컨테이너 기능을 설치한 후에는 Nano Server 호스트를 다시 부팅해야 합니다. 

```none
Restart-Computer
```

백업 후 원격 PowerShell 연결을 다시 설정합니다.

## Docker 설치

Windows 컨테이너를 사용하려면 Docker 엔진이 필요합니다. 다음 단계를 사용하여 Docker 엔진을 설치합니다.

먼저, SMB에 대한 Nano Server 방화벽을 구성했는지 확인합니다. 이는 Nano Server 호스트에서 이 명령을 실행하여 완료할 수 있습니다.

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

Nano Server 호스트에 Docker 실행 파일을 저장할 폴더를 만듭니다.

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Docker 엔진 및 클라이언트를 다운로드하고 컨테이너 호스트의 'C:\Program Files\docker\'에 복사합니다. 

> Nano Server는 현재 `Invoke-WebRequest`를 지원하지 않습니다. 원격 시스템에서 다운로드를 완료해야 하며 파일은 Nano Server 호스트에 복사해야 합니다.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile .\docker-1.12.0.zip -UseBasicParsing
```

다운로드한 패키지를 추출합니다. 작업이 완료되면 **dockerd.exe** 및 **docker.exe**를 모두 포함하는 디렉터리를 갖게 됩니다. 두 특성 모두를 Nano Server 컨테이너 호스트에 있는 **C:\Program Files\docker\** 폴더에 복사합니다. 

```none
Expand-Archive .\docker-1.12.0.zip
```

Docker 디렉터리를 Nano Server의 시스템 경로에 추가합니다.

> 원격 Nano Server 세션으로 다시 전환해야 합니다.

```none
# for quick use, does not require shell to be restarted
$env:path += “;C:\program files\docker”

# for persistent use, will apply even after a reboot 
setx PATH $env:path /M
```

Docker를 Windows 서비스로 설치합니다.

```none
dockerd --register-service
```

Docker 서비스를 시작합니다.

```none
Start-Service Docker
```

## 기본 컨테이너 이미지 설치

기본 OS 이미지는 모든 Windows Server나 Hyper-V 컨테이너의 기반으로 사용됩니다. 기본 OS 이미지는 Windows Server Core와 Nano Server 둘 다에서 기본 운영 체제로 사용할 수 있으며 `docker pull`를 사용하여 설치할 수 있습니다. Windows 컨테이너 이미지에 대한 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

Nano Server 기본 이미지를 다운로드하여 설치하려면 다음을 실행합니다.

```none
docker pull microsoft/nanoserver
```

> 지금은 Nano Server 기본 이미지만 Nano Server 컨테이너 호스트와 호환됩니다.

## Nano Server에서 Docker 관리

최상의 환경을 위해 원격 시스템에서 Nano Server의 Docker를 관리하는 것이 좋습니다. 이렇게 하려면 다음 항목을 완료해야 합니다.

### 컨테이너 호스트 준비

컨테이너 호스트에서 Docker 연결을 위한 방화벽 규칙을 만듭니다. 보안되지 않은 연결에는 포트 `2375`를 사용하고 보안 연결에는 포트 `2376`을 사용합니다.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2375
```

TCP를 통해 들어오는 연결을 허용하도록 Docker 엔진을 구성합니다.

먼저 Nano Server 호스트의 `c:\ProgramData\docker\config\daemon.json`에 `daemon.json` 파일을 만듭니다.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

그런 후 다음 명령을 실행하여 `daemon.json` 파일에 연결 구성을 추가합니다. 그러면 TCP 포트 2375를 통해 들어오는 연결을 허용하도록 Docker 엔진이 구성됩니다. 이는 보안되지 않은 연결로 권장되지 않지만 격리된 테스트에 사용할 수 있습니다. 이 연결을 보호하는 방법에 대한 자세한 내용은 [Protect the Docker Daemon on Docker.com(Docker.com의 Docker 디몬 보호)](https://docs.docker.com/engine/security/https/)을 참조하세요.

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Docker 서비스를 다시 시작합니다.

```none
Restart-Service docker
```

### 원격 클라이언트 준비

작업할 원격 시스템에 Docker 클라이언트를 다운로드합니다.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

압축된 패키지의 압축을 풉니다.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

다음 두 명령을 실행하여 시스템 경로에 Docker 디렉터리를 추가합니다.

```none
# for quick use, does not require shell to be restarted
$env:path += ";c:\program files\docker"

# for persistent use, will apply even after a reboot 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

완료되면 `docker -H` 매개 변수를 사용하여 Docker 호스트에 액세스할 수 있습니다.

```none
docker -H tcp://<IPADDRESS>:2375 run -it nanoserver cmd
```

환경 변수 `DOCKER_HOST`를 만들어 `-H` 매개 변수 요구 사항을 제거할 수 있습니다. 이 작업에는 다음 PowerShell 명령을 사용할 수 있습니다.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

이 변수를 설정하면 명령이 다음과 같이 표시됩니다.

```none
docker run -it nanoserver cmd
```

## Hyper-V 컨테이너 호스트

Hyper-V 컨테이너를 배포하려면 컨테이너 호스트에 Hyper-V 역할이 필요합니다. Hyper-V 컨테이너에 대한 자세한 내용은 [Hyper-V 컨테이너](../management/hyperv_container.md)를 참조하세요.

Windows 컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우에는 중첩된 가상화를 사용하도록 설정해야 합니다. 중첩된 가상화에 대한 자세한 내용은 [중첩된 가상화](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)를 참조하세요.


Nano Server 컨테이너 호스트에 Hyper-V 역할을 설치합니다.

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Hyper-V 역할을 설치한 후에는 Nano Server 호스트를 다시 부팅해야 합니다.

```none
Restart-Computer
```


<!--HONumber=Aug16_HO4-->


