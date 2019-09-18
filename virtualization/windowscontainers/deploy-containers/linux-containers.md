---
title: Windows의 Linux 컨테이너
description: Windows에서 Hyper-v를 사용 하 여 기본으로 Linux 컨테이너를 실행할 수 있는 다양 한 방법에 대해 알아봅니다.
keywords: LCOW, linux 컨테이너, docker, 컨테이너
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 9100d2218c160bbe9fbf24f3524c8ff5e3dd826c
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/18/2019
ms.locfileid: "10135326"
---
# <a name="linux-containers-on-windows"></a>Windows의 Linux 컨테이너

Linux 컨테이너는 전체 컨테이너 에코 시스템의 많은 비율을 구성 하며 개발자 환경 및 프로덕션 환경 모두에 대 한 기본입니다.  컨테이너는 컨테이너 호스트와 커널을 공유 하므로 Windows에서 직접 Linux 컨테이너를 실행 하는 것은 옵션이[*](linux-containers.md#other-options-we-considered)아닙니다.  이는 가상화가 그림에 제공 되는 위치입니다.

이제 Windows 및 Hyper-v 용 Docker를 사용 하 여 Linux 컨테이너를 실행 하는 두 가지 방법이 있습니다.

- 전체 Linux VM에서 Linux 컨테이너를 실행 하는 것은 일반적으로 현재는 Docker입니다.
- Linux 컨테이너를 [hyper-v 격리](../manage-containers/hyperv-container.md) (LCOW)로 실행-Windows 용 Docker의 새로운 옵션입니다.

이 문서에서는 각 접근 방식의 작동 방식에 대해 설명 하 고 진행 중인 솔루션 및 공유를 선택 하는 경우에 대 한 지침을 제공 합니다.

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM의 Linux 컨테이너

Linux VM에서 Linux 컨테이너를 실행 하려면 [Docker의 시작 가이드](https://docs.docker.com/docker-for-windows/)에 나와 있는 지침을 따르세요.

Docker는 Hyper-v에서 실행 되는 [LinuxKit](https://github.com/linuxkit/linuxkit) 기반 가상 컴퓨터를 사용 하 여 2016 (Windows에서 hyper-v 격리 또는 Linux 컨테이너를 사용할 수 있음)에서 처음 릴리스된 이후 windows 데스크톱에서 Linux 컨테이너를 실행할 수 있습니다.

이 모델에서는 Docker 클라이언트가 Windows 데스크톱에서 실행 되지만 Linux VM에서 Docker 데몬을 호출 합니다.

![Moby VM을 컨테이너 호스트로](media/MobyVM.png)

이 모델에서 모든 Linux 컨테이너는 단일 Linux 기반 컨테이너 호스트 및 모든 Linux 컨테이너를 공유 합니다.

* Windows 호스트와 함께 커널을 공유 하 고 다른 Moby VM과 공유할 수 있습니다.
* Linux VM에서 실행 되는 linux 컨테이너를 사용 하 여 저장소 및 네트워킹 속성의 일관성을 유지 합니다.

또한, Linux 컨테이너 호스트 (Moby VM)는 Docker 데몬 및 모든 Docker 데몬 종속성을 실행 해야 합니다.

Moby VM으로 실행 중인지 확인 하려면 Hyper-v 관리자 UI를 사용 하거나 관리자 권한 PowerShell 창에서 실행 `Get-VM` 하 여 moby vm에 대 한 hyper-v 관리자를 확인 하세요.

## <a name="linux-containers-with-hyper-v-isolation"></a>Linux 컨테이너-V 격리

Windows에서 Linux 컨테이너 (LCOW)를 체험 하려면 [windows 10의 linux](../quick-start/quick-start-windows-10-linux.md)컨테이너에서 linux 컨테이너 지침을 따르세요.

Hyper-v 격리를 사용 하는 linux 컨테이너는 컨테이너를 실행할 수 있을 만큼 충분 한 OS를 사용 하 여 최적화 된 Linux VM에서 각 Linux 컨테이너를 실행 합니다. Moby VM 방식과 대조적으로 각 Linux 컨테이너에는 고유한 커널과 VM 샌드박스가 있습니다. 또한 Windows의 Docker에서 직접 관리 합니다.

![Linux 컨테이너 (Hyper-v 격리) (LCOW)](media/lcow-approach.png)

LCOW 모델 컨테이너 관리는 Moby VM 방식과 LCOW 간에 컨테이너 관리가 어떻게 달라 지는 방식에 대해 자세히 살펴보고 각 LCOW 관리는 GRPC 및 containerd를 통해 수행 됩니다.  이는 LCOW에 사용 되는 Linux distro 컨테이너의 재고가 훨씬 작을 수 있다는 것을 의미 합니다.  이제 최적화 된 distro 컨테이너에 대 한 LinuxKit를 사용 하 고 있지만 Kata 같은 다른 프로젝트는 비슷한 고도로 조정 된 Linux distros (Linux 지우기)도 구축 됩니다.

각 LCOW에 대해 자세히 살펴봅니다.

![LCOW 아키텍처](media/lcow.png)

LCOW를 실행 하 고 있는지 확인 하려면으로 `C:\Program Files\Linux Containers`이동 합니다. Docker가 LCOW를 사용 하도록 구성 된 경우 여기에 몇 가지 파일이 있으며, 여기에는 Hyper-v 격리에서 실행 되는 각 컨테이너에서 실행 되는 최소 LinuxKit distro 포함 됩니다.  최적화 된 VM 구성 요소는 Moby VM의 LinuxKit image 보다 훨씬 작은 100 MB 미만입니다.

### <a name="work-in-progress"></a>작업 진행 중

LCOW는 활성 개발 아래에 있습니다. [GitHub](https://github.com/moby/moby/issues/33850) 에서 Moby 프로젝트의 진행 중인 진행률 추적

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

이러한 응용 프로그램에는 모두 볼륨 매핑이 필요 하며, 제대로 시작 되거나 실행 되지 않습니다.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>추가 정보

[LCOW를 설명 하는 Docker 블로그](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 컨테이너 비디오](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-커널 및 빌드 지침](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Moby VM vs LCOW를 사용 하는 경우

### <a name="when-to-use-moby-vm"></a>Moby VM을 사용 하는 경우

지금 바로 다음의 사용자에 게 Linux 컨테이너를 실행 하는 Moby VM 메서드를 제공 하는 것이 좋습니다.

- 안정적인 컨테이너 환경을 원하는 경우  이것은 Windows 기본값에 대 한 Docker입니다.
- Windows 또는 Linux 컨테이너를 실행 하지만 동시에는 거의 작동 하지 않습니다.
- Linux 컨테이너 간에 복잡 하거나 사용자 지정 네트워킹 요구 사항이 있습니다.
- Linux 컨테이너 간에는 커널 격리 (Hyper-v 격리)가 필요 하지 않습니다.

### <a name="when-to-use-lcow"></a>LCOW를 사용 하는 경우

지금 바로 다음 사용자에 게 LCOW 것을 권장 합니다.

- 최신 기술을 테스트 하고자 합니다.
- 한 번에 Windows 및 Linux 컨테이너를 실행 합니다.
- Linux 컨테이너 간에는 커널 격리 (Hyper-v 격리)가 필요 합니다.

## <a name="other-options-we-considered"></a>고려 하는 다른 옵션

Windows에서 Linux 컨테이너를 실행 하는 방법을 살펴보면 WSL로 간주 됩니다. 궁극적으로 Windows의 Linux 컨테이너가 Linux의 Linux 컨테이너와 일관성을 유지 하도록 가상화 기반 접근 방식을 선택 했습니다. Hyper-v를 사용 하면 LCOW 더욱 안전 해 집니다. 나중에 다시 평가할 수 있지만 지금은 LCOW에서 Hyper-v를 사용 합니다.

생각이 있으면 GitHub 또는 UserVoice를 통해 피드백을 보내 주세요.  관심 있는 특정 환경에 대 한 사용자 의견을 확인 하세요.
