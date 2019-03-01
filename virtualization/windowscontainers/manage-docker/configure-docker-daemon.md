---
title: Windows에서 Docker 구성
description: Windows에서 Docker 구성
keywords: Docker, 컨테이너
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: bbc405fc2a490cfe5082be112fde724707e24785
ms.sourcegitcommit: 21d93e5febd9b1b47ae1aa59d08086e6ec1691e0
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/28/2019
ms.locfileid: "9121055"
---
# <a name="docker-engine-on-windows"></a>Windows의 Docker 엔진

Docker 엔진 및 클라이언트 Windows 포함 되며 설치 하 고 개별적으로 구성 해야 합니다. 또한 Docker 엔진에서는 여러 사용자 지정 구성을 허용합니다. 몇 가지 예로 디먼이 들어오는 요청을 수락하는 방법, 기본 네트워킹 옵션 및 디버그/로그 설정 등을 구성할 수 있습니다. Windows에서는 이러한 구성을 구성 파일에 지정하거나 Windows 서비스 제어 관리자를 사용하여 지정할 수 있습니다. 이 문서는 Docker 엔진을 설치 및 구성 하는 방법에 자세히 설명 하 고도 자주 사용 되는 구성의 몇 가지 예를 제공 합니다.


## <a name="install-docker"></a>Docker 설치
Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진(dockerd.exe) 및 Docker 클라이언트(docker.exe)로 구성됩니다. 모든 것을 가장 쉽게 설치하는 방법은 빠른 시작 가이드에 있습니다. 데 도움을 모두 설정 하 고 첫 번째 컨테이너를 실행 합니다. 

* [Windows Server 2019의 Windows 컨테이너](../quick-start/quick-start-windows-server.md)
* [Windows 10의 Windows 컨테이너](../quick-start/quick-start-windows-10.md)

스크립트 설치에 대한 내용은 [스크립트를 이용하여 Docker EE 설치](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)를 참조하세요.

Docker를 사용 하려면 먼저 컨테이너 이미지 설치 해야 합니다. 자세한 내용은 [이미지를 사용하는 빠른 시작 가이드](../quick-start/quick-start-images.md)를 참조하십시오.

## <a name="configure-docker-with-configuration-file"></a>구성 파일을 사용하여 Docker 구성

Windows에서 Docker 엔진을 구성하는 기본 방법은 구성 파일을 사용하는 것입니다. 구성 파일은 'C:\ProgramData\Docker\config\daemon.json'에서 찾을 수 있습니다. 이 파일이 없는 경우 만들 수 있습니다.

참고: 사용 가능한 Docker 구성 옵션 중에 Windows의 Docker에 적용할 수 없는 옵션도 있습니다. 아래 예제에서는 그러한 옵션을 보여 줍니다. Docker 엔진 구성에 대한 전체 설명서는 [Docker daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)(Docker 디먼 구성 파일)을 참조하세요.

```
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
    "data-root": "",
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

```
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

마찬가지로 이 샘플에서는 대체 경로에 이미지 및 컨테이너를 유지하도록 Docker 디먼을 구성합니다. 지정하지 않은 경우 기본값은 c:\programdata\docker입니다.

```
{    
    "data-root": "d:\\docker"
}
```

마찬가지로 이 샘플에서는 포트 2376을 통한 보안 연결만 허용하도록 Docker 디먼을 구성합니다.

```
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Docker 서비스에서 Docker 구성

또한 `sc config`를 사용하여 Docker 서비스를 수정하는 방법으로 Docker 엔진을 구성할 수도 있습니다. 이 방법을 사용하면 Docker 서비스에서 직접 Docker 엔진 플래그를 설정합니다. 명령 프롬프트(cmd.exe는 PowerShell이 아님)에서 다음 명령을 실행합니다.


```
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

참고: daemon.json 파일에 이미 `"hosts": ["tcp://0.0.0.0:2375"]`가 포함되어 있으면 이 명령을 실행할 필요가 없습니다.

## <a name="common-configuration"></a>공통 구성

다음 구성 파일 예제에서는 일반적인 Docker 구성을 보여 줍니다. 이러한 구성은 단일 구성 파일로 결합할 수 있습니다.

### <a name="default-network-creation"></a>기본 네트워크 만들기 

기본 NAT 네트워크를 만들지 않도록 Docker 엔진을 구성하려면 다음을 사용합니다. 자세한 내용은 [Docker 네트워크 관리](../container-networking/network-drivers-topologies.md)를 참조하세요.

```
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>Docker 보안 그룹 설정

Docker 호스트에 로그인하고 Docker 명령으로 로컬로 실행하면 이러한 명령은 명명된 파이프를 통해 실행됩니다. 기본적으로 Administrators 그룹의 구성원만 명명된 파이프를 통해 Docker 엔진에 액세스할 수 있습니다. 이 액세스 권한이 있는 보안 그룹을 지정하려면 `group` 플래그를 사용합니다.

```
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>프록시 구성

