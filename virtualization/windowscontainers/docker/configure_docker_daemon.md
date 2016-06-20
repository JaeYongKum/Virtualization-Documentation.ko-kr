---
title: Windows에서 Docker 구성
description: Windows에서 Docker 구성
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
---

Docker 엔진은 Windows에 포함되지 않으며 별도로 설치 및 구성해야 합니다. 또한 Docker 디먼에서는 여러 사용자 지정 구성을 허용합니다. 몇 가지 예로 디먼이 들어오는 요청을 수락하는 방법, 기본 네트워킹 옵션 및 디버그/로그 설정 등을 구성할 수 있습니다. Windows에서는 이러한 구성을 구성 파일에 지정하거나 Windows 서비스 제어 관리자를 사용하여 지정할 수 있습니다. 이 문서에서는 docker 디먼을 설치 및 구성하는 방법을 자세히 설명하고 일반적으로 사용하는 구성의 몇 가지 예도 제공합니다.

## Docker 설치

Windows 컨테이너로 작업하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 이 연습에서는 둘 다 설치됩니다.

Docker 실행 파일을 저장할 폴더를 만듭니다.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Docker 디먼을 다운로드합니다.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Docker 클라이언트를 다운로드합니다.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

시스템 경로에 Docker 디렉터리를 추가합니다. 완료되면 수정된 경로를 인식할 수 있도록 PowerShell 세션을 다시 시작합니다.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Docker를 Windows 서비스로 설치하려면 다음을 실행합니다.

```none
dockerd --register-service
```

설치되면 서비스를 시작할 수 있습니다.

```none
Start-Service Docker
```

Docker를 사용하려면 먼저 컨테이너 이미지를 설치해야 합니다. 자세한 내용은 [컨테이너 이미지 관리](../management/manage_images.md)를 참조하세요.

## Docker 구성 파일

Windows에서 Docker 디먼을 구성하는 기본 방법은 구성 파일을 사용하는 것입니다. 구성 파일은 'c:\ProgramData\docker\config\daemon.json'에서 찾을 수 있습니다. 이 파일이 없는 경우 만들 수 있습니다.

참고: 사용 가능한 Docker 구성 옵션 중에 Windows의 Docker에 적용할 수 없는 옵션도 있습니다. 아래 예제에서는 그러한 옵션을 보여 줍니다. Linux용을 포함하여 Docker 디먼 구성에 대한 전체 설명서는 [Docker Daemon(Docker 디먼)]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/)을 참조하세요.

```none
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "graph": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

구성 파일에 필요한 구성 변경 내용만 추가해야 합니다. 예를 들어 이 샘플에서는 포트 2375를 통해 들어오는 요청을 허용하도록 Docker 디먼을 구성합니다. 다른 모든 구성 옵션은 기본값을 사용합니다.

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

마찬가지로 이 샘플에서는 포트 2376을 통한 보안 연결만 허용하도록 Docker 디먼을 구성합니다.

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```



## 서비스 제어 관리자

또한 `sc config`를 사용하여 Docker 서비스를 수정하는 방법으로 Docker 디먼을 구성할 수도 있습니다. 이 방법을 사용하면 Docker 서비스에서 직접 Docker 디먼 플래그를 설정합니다.


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## 공통 구성

다음 구성 파일 예제에서는 일반적인 Docker 구성을 보여 줍니다. 이러한 구성은 단일 구성 파일로 결합할 수 있습니다.

### 기본 네트워크 만들기 

기본 NAT 네트워크를 만들지 않도록 Docker 디먼을 구성하려면 다음을 사용합니다. 자세한 내용은 [Docker 네트워크 관리](../management/container_networking.md)를 참조하세요.

```none
{
    "bridge" : "none"
}
```

### Docker 보안 그룹 설정

Docker 호스트에 로그인하고 Docker 명령으로 로컬로 실행하면 이러한 명령은 명명된 파이프를 통해 실행됩니다. 기본적으로 Administrators 그룹의 구성원만 명명된 파이프를 통해 Docker 디먼에 액세스할수 있습니다. 이 액세스 권한이 있는 보안 그룹을 지정하려면 `group` 플래그를 사용합니다.

```none
{
    "group" : "docker"
}
```


<!--HONumber=Jun16_HO2-->


