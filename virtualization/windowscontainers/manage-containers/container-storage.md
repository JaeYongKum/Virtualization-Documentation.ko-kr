---
title: 컨테이너 저장소 개요
description: Windows Server 컨테이너가 호스트 및 기타 저장소 유형을 사용하는 방법
keywords: 컨테이너, 볼륨, 저장소, 마운트, 바인딩 마운트
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910273"
---
# <a name="container-storage-overview"></a>컨테이너 저장소 개요

<!-- Great diagram would be great! -->

이 항목에서는 컨테이너에서 Windows의 저장소를 사용 하는 다양 한 방법에 대 한 개요를 제공 합니다. 컨테이너는 저장소에 제공 되는 가상 컴퓨터와 다르게 동작 합니다. 기본적으로 컨테이너는 호스트에서 실행 되는 앱이 호스트의 파일 시스템에 대 한 상태를 기록 하지 않도록 빌드됩니다. 컨테이너는 기본적으로 "스크래치" 공간을 사용 하지만 Windows는 저장소를 유지할 수 있는 수단을 제공 합니다.

## <a name="scratch-space"></a>스크래치 공간

Windows 컨테이너는 기본적으로 사용 후 삭제 저장소를 사용 합니다. 모든 컨테이너 i/o는 "스크래치 공간"에서 발생 하며 각 컨테이너는 자체 스크래치를 가져옵니다. 파일 생성 및 파일 쓰기는 스크래치 공간에서 캡처되고 호스트로 이스케이프 되지 않습니다. 컨테이너 인스턴스가 중지 되 면 스크래치 공간에서 발생 한 모든 변경 사항이 throw 됩니다. 새 컨테이너 인스턴스가 시작 되 면 인스턴스에 대해 새로운 스크래치 공간이 제공 됩니다.

## <a name="layer-storage"></a>계층 저장소

컨테이너 [개요](../about/index.md)에 설명 된 대로 컨테이너 이미지는 일련의 계층으로 표현 되는 파일의 번들입니다. 계층 저장소는 컨테이너에 기본 제공 되는 모든 파일입니다. 사용자가 `docker pull`한 다음 해당 컨테이너를 `docker run`할 때마다 동일합니다.

### <a name="where-layers-are-stored-and-how-to-change-it"></a>계층이 저장되는 위치 및 이를 변경하는 방법

기본 설치에서 계층은 `C:\ProgramData\docker`에 저장되며 "image"와 "windowsfilter" 디렉터리에 분산됩니다. [Windows의 Docker 엔진](../manage-docker/configure-docker-daemon.md) 설명서에서와 같이 `docker-root` 구성을 사용하여 계층이 저장되는 위치를 변경할 수 있습니다.

> [!NOTE]
> 계층 저장소에 대해 NTFS만 지원됩니다. ReFS는 지원되지 않습니다.

계층 디렉터리에서 어떠한 파일도 수정해서는 안 됩니다. 다음과 같은 명령을 사용하여 신중하게 관리해야 합니다.

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker 로드](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>계층 저장소에서 지원되는 작업

실행 중인 컨테이너는 트랜잭션을 제외하고 대부분의 NTFS 작업을 사용할 수 있습니다. 이에는 ACL 설정이 포함되며 컨테이너 내에 있는 모든 ACL을 확인합니다. 컨테이너 내에서 여러 사용자로 프로세스를 실행하려면 `RUN net user /create ...`로 `Dockerfile` 내에 사용자를 만들고, ACL 파일을 설정한 다음 프로세스를 구성하면 [Dockerfile USER 지시문](https://docs.docker.com/engine/reference/builder/#user)을 사용하여 해당 사용자로 실행할 수 있습니다.

## <a name="persistent-storage"></a>영구 저장소

Windows 컨테이너는 바인드 탑재 및 볼륨을 통해 영구 저장소를 제공 하기 위한 메커니즘을 지원 합니다. 자세히 알아보려면 [컨테이너의 영구 저장소](./persistent-storage.md)를 참조 하세요.

## <a name="storage-limits"></a>저장소 제한

Windows 응용 프로그램의 일반적인 패턴은 새 파일을 설치하거나 만들기 전에 또는 임시 파일을 정리하기 위한 트리거로서 사용 가능한 디스크 공간의 크기를 쿼리하는 것입니다.  응용 프로그램 호환성을 최대화 하는 목표로 Windows 컨테이너의 C: 드라이브는 20GB의 가상 여유 크기를 나타냅니다.

일부 사용자는이 기본값을 재정의 하 고 사용 가능한 공간을 더 작거나 더 큰 값으로 구성할 수 있습니다. "저장소 옵트인" 구성 내에서 "크기" 옵션을 사용할 수 있습니다.

### <a name="examples"></a>예

명령줄: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

또는 docker 구성 파일을 직접 변경할 수 있습니다.

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> 이 메서드는 docker 빌드에도 작동 합니다. Docker 구성 파일을 수정하는 것에 대한 자세한 내용은 [Docker 구성](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) 설명서를 참조하세요.
