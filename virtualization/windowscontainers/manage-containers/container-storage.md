---
title: Windows Server 컨테이너 저장소
description: Windows Server 컨테이너가 호스트 및 기타 저장소 유형을 사용하는 방법
keywords: 컨테이너, 볼륨, 저장소, 마운트, 바인딩 마운트
author: patricklang
ms.openlocfilehash: 9dde3b2d7be10a8d3d393f8426976dfc5bdacfab
ms.sourcegitcommit: 9653a3f7451011426f8af934431bb14dbcb30a62
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/21/2018
ms.locfileid: "2082904"
---
# <a name="overview"></a>개요

<!-- Great diagram would be great! -->


## <a name="layer-storage"></a>계층 저장소

컨테이너에 내장된 모든 파일입니다. 사용자가 `docker pull`한 다음 해당 컨테이너를 `docker run`할 때마다 동일합니다.


### <a name="where-layers-are-stored-and-how-to-change-it"></a>계층이 저장되는 위치 및 이를 변경하는 방법

기본 설치에서 계층은 `C:\ProgramData\docker`에 저장되며 "image"와 "windowsfilter" 디렉터리에 분산됩니다. [Windows의 Docker 엔진](../manage-docker/configure-docker-daemon.md) 설명서에서와 같이 `docker-root` 구성을 사용하여 계층이 저장되는 위치를 변경할 수 있습니다.

> [!NOTE]
> 계층 저장소에 대해 NTFS만 지원됩니다. ReFS는 지원되지 않습니다.

