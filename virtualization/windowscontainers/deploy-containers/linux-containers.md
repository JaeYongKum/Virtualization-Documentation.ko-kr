---
title: Windows 10의 Linux 컨테이너
description: Hyper-V를 사용하여 마치 네이티브 컨테이너인 것처럼 Windows 10에서 Linux 컨테이너를 실행하는 여러 가지 방법을 알아봅니다.
keywords: LCOW, linux 컨테이너, docker, 컨테이너, windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 843bd0ab7ccf3a227482ba3a3d2677e36b395b29
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854017"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10의 Linux 컨테이너

Linux 컨테이너는 전체 컨테이너 에코 시스템에서 매우 큰 비중을 차지하며 개발자 환경과 프로덕션 환경 모두에 필수적입니다.  컨테이너는 컨테이너 호스트와 커널을 공유하지만, Linux 컨테이너를 Windows에서 직접 실행할 수는 없습니다[*](linux-containers.md#other-options-we-considered).  여기서 가상화가 등장합니다.

현재 Windows용 Docker 클라이언트와 Hyper-V를 사용하여 Linux 컨테이너를 실행하는 다음 두 가지 방법이 있습니다.

- 완전한 Linux VM에서 Linux 컨테이너 실행 - 현재 Docker에서 이 방법을 주로 사용합니다.
- [Hyper-V 격리](../manage-containers/hyperv-container.md)(LCOW)를 사용하여 Linux 컨테이너 실행 - Windows용 Docker 클라이언트의 새로운 옵션입니다.

> _Windows Server OS에서 Linux 컨테이너를 실행하는 기능은 아직 실험 단계입니다. 이 기능을 사용하려면 Docker EE 프로그램의 추가 라이선스가 필요합니다. **이 문서의 나머지 부분은 Windows 10에만 적용됩니다**._

이 문서에서는 각 방법의 작동 원리를 간략하게 설명하고, 언제 어떤 솔루션을 선택해야 하는지 지침을 제공하고, 진행 중인 작업을 공유합니다.

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM의 Linux 컨테이너

Linux VM에서 Linux 컨테이너를 실행하려면 [Docker 시작 가이드](https://docs.docker.com/docker-for-windows/)를 따르세요.

Docker는 Hyper-V에서 실행되는 [LinuxKit](https://github.com/linuxkit/linuxkit) 기반 가상 머신을 사용하여 2016년(Hyper-V 격리 또는 Windows의 Linux 컨테이너를 사용할 수 있게 되기 전)에 처음 릴리스될 때부터 Windows 데스크톱에서 Linux 컨테이너를 실행할 수 있었습니다.

이 모델에서는 Docker 클라이언트가 Windows 데스크톱에서 실행되지만, Linux VM에서 Docker 디먼을 호출합니다.

![컨테이너 호스트로 실행되는 Moby VM](media/MobyVM.png)

이 모델에서는 모든 Linux 컨테이너가 단일 Linux 기반 컨테이너 호스트를 공유하고, 모든 Linux 컨테이너가 다음과 같이 작동합니다.

* 서로 그리고 Moby VM과 커널을 공유하지만 Windows 호스트와는 공유하지 않습니다.
* Linux에서 실행되는 Linux 컨테이너와 일치하는 스토리지 및 네트워킹 속성을 갖습니다(Linux VM에서 실행되므로).

또한 Linux 컨테이너 호스트(Moby VM)는 Docker 디먼 및 모든 Docker 디먼 종속 요소를 실행해야 합니다.

Moby VM으로 실행 중인지 확인하려면 Hyper-V Manager UI를 사용하거나 관리자 권한 PowerShell 창에서 `Get-VM`을 실행하여 Moby VM용 Hyper-V Manager를 확인합니다.

## <a name="linux-containers-with-hyper-v-isolation"></a>Hyper-V 격리를 사용하는 Linux 컨테이너

Windows 10의 Linux 컨테이너(LCOW10)를 사용하려면 [Windows 10의 Linux 컨테이너](../quick-start/quick-start-windows-10-linux.md)의 Linux 컨테이너 지침을 따르세요. 

Hyper-V 격리를 사용하는 Linux 컨테이너는 컨테이너를 실행할 수 있는 OS 버전을 사용하는 최적화된 Linux VM에서 각 Linux 컨테이너를 실행합니다. Moby VM 방식과는 달리, Linux 컨테이너마다 자체 커널과 자체 VM 샌드박스가 있습니다. 또한 Windows의 Docker를 통해 직접 관리됩니다.

![Hyper-V 격리를 사용하는 Linux 컨테이너(LCOW)](media/lcow-approach.png)

Moby VM 방식과 LCOW 방식의 컨테이너 관리가 어떻게 다른지 잘 살펴보면, LCOW 모델에서는 컨테이너 관리가 계속해서 Windows에서 이루어지고 각 LCOW 관리는 GRPC 및 containerd를 통해 발생합니다.  즉, LCOW에 사용되는 Linux 배포판 컨테이너는 인벤토리를 훨씬 더 줄일 수 있습니다.  현재 최적화된 배포판 컨테이너에 LinuxKit를 사용하고 있으며, Kata와 같은 다른 프로젝트에서도 비슷하게 고도로 튜닝된 Linux 배포판(Clear Linux)을 빌드하고 있습니다.

각 LCOW를 자세히 살펴봅시다.

![LCOW 아키텍처](media/lcow.png)

LCOW를 실행 중인지 확인하려면 `C:\Program Files\Linux Containers`로 이동합니다. LCOW를 사용하도록 Docker가 구성된 경우에는 Hyper-V 격리를 사용하는 각 컨테이너에서 실행되는 최소 LinuxKit 배포판을 포함하는 파일 몇 개가 이 위치에 있습니다.  최적화된 VM 구성 요소는 Moby VM의 LinuxKit 이미지보다 훨씬 작은 100MB 미만입니다.

### <a name="work-in-progress"></a>작업 진행 중

LCOW는 현재 개발 중입니다. Moby 프로젝트의 현재 진행 상황은 [GitHub](https://github.com/moby/moby/issues/33850)에서 추적할 수 있습니다.

#### <a name="bind-mounts"></a>마운트 바인딩

`docker run -v ...`으로 마운팅 볼륨을 바인딩하면 Windows NTFS 파일 시스템에서 파일을 저장하므로 POSIX 작업을 하려면 변환이 필요합니다. 일부 파일 시스템 작동이 현재 부분적으로 구현되거나 구현되지 않습니다. 이에 따라 일부 앱이 호환되지 않을 수 있습니다.

바인딩되어 탑재된 볼륨에 대해 이러한 작업은 현재 작동되지 않습니다.

* MkNod
* XAttrWalk
* XAttrCreate
* 잠금
* Getlock
* Auth
* 플러시
* INotify

완전히 구현되지 않는 다음과 같은 사항도 있습니다.

* GetAttr – Nlink 개수가 항상 2개로 보고됩니다.
* Open – ReadWrite, WriteOnly, 및 ReadOnly 플래그만 구현됩니다.

이러한 애플리케이션은 모두 볼륨 매핑이 필요하며 올바르게 시작되거나 실행되지 않습니다.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>추가 정보

[LCOW에 대해 설명하는 Docker 블로그](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 컨테이너 비디오](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-커널 및 빌드 지침](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Moby VM 및 LCOW 사용 시기

### <a name="when-to-use-moby-vm"></a>Moby VM 사용 시기

현재는 다음과 같은 분들에게 Linux 컨테이너를 실행하는 Moby VM 방법을 권장합니다.

- 안정적인 컨테이너 환경을 원합니다.  이는 Windows용 Docker 클라이언트의 기본적인 특징입니다.
- Windows 또는 Linux 컨테이너를 실행하지만 두 개를 동시에 실행하는 경우는 거의 없습니다.
- Linux 컨테이너 간에 복잡한 네트워킹 요구 사항 또는 맞춤형 네트워킹 요구 사항이 있습니다.
- Linux 컨테이너 간에 커널 격리(Hyper-V 격리)가 필요 없습니다.

### <a name="when-to-use-lcow"></a>LCOW를 사용하는 시기

현재는 다음과 같은 분들에게 LCOW를 권장합니다.

- 최신 기술을 테스트하고 싶습니다.
- Windows와 Linux 컨테이너를 동시에 실행합니다.
- Linux 컨테이너 간에 커널 격리(Hyper-V 격리)가 필요합니다.

## <a name="other-options-we-considered"></a>그 외에 고민해 본 옵션

저희는 Windows에서 Linux 컨테이너를 실행하는 방법을 살펴보면서 WSL을 고민했습니다. 결국 저희는 Windows의 Linux 컨테이너가 Linux의 Linux 컨테이너와 일치하도록 가상화 기반 접근 방식을 선택했습니다. 또한 Hyper-V를 사용하면 LCOW의 보안이 강화됩니다. 이후에 다시 평가하겠지만, 현재는 LCOW에서 Hyper-V를 계속 사용할 것입니다.

아이디어가 있는 분들은 GitHub 또는 UserVoice를 통해 피드백을 보내 주시기 바랍니다.  특히 여러분이 원하는 구체적인 경험에 대한 피드백을 보내 주시면 감사하겠습니다.