`docker search` 및 `docker pull`에 대한 프록시 정보를 설정하려면 `HTTP_PROXY` 또는 `HTTPS_PROXY` 이름과 프록시 정보 값을 사용하여 Windows 환경 변수를 만듭니다. 이 작업은 PowerShell에서 다음과 유사한 명령을 사용하여 수행할 수 있습니다.

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

변수를 설정한 후에는 Docker 서비스를 다시 시작합니다.

```powershell
Restart-Service docker
```

자세한 내용은 [Docker.com의 Windows 구성 파일](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)을 참조하세요.

## <a name="uninstall-docker"></a>Docker 제거
*이 섹션의 단계를 사용하여 Docker를 제거하고 Windows 10 또는 Windows Server 2016 시스템에서 Docker 시스템 구성 요소의 전체 정리를 수행할 수 있습니다.*

> 참고: 다음 단계의 모든 명령은 **권리자 권한** PowerShell 세션으로 실행해야 합니다.

### <a name="step-1-prepare-your-system-for-dockers-removal"></a>1단계: Docker의 제거를 위해 시스템 준비 
Docker를 제거하기 전에 시스템에서 실행 중인 컨테이너가 없는지 확인하지 않은 경우 확인하는 것이 좋습니다. 이를 수행하는 몇 가지 유용한 명령은 다음과 같습니다.
```
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```
Docker를 제거하기 전에 모든 컨테이너, 컨테이너 이미지, 네트워크 및 볼륨 또한 시스템에서 제거하는 것이 좋습니다.
```
docker system prune --volumes --all
```

### <a name="step-2-uninstall-docker"></a>2단계: Docker 제거 

#### ***<a name="steps-to-uninstall-docker-on-windows-10"></a>Windows 10에서 Docker를 제거하는 단계:10:***
- Windows 10 컴퓨터에서 **"설정" > "앱"** 으로 이동합니다.
- **"앱 및 기능"** 아래에서 **"Windows용 Docker"** 를 선택합니다.
- **"Windows용 Docker" > "제거"** 를 클릭합니다.

#### ***<a name="steps-to-uninstall-docker-on-windows-server-2016"></a>Windows Server 2016에서 Docker를 제거하는 단계:16:***
권리자 권한 PowerShell 세션에서 `Uninstall-Package` 및 `Uninstall-Module` cmdlet을 사용하여 시스템에서 Docker 모듈 및 해당 패키지 관리 공급자를 제거합니다. 
> 팁: Docker를 설치하는 데 사용한 패키지 공급자를 찾을 수 있습니다. `PS C:\> Get-PackageProvider -Name *Docker*`

*예*:
```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

### <a name="step-3-cleanup-docker-data-and-system-components"></a>3단계: Docker 데이터 및 시스템 구성 요소 정리
Docker의 *기본 네트워크*를 제거하여 Docker를 삭제하고 난 후 구성이 시스템에 남아 있지 않도록 합니다.
```
Get-HNSNetwork | Remove-HNSNetwork
```
시스템에서 Docker의 *프로그램 데이터* 제거:
```
Remove-Item "C:\ProgramData\Docker" -Recurse
```
Windows에서 Docker/컨테이너와 관련된 *Windows 선택적 기능*을 제거하고자 할 수도 있습니다. 

최소한 여기에는 Docker를 설치할 때 모든 Windows 10 또는 Windows Server 2016에서 자동으로 활성화되는 "컨테이너" 기능이 포함됩니다. Docker를 설치할 때 Windows 10에서 자동으로 활성화되지만 Windows Server 2016에서는 명시적으로 활성화해야 하는 "Hyper-V"가 포함될 수도 있습니다.

> **Hyper-V 비활성화에 대한 중요한 참고 사항:** [Hyper-V 기능](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/)은 단순히 컨테이너보다 더 많은 것을 활성화할 수 있는 일반적인 가상화 기능입니다! Hyper-V 기능을 비활성화하기 전에 이를 필요로 하는 다른 가상화 구성 요소는 없는지 확인합니다.

#### ***<a name="steps-to-remove-windows-features-on-windows-10"></a>Windows 10에서 Windows 기능을 제거하는 단계:10:***
- Windows 10 컴퓨터에서 **"제어판" > "프로그램" > "프로그램 및 기능" > "Windows 기능 켜기/끄기"** 로 이동합니다.
- 비활성화하려는 기능의 이름을 찾습니다. 이 경우 **"컨테이너"** 및 (선택 사항) **"Hyper-V"** 입니다.
- 비활성화하려는 기능 이름 옆의 확인란 선택을 **취소**합니다.
- **"확인"** 을 클릭합니다.

#### ***<a name="steps-to-remove-windows-features-on-windows-server-2016"></a>Windows Server 2016에서 Windows 기능을 제거하는 단계:16:***
관리자 권한 PowerShell 세션에서 다음 명령을 사용하여 시스템에서 **"컨테이너"** 및 (선택 사항) **"Hyper-V"** 기능을 비활성화합니다.
```
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V 
```

### <a name="step-4-reboot-your-system"></a>4단계: 시스템 재부팅
제거/정리 단계를 완료하려면 관리자 권한 PowerShell 세션에서 다음을 실행합니다.
```
Restart-Computer -Force
```
