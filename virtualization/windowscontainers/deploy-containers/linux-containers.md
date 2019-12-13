---
title: Windows의 Linux 컨테이너
description: Hyper-v를 사용 하 여 네이티브 인 것 처럼 Windows에서 Linux 컨테이너를 실행할 수 있는 다양 한 방법에 대해 알아봅니다.
keywords: LCOW, linux 컨테이너, docker, 컨테이너
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910573"
---
# <a name="linux-containers-on-windows"></a>Windows의 Linux 컨테이너

Linux 컨테이너는 전반적인 컨테이너 에코 시스템의 매우 큰 비율을 구성 하며 개발자 환경 및 프로덕션 환경 모두에 대 한 기본입니다.  컨테이너는 컨테이너 호스트와 커널을 공유 하므로 Windows에서 직접 Linux 컨테이너를 실행 하는 것은[*](linux-containers.md#other-options-we-considered)옵션이 아닙니다.  여기서는 가상화가 그림에 제공 됩니다.

이제 Windows용 Docker 및 Hyper-v를 사용 하 여 Linux 컨테이너를 실행 하는 두 가지 방법이 있습니다.

- 전체 Linux VM에서 Linux 컨테이너를 실행 합니다 .이는 일반적으로 일반적으로 수행 하는 Docker입니다.
- [Hyper-v 격리](../manage-containers/hyperv-container.md) 를 사용 하 여 Linux 컨테이너 실행 (LCOW)-Windows용 Docker의 새로운 옵션입니다.

이 문서에서는 각 방법의 작동 방식에 대해 간략하게 설명 하 고 진행 중인 솔루션 및 공유 작업을 선택 하는 시기에 대 한 지침을 제공 합니다.

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM의 Linux 컨테이너

Linux VM에서 Linux 컨테이너를 실행 하려면 [Docker의 시작 가이드](https://docs.docker.com/docker-for-windows/)에 있는 지침을 따르세요.

Docker는 Hyper-v에서 실행 되는 [LinuxKit](https://github.com/linuxkit/linuxkit) 기반 가상 컴퓨터를 사용 하 여 2016 (Windows에서 hyper-v 격리 또는 Linux 컨테이너를 사용할 수 있음)에서 처음 릴리스된 이후 windows 데스크톱에서 linux 컨테이너를 실행할 수 있었습니다.

이 모델에서 Docker 클라이언트는 Windows 데스크톱에서 실행 되지만 Linux VM에서 Docker 디먼을 호출 합니다.

![Moby VM을 컨테이너 호스트로](media/MobyVM.png)

이 모델에서 모든 Linux 컨테이너는 단일 Linux 기반 컨테이너 호스트 및 모든 Linux 컨테이너를 공유 합니다.

* Windows 호스트를 제외 하 고 서로 다른 및 Moby VM과 커널을 공유 합니다.
* Linux VM에서 실행 되 고 있으므로 linux에서 실행 되는 Linux 컨테이너와 일관 된 저장소 및 네트워킹 속성이 있습니다.

또한 Linux 컨테이너 호스트 (Moby VM)는 Docker 데몬 및 모든 Docker 데몬 종속성을 실행 해야 함을 의미 합니다.

Moby VM을 사용 하 여 실행 하 고 있는지 확인 하려면 Hyper-v 관리자 UI를 사용 하거나 관리자 권한 PowerShell 창에서 `Get-VM`를 실행 하 여 Moby VM에 대 한 Hyper-v 관리자를 확인 합니다.

## <a name="linux-containers-with-hyper-v-isolation"></a>Hyper-v 격리를 사용 하는 Linux 컨테이너

Windows에서 Linux 컨테이너 (LCOW)를 시도 하려면 [windows 10에서](../quick-start/quick-start-windows-10-linux.md)linux 컨테이너의 linux 컨테이너 지침을 따릅니다.

Hyper-v 격리를 사용 하는 linux 컨테이너는 컨테이너를 실행 하는 데 충분 한 OS를 사용 하 여 최적화 된 Linux VM에서 각 Linux 컨테이너를 실행 합니다. Moby VM 방식과 달리 각 Linux 컨테이너에는 고유한 커널 및 자체 VM 샌드박스가 있습니다. 또한 Windows의 Docker에서 직접 관리 됩니다.

![Hyper-v 격리를 사용 하는 Linux 컨테이너 (LCOW)](media/lcow-approach.png)

LCOW 모델 컨테이너 관리는 Windows에 유지 되 고 각 LCOW 관리는 GRPC 및 containerd를 통해 수행 되므로 Moby VM 접근 방식 및 LCOW 간에 컨테이너 관리가 어떻게 달라 지는 지 자세히 살펴보세요.  즉, LCOW에 사용 되는 Linux 배포판 컨테이너는 재고를 훨씬 더 줄일 수 있습니다.  현재는 최적화 된 배포판 컨테이너에 사용 되는 LinuxKit를 사용 하 고 있지만 Kata와 같은 다른 프로젝트는 비슷한 매우 조정 된 Linux 배포판 (Linux 일반)도 구축 하 고 있습니다.

각 LCOW에 대해 자세히 살펴봅니다.

![LCOW 아키텍처](media/lcow.png)

LCOW를 실행 하 고 있는지 확인 하려면 `C:\Program Files\Linux Containers`으로 이동 합니다. Docker가 LCOW를 사용 하도록 구성 된 경우 Hyper-v 격리에서 실행 되는 각 컨테이너에서 실행 되는 최소 LinuxKit 배포판를 포함 하는 몇 가지 파일이 여기에 포함 됩니다.  최적화 된 VM 구성 요소는 Moby VM의 LinuxKit 이미지 보다 훨씬 작은 100 MB 미만입니다.

### <a name="work-in-progress"></a>작업 진행 중

LCOW가 활성 개발 중입니다. [GitHub](https://github.com/moby/moby/issues/33850) 의 Moby 프로젝트에서 진행 중인 진행 상황 추적

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

이러한 응용 프로그램은 모두 볼륨 매핑이 필요 하며 시작 되거나 올바르게 실행 되지 않습니다.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>추가 정보

[LCOW를 설명 하는 Docker 블로그](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 컨테이너 비디오](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-커널 plus 빌드 지침](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Moby VM 및 LCOW를 사용 하는 경우

### <a name="when-to-use-moby-vm"></a>Moby VM을 사용 하는 경우

지금은 Linux 컨테이너를 실행 하는 Moby VM 메서드를 다음과 같은 사용자에 게 권장 합니다.

- 안정적인 컨테이너 환경을 원합니다.  Windows용 Docker 기본값입니다.
- Windows 또는 Linux 컨테이너를 실행 하지만 동시에는 거의 발생 하지 않습니다.
- Linux 컨테이너 간에 복잡 하거나 사용자 지정 네트워킹 요구 사항이 있습니다.
- Linux 컨테이너 간에 커널 격리 (Hyper-v 격리)가 필요 하지 않습니다.

### <a name="when-to-use-lcow"></a>LCOW를 사용 하는 경우

지금은 다음과 같은 사람들에 게 LCOW 하는 것이 좋습니다.

- 최신 기술을 테스트 하려고 합니다.
- Windows 및 Linux 컨테이너를 동시에 실행 합니다.
- Linux 컨테이너 간에 커널 격리 (Hyper-v 격리)가 필요 합니다.

## <a name="other-options-we-considered"></a>고려 하는 기타 옵션

Windows에서 Linux 컨테이너를 실행 하는 방법을 살펴보면 WSL로 간주 합니다. 궁극적으로 Windows의 Linux 컨테이너가 linux의 Linux 컨테이너와 일치 하도록 가상화 기반 접근 방식을 선택 했습니다. Hyper-v를 사용 하면 LCOW 더 안전 합니다. Microsoft는 나중에 다시 평가할 수도 있지만 지금은 LCOW에서 Hyper-v를 계속 사용 합니다.

생각이 있으면 GitHub 또는 UserVoice를 통해 의견을 보내 주시기 바랍니다.  특히 보려는 특정 환경에 대 한 의견을 보내주셔서 감사 합니다.
