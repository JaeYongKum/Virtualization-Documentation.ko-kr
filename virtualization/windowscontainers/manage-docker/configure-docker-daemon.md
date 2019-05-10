---
title: Windows에서 Docker 구성
description: Windows에서 Docker 구성
keywords: Docker, 컨테이너
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: a04d356415e7bed84980747edc927cc1eaa1e7c1
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621091"
---
# <a name="docker-engine-on-windows"></a>Windows의 Docker 엔진

Docker 엔진 및 클라이언트 Windows에 포함 되지 및 설치 하 고 개별적으로 구성 해야 합니다. 또한 Docker 엔진에서는 여러 사용자 지정 구성을 허용합니다. 몇 가지 예로 디먼이 들어오는 요청을 수락하는 방법, 기본 네트워킹 옵션 및 디버그/로그 설정 등을 구성할 수 있습니다. Windows에서는 이러한 구성을 구성 파일에 지정하거나 Windows 서비스 제어 관리자를 사용하여 지정할 수 있습니다. 이 문서는 Docker 엔진을 설치 및 구성 하는 방법에 자세히 설명 하 고도 자주 사용 되는 구성의 몇 가지 예를 제공 합니다.

## <a name="install-docker"></a>Docker 설치

Windows 컨테이너를 사용 하려면 Docker를 해야 합니다. Docker는 Docker 엔진(dockerd.exe) 및 Docker 클라이언트(docker.exe)로 구성됩니다. 조직에 설치 된 하는 가장 쉬운 방법은 빠른 시작 가이드 도움이 되는 모든 설정 및 첫 번째 컨테이너를 실행 중인 합니다.

- [Windows Server 2019의 Windows 컨테이너](../quick-start/quick-start-windows-server.md)
- [Windows 10의 Windows 컨테이너](../quick-start/quick-start-windows-10.md)

설치를 [사용 하 여 Docker EE를 설치 하는 스크립트를](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)참조 하세요.

Docker를 사용 하려면 먼저 컨테이너 이미지를 설치 해야 합니다. 자세한 내용은 [이미지를 사용 하기 위한 빠른 시작 가이드](../quick-start/quick-start-images.md)를 참조 하세요.

## <a name="configure-docker-with-a-configuration-file"></a>구성 파일을 사용 하 여 Docker 구성

Windows에서 Docker 엔진을 구성하는 기본 방법은 구성 파일을 사용하는 것입니다. 구성 파일은 'C:\ProgramData\Docker\config\daemon.json'에서 찾을 수 있습니다. 아직 없는 경우이 파일을 만들 수 있습니다.

>[!NOTE]
>Windows에서 Docker에 모든 사용 가능한 Docker 구성 옵션에 적용 됩니다. 다음 예제에서는 적용 않는 구성 옵션을 보여 줍니다. Docker 엔진 구성에 대 한 자세한 내용은 [Docker 디먼 구성 파일](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)을 참조 하세요.

```json
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

구성 파일을 구성 변경 내용만된 추가 해야 합니다. 예를 들어 다음 샘플에서는 포트 2375에서 들어오는 연결을 허용 하도록 Docker 엔진을 구성 합니다. 다른 모든 구성 옵션은 기본값을 사용합니다.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

마찬가지로, 다음 샘플에서는 대체 경로에 이미지 및 컨테이너를 유지 하도록 Docker 디먼을 구성 합니다. 지정 하지 않은 경우 기본값은 `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

다음 샘플에서는 포트 2376 통한 보안된 연결만 허용 하도록 Docker 디먼을 구성 합니다.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Docker 서비스에서 Docker 구성

Docker 서비스를 수정 하 여 Docker 엔진을 구성할 `sc config`. 이 방법을 사용하면 Docker 서비스에서 직접 Docker 엔진 플래그를 설정합니다. 명령 프롬프트(cmd.exe는 PowerShell이 아님)에서 다음 명령을 실행합니다.

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Daemon.json 파일에 이미 포함 되어 있으면이 명령을 실행할 필요가 없습니다는 `"hosts": ["tcp://0.0.0.0:2375"]` 항목입니다.

## <a name="common-configuration"></a>일반적인 구성

다음 구성 파일 예제에서는 일반적인 Docker 구성을 보여 줍니다. 이러한 구성은 단일 구성 파일로 결합할 수 있습니다.

### <a name="default-network-creation"></a>기본 네트워크 만들기

파일을 기본 NAT 네트워크를 만들지 못합니다 되도록 Docker 엔진을 구성 하려면 다음 구성을 사용 합니다.

```json
{
    "bridge" : "none"
}
```

자세한 내용은 [Docker 네트워크 관리](../container-networking/network-drivers-topologies.md)를 참조하세요.

