---
title: "Windows에서 Docker 구성"
description: "Windows에서 Docker 구성"
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: d30136e66bf15dc015629e359422c9b8346b8426
ms.openlocfilehash: 3ee39f57890248951b69887edc87c9fedb13c285

---

# Windows의 Docker 엔진

Docker 엔진 및 클라이언트는 Windows에 포함되어 있지 않으므로 개별적으로 설치하고 구성해야 합니다. 또한 Docker 엔진에서는 여러 사용자 지정 구성을 허용합니다. 몇 가지 예로 디먼이 들어오는 요청을 수락하는 방법, 기본 네트워킹 옵션 및 디버그/로그 설정 등을 구성할 수 있습니다. Windows에서는 이러한 구성을 구성 파일에 지정하거나 Windows 서비스 제어 관리자를 사용하여 지정할 수 있습니다. 이 문서에서는 Docker 엔진을 설치 및 구성하는 방법을 자세히 설명하고 일반적으로 사용하는 구성의 몇 가지 예도 제공합니다.

## Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 이 연습에서는 둘 다 설치됩니다.

Docker 엔진을 다운로드합니다.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

zip 보관 파일을 프로그램 파일로 확장합니다.

```
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
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

## 구성 파일을 사용하여 Docker 구성

Windows에서 Docker 엔진을 구성하는 기본 방법은 구성 파일을 사용하는 것입니다. 구성 파일은 'c:\ProgramData\docker\config\daemon.json'에서 찾을 수 있습니다. 이 파일이 없는 경우 만들 수 있습니다.

참고: 사용 가능한 Docker 구성 옵션 중에 Windows의 Docker에 적용할 수 없는 옵션도 있습니다. 아래 예제에서는 그러한 옵션을 보여 줍니다. Linux용을 포함하여 Docker 엔진 구성에 대한 전체 설명서는 [Docker Daemon]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/)(Docker 디먼)을 참조하세요.

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

구성 파일에 필요한 구성 변경 내용만 추가해야 합니다. 예를 들어 이 샘플에서는 포트 2375를 통해 들어오는 요청을 허용하도록 Docker 엔진을 구성합니다. 다른 모든 구성 옵션은 기본값을 사용합니다.

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

## Docker 서비스에서 Docker 구성

또한 `sc config`를 사용하여 Docker 서비스를 수정하는 방법으로 Docker 엔진을 구성할 수도 있습니다. 이 방법을 사용하면 Docker 서비스에서 직접 Docker 엔진 플래그를 설정합니다. 명령 프롬프트(cmd.exe는 Powershell이 아님)에서 다음 명령을 실행합니다.


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

참고: daemon.json 파일에 이미 `"hosts": ["tcp://0.0.0.0:2375"]`가 포함되어 있으면 이 명령을 실행할 필요가 없습니다.

## 공통 구성

다음 구성 파일 예제에서는 일반적인 Docker 구성을 보여 줍니다. 이러한 구성은 단일 구성 파일로 결합할 수 있습니다.

### 기본 네트워크 만들기 

기본 NAT 네트워크를 만들지 않도록 Docker 엔진을 구성하려면 다음을 사용합니다. 자세한 내용은 [Docker 네트워크 관리](../management/container_networking.md)를 참조하세요.

```none
{
    "bridge" : "none"
}
```

### Docker 보안 그룹 설정

Docker 호스트에 로그인하고 Docker 명령으로 로컬로 실행하면 이러한 명령은 명명된 파이프를 통해 실행됩니다. 기본적으로 Administrators 그룹의 구성원만 명명된 파이프를 통해 Docker 엔진에 액세스할 수 있습니다. 이 액세스 권한이 있는 보안 그룹을 지정하려면 `group` 플래그를 사용합니다.

```none
{
    "group" : "docker"
}
```

## 프록시 구성

`docker search` 및 `docker pull`에 대한 프록시 정보를 설정하려면 `HTTP_PROXY` 또는 `HTTPS_PROXY` 이름과 프록시 정보 값을 사용하여 Windows 환경 변수를 만듭니다. 이 작업은 PowerShell에서 다음과 유사한 명령을 사용하여 수행할 수 있습니다.

```none
[Environment]::SetEnvironmentVariable("HTTP_PROXY”, “http://username:password@proxy:port/”, [EnvironmentVariableTarget]::Machine)
```

변수를 설정한 후에는 Docker 서비스를 다시 시작합니다.

```none
restart-service docker
```

자세한 내용은 [Daemon Socket Options on Docker.com](https://docs.docker.com/v1.10/engine/reference/commandline/daemon/#daemon-socket-option)(Docker.com의 디먼 소켓 옵션)을 참조하세요.

## 로그 수집

Docker 엔진은 파일 대신 Windows '응용 프로그램' 이벤트 로그에 기록합니다. 이러한 로그는 Windows PowerShell을 사용하여 쉽게 읽고 정렬하고 필터링할 수 있습니다.

예를 들어 가장 오래된 순서부터 시작하여 최근 5분의 Docker 엔진 로그가 표시됩니다.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

또한 다른 도구 또는 스프레드시트로 읽을 수 있도록 CSV 파일로 쉽게 파이프할 수 있습니다.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.csv ```



<!--HONumber=Oct16_HO1-->