계층 디렉터리에서 어떠한 파일도 수정해서는 안 됩니다. 다음과 같은 명령을 사용하여 신중하게 관리해야 합니다.

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker Pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker load](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>계층 저장소에서 지원되는 작업

실행 중인 컨테이너는 트랜잭션을 제외하고 대부분의 NTFS 작업을 사용할 수 있습니다. 이에는 ACL 설정이 포함되며 컨테이너 내에 있는 모든 ACL을 확인합니다. 컨테이너 내에서 여러 사용자로 프로세스를 실행하려면 `RUN net user /create ...`로 `Dockerfile` 내에 사용자를 만들고, ACL 파일을 설정한 다음 프로세스를 구성하면 [Dockerfile USER 지시문](https://docs.docker.com/engine/reference/builder/#user)을 사용하여 해당 사용자로 실행할 수 있습니다.


##  <a name="image-size"></a>이미지 크기
Windows 응용 프로그램의 일반적인 패턴은 새 파일을 설치하거나 만들기 전에 또는 임시 파일을 정리하기 위한 트리거로서 사용 가능한 디스크 공간의 크기를 쿼리하는 것입니다.  응용 프로그램 호환성의 극대화를 목표로 Windows 컨테이너에 있는 C: 드라이브는 20GB의 사용 가능한 가상 크기를 나타냅니다.  일부 사용자는 이 기본값을 다시 정의하여 사용 가능한 공간을 더 적거나 큰 값으로 구성하고자 할 수 있으며 "storage-opt" 구성 내에서 "size" 옵션을 통해 이를 수행할 수 있습니다.

### <a name="examples"></a>예
명령줄: `docker run --storage-opt "size=50GB" microsoft/windowsservercore:1709 cmd`

Docker 구성 파일
```
"storage-opts": [
    "size=50GB"
  ]
```
> 이 방법은 Docker 빌드에 대해 작동합니다.
Docker 구성 파일을 수정하는 것에 대한 자세한 내용은 [Docker 구성](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) 설명서를 참조하세요.


## <a name="persistent-volumes"></a>영구 볼륨

영구 저장소는 몇 가지 방법으로 컨테이너에 제공할 수 있습니다.

- 바인딩 마운트
- 명명된 볼륨

Docker에는 [볼륨을 사용하는](https://docs.docker.com/engine/admin/volumes/volumes/) 방법에 대한 훌륭한 개요가 들어 있으니 먼저 읽는 것이 좋습니다. 이 페이지의 나머지 부분에서는 Linux와 Windows의 차이점에 초점을 맞추며 Windows의 예를 제공합니다.


### <a name="bind-mounts"></a>바인딩 마운트

[바인딩 마운트](https://docs.docker.com/engine/admin/volumes/bind-mounts/)를 통해 컨테이너가 호스트와 디렉터리를 공유할 수 있습니다. 이는 컨테이너를 다시 시작하거나 여러 컨테이서와 공유하고자 할 때 사용할 수 있는 로컬 컴퓨터에 파일을 저장할 장소를 원하는 경우 유용합니다. 여러 대의 컴퓨터에서 동일한 파일에 액세스하여 컨테이너를 실행하려는 경우 명명된 볼륨 또는 SMB 마운트를 대신 사용해야 합니다.

#### <a name="permissions"></a>권한

바인딩 마운트에 사용되는 권한 모델은 사용자 컨테이너에 대한 격리 수준에 따라 달라집니다.

Windows Server 버전 1709의 Linux 컨테이너를 포함하여 **Hyper-V 격리**를 사용하는 컨테이너는 단순 읽기 전용 또는 읽기-쓰기 권한 모델을 사용합니다.
`LocalSystem` 계정을 사용하는 호스트에서 파일에 액세스합니다. 컨테이너에서 액세스가 거부된 경우 `LocalSystem`이 호스트의 디렉터리에 액세스할 수 있는지 확인합니다.
읽기 전용 플래그를 사용하면 컨테이너 내에서 볼륨을 변경하더라도 호스트의 디렉터리에 표시되거나 유지되지 않습니다.

**프로세스 격리**를 사용하는 Windows Server 컨테이너는 데이터에 액세스하기 위해 컨테이너 내에 있는 프로세스 ID를 사용하기 때문에(파일 ACL이 적용됨을 의미함) 약간 다릅니다.
컨테이너(Windows Server Core의 "ContainerAdministrator" 및 Nano Server 컨테이너의 "ContainerUser", 기본값)에서 실행되는 프로세스의 ID가 `LocalSystem` 대신 마운트된 볼륨의 파일 및 디렉터리에 액세스하는 데 사용되며 데이터를 사용하기 위해 액세스 권한을 부여받아야 합니다.
이러한 ID는 파일이 저장되는 호스트가 아닌 컨테이너의 컨텍스트 내에만 존재하기 때문에 컨테이너에 대한 액세스 권한을 부여하기 위해 ACL을 구성할 때 `Authenticated Users`와 같은 잘 알려진 보안 그룹을 사용해야 합니다.

> [!WARNING]
> 신뢰할 수 없는 컨테이너에 `C:\`와 같이 중요한 디렉터리를 바인딩 마운트하지 마십시오. 이렇게 하면 일반적으로 액세스할 수 없으며 보안 위반을 일으킬 수 있는 호스트에서 파일을 변경할 수 있습니다.

사용 예제: 

- `docker run -v c:\ContainerData:c:\data:RO` 읽기 전용 액세스
- `docker run -v c:\ContainerData:c:\data:RW` 읽기-쓰기 액세스
- `docker run -v c:\ContainerData:c:\data` 읽기-쓰기 액세스(기본값)

#### <a name="symlinks"></a>symlink

symlink는 컨테이너에서 확인됩니다. symlink인 컨테이너 또는 symlink를 포함하는 컨테이너에 대한 호스트 경로를 바인딩 마운트하면 컨테이너는 이에 액세스할 수 없게 됩니다.

#### <a name="smb-mounts"></a>SMB 마운트

Windows Server 버전 1709에서 "SMB 글로벌 매핑"이라는 새로운 기능을 통해 호스트에서 SMB 공유를 마운트한 다음 이 공유에 있는 디렉터리를 컨테이너에 전달할 수 있습니다. 컨테이너는 특정 서버, 공유, 사용자 이름 또는 함호로 구성할 필요가 없습니다. 모두 호스트에서 대신 처리됩니다. 컨테이너는 로컬 저장소에서처럼 동일하게 작동합니다.

##### <a name="configuration-steps"></a>구성 단계

1. 컨테이너 호스트에서 원격 SMB 공유를 글로벌 매핑합니다. $creds = Get-Credential New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G: 이 명령은 원격 SMB 서버 인증을 위해 자격 증명을 사용합니다. 그런 다음 원격 공유 경로를 G: 드라이브 문자에 매핑합니다(사용 가능한 다른 모든 드라이브 문자일 수 있음). 이 컨테이너 호스트에서 생성된 컨테이너는 이제 자신의 데이터 볼륨을 G: 드라이브에 있는 경로에 매핑할 수 있습니다.

> 참고: 컨테이너에 대해 SMB 글로벌 매핑을 사용할 때 컨테이너 호스트의 모든 사용자가 원격 공유에 액세스할 수 있습니다. 컨테이너 호스트에서 실행되는 모든 응용 프로그램 또한 매핑된 원격 공유에 액세스할 수 있습니다.

2. 글로벌 마운트된 SMB 공유에 매핑된 데이터 볼륨이 있는 컨테이너 만들기 docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

컨테이너 내에서 G:\AppData1은 원격 공유의 "ContainerData" 디렉터리에 매핑됩니다. 글로벌 매핑된 원격 공유에 저장된 모든 데이터는 컨테이너 내에 있는 응용 프로그램에서 사용할 수 있습니다. 여러 컨테이너가 동일한 명령을 사용하여 이 공유된 데이터에 대한 읽기/쓰기 액세스를 가져올 수 있습니다.

이 SMB 글로벌 매핑 지원은 다음을 포함하여 호환되는 모든 SMB 서버 위에서 작업할 수 있는 SMB 클라이언트 측 기능입니다.

- 저장소 공간 다이렉트(S2D) 또는 기존 SAN 위의 스케일 아웃 파일 서버
- Azure Files(SMB 공유)
- 기존 파일 서버
- SMB 프로토콜(예: NAS 어플라이언스) 타사 구현

> [!NOTE]
> SMB 글로벌 매핑은 Windows Server 버전 1709에서 DFS, DFSN, DFSR 공유를 지원하지 않습니다.

### <a name="named-volumes"></a>명명된 볼륨

명명된 볼륨을 사용하여 이름별 볼륨을 만들고, 컨테이너에 할당하고, 나중에 같은 이름으로 다시 사용할 수 있습니다. 이를 만들 실제 경로를 추적할 필요가 없으며 이름만 기억하면 됩니다. Windows의 Docker 엔진에는 로컬 컴퓨터에 볼륨을 만들 수 있는 명명된 볼륨 플러그 인이 기본 제공됩니다. 여러 대의 컴퓨터에서 명명된 볼륨을 사용하려면 추가 플러그 인이 필요합니다.

예시 단계:

1. `docker volume create unwound` - 'unwound'라는 볼륨 만들기
2. `docker run -v unwound:c:\data microsoft/windowsservercore` - c:\data에 매핑된 볼륨이 있는 컨테이너 시작
3. 컨테이너의 c:\data에 일부 파일을 쓴 다음 컨테이너 중지
4. `docker run -v unwound:c:\data microsoft/windowsservercore` - 새 컨테이너 시작
5. 새 컨테이너에서 `dir c:\data` 실행 - 파일이 여전히 있음