### <a name="set-docker-security-group"></a>Docker 보안 그룹 설정

Docker 호스트에 로그인 하 고 Docker 명령으로 로컬로 실행 하는 경우 이러한 명령은 명명 된 파이프를 통해 실행 됩니다. 기본적으로 Administrators 그룹의 구성원만 명명된 파이프를 통해 Docker 엔진에 액세스할 수 있습니다. 이 액세스 권한이 있는 보안 그룹을 지정하려면 `group` 플래그를 사용합니다.

```json
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

자세한 내용은 [Docker.com의 Windows 구성 파일](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)을 참조 하세요.

## <a name="how-to-uninstall-docker"></a>Docker를 제거 하는 방법

이 섹션에는 Docker를 제거 하 고 Windows 10 또는 Windows Server 2016 시스템에서 Docker 시스템 구성 요소의 전체 정리를 수행 하는 방법을 알려줍니다.

>[!NOTE]
>모든 명령은 관리자 권한 PowerShell 세션에서 다음이 지침에서 실행 해야 합니다.

### <a name="prepare-your-system-for-dockers-removal"></a>Docker의 제거를 위해 시스템 준비

Docker를 제거 하기 전에 시스템에서 실행 중인 컨테이너가 없는지 있는지 확인 합니다.

컨테이너를 실행 하기 위한 확인 하려면 다음 cmdlet을 실행 합니다.

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

시스템에서 Docker를 제거 하기 전에 모든 컨테이너, 컨테이너 이미지, 네트워크 및 볼륨을 제거 하는 좋은 방법 이기도 합니다. 다음 cmdlet을 실행 하 여이 수행할 수 있습니다.

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Docker 제거

다음으로, 실제로 Docker를 제거 해야 합니다.

Windows 10에서 Docker를 제거 하려면

- **설정**으로 이동 > Windows 10 컴퓨터에서**앱**
- **앱 & 기능** **Windows 용 Docker를** 찾기
- **Windows 용 Docker**로 이동 > **제거**

Windows Server 2016에서 Docker를 제거 합니다.

관리자 권한 PowerShell 세션에서 다음 예와 같이 시스템에서 Docker 모듈 및 해당 패키지 관리 공급자를 제거 **제거 패키지** 및 **제거 모듈** cmdlet 사용:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>사용 하 여 Docker를 설치 하는 데 사용한 패키지 공급자를 찾을 수 있습니다. `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Docker 데이터 및 시스템 구성 요소 정리

Docker를 제거한 후 Docker이 사라진 후 해당 구성을 시스템에 유지 되지 않습니다 Docker의 기본 네트워크 제거 해야 합니다. 다음 cmdlet을 실행 하 여이 수행할 수 있습니다.

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

시스템에서 Docker의 프로그램 데이터를 제거 하려면 다음 cmdlet을 실행 합니다.

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Windows에서 Docker/컨테이너와 관련된 Windows 선택적 기능을 제거하고자 할 수도 있습니다.

여기에 Docker를 설치할 때 모든 Windows 10 또는 Windows Server 2016에서 자동으로 활성화 되는 "컨테이너" 기능이 포함 됩니다. Docker를 설치할 때 Windows 10에서 자동으로 활성화되지만 Windows Server 2016에서는 명시적으로 활성화해야 하는 "Hyper-V"가 포함될 수도 있습니다.

>[!IMPORTANT]
>[Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) 기능은 뿐 아니라 더 많은 컨테이너 수 있는 일반적인 가상화 기능입니다. Hyper-v 기능을 비활성화 하기 전에 없는 다른 가상화 구성 요소는 시스템에서 Hyper-v를 필요로 하는 있는지 확인 합니다.

Windows 10에서 Windows 기능 제거 하려면:

- **제어판** > **프로그램** > **프로그램 및 기능** > **Windows 기능 켜기 또는 끄기**.
- 기능 또는 사용 하지 않도록 설정 하려는 기능 이름 찾기-이 경우 **컨테이너** 와 **Hyper-v**(선택 사항).
- 비활성화 하려는 기능 이름 옆에 있는 확인란의 선택을 취소 합니다.
- **"확인"를** 선택 합니다.

제거 하려면 Windows Server 2016에서 Windows 기능:

관리자 권한 PowerShell 세션에서 **컨테이너** 및 (선택 사항) 시스템에서 **Hyper-v** 기능을 사용 하지 않도록 설정 하려면 다음 cmdlet을 실행 합니다.

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>시스템 재부팅

제거 및 정리를 완료 하려면 시스템을 다시 관리자 권한 PowerShell 세션에서 다음 cmdlet을 실행 합니다.

```powershell
Restart-Computer -Force
```
