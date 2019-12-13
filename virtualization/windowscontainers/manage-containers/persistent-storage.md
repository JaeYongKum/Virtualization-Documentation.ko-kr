---
title: 컨테이너의 영구 저장소
description: Windows 컨테이너에서 영구적으로 저장소를 사용할 수 있는 방법
keywords: 컨테이너, 볼륨, 저장소, 마운트, 바인딩 마운트
author: cwilhit
ms.openlocfilehash: 945a78d4ecb9c96da4de8f7246f84b6b444dd5b5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909673"
---
# <a name="persistent-storage-in-containers"></a>컨테이너의 영구 저장소

<!-- Great diagram would be great! -->

앱이 컨테이너의 데이터를 유지할 수 있거나 컨테이너 빌드 시간에 포함 되지 않은 컨테이너에 파일을 표시 하려는 경우가 있을 수 있습니다. 영구 저장소는 다음과 같은 몇 가지 방법으로 컨테이너에 제공할 수 있습니다.

- 마운트 바인딩
- 명명된 볼륨

Docker에는 [볼륨을 사용하는](https://docs.docker.com/engine/admin/volumes/volumes/) 방법에 대한 훌륭한 개요가 들어 있으니 먼저 읽는 것이 좋습니다. 이 페이지의 나머지 부분에서는 Linux와 Windows의 차이점에 초점을 맞추며 Windows의 예를 제공합니다.

## <a name="bind-mounts"></a>바인딩 마운트

[바인딩 마운트](https://docs.docker.com/engine/admin/volumes/bind-mounts/)를 통해 컨테이너가 호스트와 디렉터리를 공유할 수 있습니다. 이는 컨테이너를 다시 시작하거나 여러 컨테이서와 공유하고자 할 때 사용할 수 있는 로컬 컴퓨터에 파일을 저장할 장소를 원하는 경우 유용합니다. 여러 대의 컴퓨터에서 동일한 파일에 액세스하여 컨테이너를 실행하려는 경우 명명된 볼륨 또는 SMB 마운트를 대신 사용해야 합니다.

### <a name="permissions"></a>권한

바인딩 마운트에 사용되는 권한 모델은 사용자 컨테이너에 대한 격리 수준에 따라 달라집니다.

**Hyper-v 격리** 를 사용 하는 컨테이너는 단순 읽기 전용 또는 읽기/쓰기 권한 모델을 사용 합니다. `LocalSystem` 계정을 사용하는 호스트에서 파일에 액세스합니다. 컨테이너에서 액세스가 거부된 경우 `LocalSystem`이 호스트의 디렉터리에 액세스할 수 있는지 확인합니다. 읽기 전용 플래그를 사용하면 컨테이너 내에서 볼륨을 변경하더라도 호스트의 디렉터리에 표시되거나 유지되지 않습니다.

**프로세스 격리** 를 사용 하는 Windows 컨테이너는 컨테이너 내에서 프로세스 id를 사용 하 여 데이터에 액세스 하기 때문에 약간 다릅니다. 즉, 파일 acl이 적용 됩니다. 컨테이너(Windows Server Core의 "ContainerAdministrator" 및 Nano Server 컨테이너의 "ContainerUser", 기본값)에서 실행되는 프로세스의 ID가 `LocalSystem` 대신 마운트된 볼륨의 파일 및 디렉터리에 액세스하는 데 사용되며 데이터를 사용하기 위해 액세스 권한을 부여받아야 합니다.

이러한 id는 파일이 저장 된 호스트가 아니라 컨테이너의 컨텍스트 내에만 존재 하므로, 컨테이너에 대 한 액세스 권한을 부여 하도록 Acl을 구성할 때 `Authenticated Users`와 같은 잘 알려진 보안 그룹을 사용 해야 합니다.

> [!WARNING]
> 신뢰할 수 없는 컨테이너에 `C:\`와 같이 중요한 디렉터리를 바인딩 마운트하지 마십시오. 이렇게 하면 일반적으로 액세스할 수 없으며 보안 위반을 일으킬 수 있는 호스트에서 파일을 변경할 수 있습니다.

사용 예제:

- 읽기 전용 액세스를 위한 `docker run -v c:\ContainerData:c:\data:RO`
- 읽기/쓰기 액세스에 대 한 `docker run -v c:\ContainerData:c:\data:RW`
- 읽기/쓰기 액세스에 대 한 `docker run -v c:\ContainerData:c:\data` (기본값)

### <a name="symlinks"></a>symlink

symlink는 컨테이너에서 확인됩니다. symlink인 컨테이너 또는 symlink를 포함하는 컨테이너에 대한 호스트 경로를 바인딩 마운트하면 컨테이너는 이에 액세스할 수 없게 됩니다.

### <a name="smb-mounts"></a>SMB 마운트

Windows Server 버전 1709 이상에서 "SMB 글로벌 매핑" 이라는 기능을 사용 하면 호스트에 SMB 공유를 탑재 한 다음 해당 공유의 디렉터리를 컨테이너에 전달할 수 있습니다. 컨테이너는 특정 서버, 공유, 사용자 이름 또는 함호로 구성할 필요가 없습니다. 모두 호스트에서 대신 처리됩니다. 컨테이너는 로컬 저장소에서처럼 동일하게 작동합니다.

#### <a name="configuration-steps"></a>구성 단계

1. 컨테이너 호스트에서 원격 SMB 공유를 전역적으로 매핑합니다.
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    이 명령은 자격 증명을 사용 하 여 원격 SMB 서버에 인증 합니다. 그런 다음 원격 공유 경로를 G: 드라이브 문자에 매핑합니다(사용 가능한 다른 모든 드라이브 문자일 수 있음). 이 컨테이너 호스트에서 생성된 컨테이너는 이제 자신의 데이터 볼륨을 G: 드라이브에 있는 경로에 매핑할 수 있습니다.

    > [!NOTE]
    > 컨테이너에 대해 SMB 전역 매핑을 사용 하는 경우 컨테이너 호스트의 모든 사용자가 원격 공유에 액세스할 수 있습니다. 컨테이너 호스트에서 실행되는 모든 응용 프로그램 또한 매핑된 원격 공유에 액세스할 수 있습니다.

2. 글로벌 마운트된 SMB 공유에 매핑된 데이터 볼륨이 있는 컨테이너 만들기 docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    컨테이너 내에서 G:\AppData1은 원격 공유의 "ContainerData" 디렉터리에 매핑됩니다. 글로벌 매핑된 원격 공유에 저장된 모든 데이터는 컨테이너 내에 있는 응용 프로그램에서 사용할 수 있습니다. 여러 컨테이너가 동일한 명령을 사용하여 이 공유된 데이터에 대한 읽기/쓰기 액세스를 가져올 수 있습니다.

이 SMB 글로벌 매핑 지원은 다음을 포함하여 호환되는 모든 SMB 서버 위에서 작업할 수 있는 SMB 클라이언트 측 기능입니다.

- 저장소 공간 다이렉트(S2D) 또는 기존 SAN 위의 스케일 아웃 파일 서버
- Azure Files(SMB 공유)
- 기존 파일 서버
- SMB 프로토콜(예: NAS 어플라이언스) 타사 구현

> [!NOTE]
> SMB 글로벌 매핑은 Windows Server 버전 1709에서 DFS, DFSN, DFSR 공유를 지원하지 않습니다.

## <a name="named-volumes"></a>명명된 볼륨

명명된 볼륨을 사용하여 이름별 볼륨을 만들고, 컨테이너에 할당하고, 나중에 같은 이름으로 다시 사용할 수 있습니다. 이를 만들 실제 경로를 추적할 필요가 없으며 이름만 기억하면 됩니다. Windows의 Docker 엔진에는 로컬 컴퓨터에 볼륨을 만들 수 있는 명명된 볼륨 플러그 인이 기본 제공됩니다. 여러 대의 컴퓨터에서 명명된 볼륨을 사용하려면 추가 플러그 인이 필요합니다.

예시 단계:

1. `docker volume create unwound`-' 해제 ' 라는 볼륨을 만듭니다.
2. c:\data에 매핑된 볼륨이 있는 컨테이너를 `docker run -v unwound:c:\data microsoft/windowsservercore` 시작 합니다.
3. 컨테이너의 c:\data에 일부 파일을 쓴 다음 컨테이너 중지
4. `docker run -v unwound:c:\data microsoft/windowsservercore`-새 컨테이너를 시작 합니다.
5. 새 컨테이너에서 `dir c:\data` 실행 - 파일이 여전히 있음

> [!NOTE]
> Windows Server는 대상 경로 이름 (컨테이너 내의 경로)을 소문자로 변환 합니다. `-v unwound:c:\MyData`또는 Linux 컨테이너의 `-v unwound:/app/MyData`는 `c:\mydata`의 컨테이너 내에 디렉터리를 생성 하거나 Linux 컨테이너의 `/app/mydata`를 매핑하고 (존재 하지 않는 경우 생성) 합니다.
