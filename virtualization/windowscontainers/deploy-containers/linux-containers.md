---
title: Windows의 Linux 컨테이너
description: Hyper-v를 사용 하 여 네이티브 경우 처럼 Windows에서 Linux 컨테이너를 실행 하는 다양 한 방법을 알아봅니다.
keywords: LCOW, linux 컨테이너, docker, 컨테이너
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 477c6079d6e90a206386d1810bdb1449e087a4be
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948092"
---
# <a name="linux-containers-on-windows"></a>Windows의 Linux 컨테이너

Linux 컨테이너 전체 컨테이너 생태계의 거 대 한 %를 차지 하 고는 개발자 환경과 프로덕션 환경에 기본적인 요소입니다.  하지만 컨테이너를 컨테이너 호스트와 커널을 공유 하기 때문 Windows에서 직접 Linux 컨테이너를 실행할 수 없는[*](linux-containers.md#other-options-we-considered).  가상화 그림에 제공 되는 위치입니다.

지금은 Windows 용 Docker와 하이퍼 사항 Linux 컨테이너를 실행 하는 방법은 두 가지가 있습니다

1. -전체 Linux VM에서 Linux 컨테이너를 실행할 어떤 Docker 일반적으로 전환한입니다.
1. Windows 용 Docker에서에서 새로운 옵션입니다 (LCOW)- [Hyper-v 격리](../manage-containers/hyperv-container.md) 를 사용 하 여 Linux 컨테이너를 실행 합니다.

이 문서에서는 각 방법을 작동, 어떤 솔루션을 선택 하는 경우에 대 한 지침을 제공 및 진행 중인 작업을 공유 하는 방법을 설명 합니다.

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM에서 Linux 컨테이너

Linux VM에서 Linux 컨테이너를 실행 하려면 [Docker의 get 시작 가이드](https://docs.docker.com/docker-for-windows/)의 지침을 따릅니다.

Docker Windows에서 Linux 컨테이너를 실행할 수 있었던 처음 출시 이후 데스크톱 2016에서 (Hyper-v 격리 또는 LCOW 결말이) 전에 [LinuxKit](https://github.com/linuxkit/linuxkit) 를 사용 하 여 기반 Hyper-v에서 실행 되는 가상 컴퓨터.

이 모델에서는 Docker 클라이언트는 Linux VM에 Docker 디먼에 호출 하지만 Windows 바탕 화면에서 실행 됩니다.

![컨테이너 호스트와 Moby VM](media/MobyVM.png)

이 모델에서는 모든 Linux 컨테이너 Linux 기반 단일 컨테이너 호스트 및 모든 Linux 컨테이너를 공유:

* Windows 호스트 하지 하지만 서로 및 Moby VM을 사용 하 여 커널을 공유 합니다.
* 일관 된 저장소 있고 속성 (Linux VM에서 실행 중인) 이후 Linux에서 실행 되는 Linux 컨테이너를 사용 하 여 네트워킹 합니다.

또한 Linux 컨테이너 호스트 (Moby VM)를 Docker 디먼 및 모든 Docker 디먼 종속성을 실행 해야 합니다. 의미 합니다.

Moby VM을 사용 하 여 실행 중인 경우를 보려면 Hyper-v 관리자를 실행 하 여 또는 Hyper-v 관리자 UI를 사용 하 여 Moby VM에 대 한 확인 `Get-VM` 관리자 권한 PowerShell 창에 있습니다.

## <a name="linux-containers-with-hyper-v-isolation"></a>Hyper-v 격리를 사용 하 여 Linux 컨테이너

[이 get 시작 가이드](../quick-start/quick-start-windows-10.md) 의 Linux 컨테이너 지침에 따라 LCOW를 실행 하려면

Hyper-v 격리를 사용 하 여 Linux 컨테이너는 컨테이너를 실행 하려면 충분 한 OS 사용 하 여 최적화 된 Linux VM에서 각 Linux 컨테이너 (LCOW)를 실행 합니다.  Moby VM 접근 방식을 달리 각 LCOW 커널 자체 있으며 자체 VM 샌드박스 것입니다.  하 게도 관리 Windows에서 Docker에서 직접 합니다.

![Hyper-v 격리 (LCOW)를 사용 하 여 Linux 컨테이너](media/lcow-approach.png)

컨테이너 관리 Moby VM 방법과 LCOW 간의 차이점에 대해 자세히 수행에서 LCOW 모델 컨테이너 관리는 Windows에 유지 하 고 GRPC 및 containerd 통해 발생 하는 각 LCOW 관리.  이 즉, Linux 배포판 컨테이너 LCOW 사용할 훨씬 더 작은 인벤토리를 가질 수 있습니다.  오른쪽 이제 사용 하 고 LinuxKit 최적화 된 배포판 컨테이너를 사용 하지만 Kata와 같은 다른 프로젝트에 유사한 높은 조정 Linux 배포판 (지우기 Linux)도 빌드하는 합니다.

각 LCOW에 대해 자세히 다음과 같습니다.

![LCOW 아키텍처](media/lcow.png)

이동 LCOW 실행 중인 경우를 보려면 `C:\Program Files\Linux Containers`.  Docker LCOW를 사용 하도록 구성 된, 여기서 각 Hyper-v 컨테이너에서 실행 되는 최소 LinuxKit 배포판 포함 된 몇 가지 파일 됩니다.  최적화 된 VM 구성 요소 100MB 미만, Moby VM에서 LinuxKit 이미지 보다 훨씬 더 작은 인지 확인 합니다.

### <a name="work-in-progress"></a>작업 진행 중

LCOW는 적극적으로 개발 중입니다.  [GitHub](https://github.com/moby/moby/issues/33850) 에서 Moby 프로젝트의 현재 진행 추적

#### <a name="bind-mounts"></a>마운트 바인딩

`docker run -v ...`으로 마운팅 볼륨을 바인딩하면 Windows NTFS 파일 시스템에서 파일을 저장하므로 POSIX 작업을 하려면 변환이 필요합니다. 일부 파일 시스템 작동이 현재 부분적으로 구현되거나 구현되지 않습니다. 이에 따라 일부 앱이 호환되지 않을 수 있습니다.

바인딩되어 탑재된 볼륨에 대해 이러한 작업은 현재 작동되지 않습니다.

* MkNod
* XAttrWalk
* XAttrCreate
* Lock
* Getlock
* Auth
* Flush
* INotify

완전히 구현되지 않는 다음과 같은 사항도 있습니다.

* GetAttr – Nlink 개수가 항상 2개로 보고됩니다.
* Open – ReadWrite, WriteOnly, 및 ReadOnly 플래그만 구현됩니다.

이러한 응용 프로그램 모두는 볼륨 매핑이 요구를 시작 하거나 올바르게 실행 됩니다.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>추가 정보

[LCOW를 설명 하는 docker 블로그](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 컨테이너 비디오](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW 커널 및 빌드 지침](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Moby VM vs LCOW를 사용 하는 경우

### <a name="when-to-use-moby-vm"></a>Moby VM을 사용 하는 경우

오른쪽 이제 권장 하는 사람들이 Linux 컨테이너를 실행 하는 Moby VM 방법:

1. 안정적인 컨테이너 환경이 합니다.  Windows 용 Docker 기본값입니다.
1. Windows 또는 Linux 컨테이너 거의 둘 다 동시에 실행 합니다.
1. 복잡 한 사용자 지정 네트워킹 요구 사항 Linux 컨테이너 간에 또는 합니다.
1. Linux 컨테이너 간에 커널 격리 (Hyper-v 격리) 필요 하지 마십시오.

### <a name="when-to-use-lcow"></a>LCOW를 사용 하는 경우

오른쪽 이제 권장 하는 사람들이 LCOW:

1. 테스트는 최신 기술을 하려고 합니다.
1. 이와 동시에 Windows 및 Linux 컨테이너를 실행 합니다.
1. Linux 컨테이너 간에 커널 격리 (Hyper-v 격리) 필요 합니다.

## <a name="other-options-we-considered"></a>다른 옵션을 처리 하는 것

Windows에서 Linux 컨테이너를 실행 하는 방법을 찾을 것, WSL 것으로 간주 했습니다.  궁극적으로 Windows에서 Linux 컨테이너는 Linux에서 Linux 컨테이너와 일치 하는 가상화 기반 방법 선택 했습니다.  Hyper-v를 사용 하 여 하기도 LCOW 더 안전 합니다.  나중에 다시 평가 수 있지만 지금은 LCOW는 Hyper-v를 사용 하 여 계속 합니다.

생각 하는 경우 github 또는 UserVoice 통해 피드백을 보내 합니다.  특히 참조 하려는 특정 환경에 대 한 피드백 해 주셔서 감사 합니다.