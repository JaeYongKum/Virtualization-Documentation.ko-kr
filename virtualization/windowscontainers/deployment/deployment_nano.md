---
title: "Nano Server에 Windows 컨테이너 배포"
description: "Nano Server에 Windows 컨테이너 배포"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 1ba6af300d0a3eba3fc6d27598044f983a4c9168
ms.openlocfilehash: f2790186aa641378b1981a1f946665ca46fdbd73

---

# 컨테이너 호스트 배포 - Nano Server

**이 예비 콘텐츠는 변경될 수 있습니다.** 

Nano Server에서 Windows 컨테이너의 구성을 시작하려면 Nano Server가 실행되는 시스템이 필요하며 이 시스템과 PowerShell의 원격 연결도 있어야 합니다. Nano Server 배포 및 연결에 대한 자세한 내용은 [Getting Started with Nano Server(Nano Server 시작)]( https://technet.microsoft.com/en-us/library/mt126167.aspx)를 참조하세요.

Nano Server의 평가본은 [여기](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)에 있습니다.

## 컨테이너 기능 설치

Nano Server 패키지 관리 공급자를 설치합니다.

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

## Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 다음 단계를 사용하여 Docker 디먼 및 클라이언트를 설치합니다.

Nano Server 호스트에 Docker 실행 파일을 저장할 폴더를 만듭니다.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Docker 디먼 및 클라이언트를 다운로드하고 컨테이너 호스트의 'C:\Program Files\docker\'에 복사합니다. 

**참고** - Nano Server는 현재 `Invoke-WebRequest`를 지원하지 않으며 원격 시스템에서 다운로드를 완료해야 합니다.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Docker 클라이언트를 다운로드합니다.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Docker 디먼 및 클라이언트를 다운로드하고 Nano Server 컨테이너 호스트에 복사했으면 호스트에서 이 명령을 실행하여 Docker를 Windows 서비스로 설치합니다.

```none
& 'C:\Program Files\docker\dockerd.exe' --register-service
```

Docker 서비스를 시작합니다.

```none
Start-Service Docker
```

## 기본 컨테이너 이미지 설치

기본 OS 이미지는 모든 Windows Server나 Hyper-V 컨테이너의 기반으로 사용됩니다. 기본 OS 이미지는 Windows Server Core와 Nano Server 둘 다에서 기본 운영 체제로 사용할 수 있으며 컨테이너 이미지 공급자를 사용하여 설치할 수 있습니다. Windows 컨테이너 이미지에 대한 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

다음 명령을 사용하여 컨테이너 이미지 공급자를 설치할 수 있습니다.

```none
Install-PackageProvider ContainerImage -Force
```

Nano Server 기본 이미지를 다운로드하여 설치하려면 다음을 실행합니다.

```none
Install-ContainerImage -Name NanoServer
```

**참고** - 지금은 Nano Server 기본 이미지만 Nano Server 컨테이너 호스트와 호환됩니다.

Docker 서비스를 다시 시작합니다.

```none
Restart-Service Docker
```

Nano Server 기본 이미지에 최신으로 태그 지정합니다.

```none
& 'C:\Program Files\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Nano Server에서 Docker 관리

최상의 환경을 위해 원격 시스템에서 Nano Server의 Docker를 관리하는 것이 좋습니다. 이렇게 하려면 다음 항목을 완료해야 합니다.

**Docker 디먼 준비:**

컨테이너 호스트에서 Docker 연결을 위한 방화벽 규칙을 만듭니다. 보안되지 않은 연결에는 포트 `2375`를 사용하고 보안 연결에는 포트 `2376`을 사용합니다.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

TCP를 통해 들어오는 연결을 허용하도록 Docker 디먼을 구성합니다.

먼저 `c:\ProgramData\docker\config\daemon.json`에서 `daemon.json` 파일을 만듭니다.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

그런 다음 이 JSON을 구성 파일에 복사합니다. 그러면 TCP 포트 2375를 통해 들어오는 연결을 허용하도록 Docker 디먼이 구성됩니다. 이는 보안되지 않은 연결로 권장되지 않지만 격리된 테스트에 사용할 수 있습니다.

```none
{
    "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
}
```

다음 예제에서는 보안된 원격 연결을 구성합니다. TLS 인증서를 만들고 적절한 위치에 복사해야 합니다. 자세한 내용은 [Windows의 Docker 디먼](./docker_windows.md)을 참조하세요.

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

Docker 서비스를 다시 시작합니다.

```none
Restart-Service docker
```

**Docker 클라이언트 준비:**

Docker 클라이언트를 저장할 디렉터리를 만듭니다.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

원격 관리 시스템에 Docker 클라이언트를 다운로드합니다.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "C:\Program Files\docker\docker.exe"
```

시스템 경로에 Docker 디렉터리를 추가합니다.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

수정된 경로를 인식할 수 있도록 PowerShell 또는 명령 세션을 다시 시작합니다.

완료되면 `docker -H` 매개 변수를 사용하여 Docker 호스트에 액세스할 수 있습니다.

```none
docker -H tcp://10.0.0.5:2375 run -it nanoserver cmd
```

환경 변수 `DOCKER_HOST`를 만들어 `-H` 매개 변수 요구 사항을 제거할 수 있습니다. 이 작업에는 다음 PowerShell 명령을 사용할 수 있습니다.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2375"
```

이 변수를 설정하면 명령이 다음과 같이 표시됩니다.

```none
docker run -it nanoserver cmd
```

## Hyper-V 컨테이너 호스트

Hyper-V 컨테이너를 배포하려면 Hyper-V 역할이 필요합니다. Hyper-V 컨테이너에 대한 자세한 내용은 [Hyper-V 컨테이너](../management/hyperv_container.md)를 참조하세요.

Windows 컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우에는 중첩된 가상화를 사용하도록 설정해야 합니다. 중첩된 가상화에 대한 자세한 내용은 [중첩된 가상화](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)를 참조하세요.


Hyper-V 역할 설치:

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Hyper-V 역할을 설치한 후에는 Nano Server 호스트를 다시 부팅해야 합니다.

```none
Restart-Computer
```






<!--HONumber=Jun16_HO4-->


