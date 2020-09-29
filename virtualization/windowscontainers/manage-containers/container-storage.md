---
title: 컨테이너 스토리지 개요
description: Windows Server 컨테이너가 호스트 및 기타 저장소 유형을 사용하는 방법
keywords: 컨테이너, 볼륨, 스토리지, 마운트, 바인딩 마운트
author: cwilhit
ms.author: jgerend
ms.topic: overview
ms.openlocfilehash: e5cc4770cfa014ca307db1bb7ec4a3bb81c2dd32
ms.sourcegitcommit: 160405a16d127892b6e2897efa95680f29f0496a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/22/2020
ms.locfileid: "90990926"
---
# <a name="container-storage-overview"></a>컨테이너 스토리지 개요

<!-- Great diagram would be great! -->

이 토픽에서는 컨테이너가 Windows에서 스토리지를 사용하는 다양한 방법에 대해 간략하게 설명합니다. 컨테이너는 스토리지와 관련하여 가상 머신과 다른 방식으로 작동합니다. 기본적으로 컨테이너는 컨테이너 내에서 실행되는 앱이 호스트의 파일 시스템 전체에 상태를 기록하지 않도록 빌드됩니다. 컨테이너는 기본적으로 "스크래치" 공간을 사용하지만, Windows 역시 스토리지를 유지할 수 있는 수단을 제공합니다.

## <a name="scratch-space"></a>스크래치 공간

Windows 컨테이너는 기본적으로 사용 후 삭제 스토리지를 사용합니다. 모든 컨테이너 I/O는 "스크래치 공간"에서 발생하며 각 컨테이너는 자체 스크래치를 갖습니다. 파일 생성 및 파일 쓰기는 스크래치 공간에서 캡처되며 호스트로 이스케이프되지 않습니다. 컨테이너 인스턴스가 중지되면 스크래치 공간에서 발생한 모든 변경 사항이 제거됩니다. 새 컨테이너 인스턴스가 시작되면 해당 인스턴스에 대한 새 스크래치 공간이 제공됩니다.

## <a name="layer-storage"></a>계층 저장소

[컨테이너 개요](../about/index.md)에서 설명했듯이, 컨테이너 이미지는 일련의 계층으로 표현되는 파일 번들입니다. 계층 스토리지는 컨테이너에 내장된 모든 파일입니다. 사용자가 `docker pull`한 다음 해당 컨테이너를 `docker run`할 때마다 동일합니다.

### <a name="where-layers-are-stored-and-how-to-change-it"></a>계층이 저장되는 위치 및 이를 변경하는 방법

기본 설치에서 계층은 `C:\ProgramData\docker`에 저장되며 "image"와 "windowsfilter" 디렉터리에 분산됩니다. [Windows의 Docker 엔진](../manage-docker/configure-docker-daemon.md) 설명서에서와 같이 `docker-root` 구성을 사용하여 계층이 저장되는 위치를 변경할 수 있습니다.

> [!NOTE]
> 계층 스토리지에 대해 NTFS만 지원됩니다. ReFS는 지원되지 않습니다.

계층 디렉터리에서 어떠한 파일도 수정해서는 안 됩니다. 다음과 같은 명령을 사용하여 신중하게 관리해야 합니다.

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker load](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>계층 스토리지에서 지원되는 작업

실행 중인 컨테이너는 트랜잭션을 제외하고 대부분의 NTFS 작업을 사용할 수 있습니다. 이에는 ACL 설정이 포함되며 컨테이너 내에 있는 모든 ACL을 확인합니다. 컨테이너 내에서 여러 사용자로 프로세스를 실행하려면 `RUN net user /create ...`로 `Dockerfile` 내에 사용자를 만들고, ACL 파일을 설정한 다음 프로세스를 구성하면 [Dockerfile USER 지시문](https://docs.docker.com/engine/reference/builder/#user)을 사용하여 해당 사용자로 실행할 수 있습니다.

## <a name="persistent-storage"></a>영구 스토리지

Windows 컨테이너는 바인드 탑재 및 볼륨을 통해 영구 스토리지를 제공하는 메커니즘을 지원합니다. 자세한 내용은 [컨테이너의 영구 스토리지](./persistent-storage.md)를 참조하세요.

## <a name="storage-limits"></a>스토리지 제한

Windows 애플리케이션의 일반적인 패턴은 새 파일을 설치하거나 만들기 전에 또는 임시 파일을 정리하기 위한 트리거로서 사용 가능한 디스크 공간의 크기를 쿼리하는 것입니다.  애플리케이션 호환성의 극대화를 목표로 Windows 컨테이너에 있는 C: 드라이브는 20GB의 사용 가능한 가상 크기를 나타냅니다.

이 기본값을 재정의하고 사용 가능한 공간을 더 작은 값으로 또는 더 큰 값으로 구성하려는 사용자도 있을 것입니다. 이 경우 “스토리지 옵션” 구성에서 “크기” 옵션을 사용하면 됩니다.

### <a name="examples"></a>예

명령줄: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

또는 다음과 같이 docker 구성 파일을 직접 변경할 수도 있습니다.

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> 이 방법은 docker 빌드에도 작동합니다. Docker 구성 파일을 수정하는 것에 대한 자세한 내용은 [Docker 구성](../manage-docker/configure-docker-daemon.md#configure-docker-with-a-configuration-file) 설명서를 참조하세요.