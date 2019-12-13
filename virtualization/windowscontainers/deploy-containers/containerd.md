---
title: Windows 컨테이너 플랫폼
description: Windows에서 사용할 수 있는 새 컨테이너 구성 요소에 대해 자세히 알아보세요.
keywords: LCOW, linux 컨테이너, docker, 컨테이너, containerd, cri, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 74e22702aa4be30055b3f4f48c7fac926d793095
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909923"
---
# <a name="container-platform-tools-on-windows"></a>Windows의 컨테이너 플랫폼 도구

Windows 컨테이너 플랫폼이 확장 되 고 있습니다. Docker는 컨테이너의 첫 번째 부분입니다. 이제 다른 컨테이너 플랫폼 도구를 빌드하고 있습니다.

* [containerd/cri](https://github.com/containerd/cri) -windows Server 2019/windows 10 1809의 새로운 기능입니다.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -runc에 해당 하는 Windows 컨테이너 호스트입니다.
* [hcs](https://docs.microsoft.com/virtualization/api/) -더 쉽게 사용할 수 있도록 호스트 계산 서비스 + 편리한 shim입니다.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet--를 통한 evirtualization](https://github.com/microsoft/dotnet-computevirtualization)

이 문서에서는 Windows 및 Linux 컨테이너 플랫폼과 각 컨테이너 플랫폼 도구에 대해 설명 합니다.

## <a name="windows-and-linux-container-platform"></a>Windows 및 Linux 컨테이너 플랫폼

Linux 환경에서 Docker와 같은 컨테이너 관리 도구는 [runc](https://github.com/opencontainers/runc) 및 [containerd](https://containerd.io/)의 보다 세분화 된 컨테이너 도구 집합을 기반으로 합니다.

![Linux의 Docker 아키텍처](media/docker-on-linux.png)

`runc`은 [OCI 컨테이너 런타임 사양](https://github.com/opencontainers/runtime-spec)에 따라 컨테이너를 만들고 실행 하는 데 사용할 수 있는 Linux 명령줄 도구입니다.

`containerd`는 컨테이너 이미지를 다운로드 하 고 컨테이너 실행 및 감독에 대 한 압축을 푸는 컨테이너 수명 주기를 관리 하는 디먼입니다.

Windows에서는 다른 방법을 사용 했습니다.  Docker를 사용 하 여 Windows 컨테이너를 지원 하기 시작할 때 HCS (호스트 계산 서비스)에 직접 빌드 했습니다.  [이 블로그 게시물](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) 은 HCS를 구축한 이유에 대 한 정보와이 방법을 컨테이너에 처음으로 사용한 이유를 모두 제공 합니다.

![Windows의 초기 Docker 엔진 아키텍처](media/hcs.png)

이 시점에서 Docker는 여전히 HCS로 직접 호출 합니다. 그러나 앞으로 Windows 컨테이너 및 Windows 컨테이너 호스트를 포함 하도록 확장 하는 컨테이너 관리 도구는 Linux의 containerd 및 runc에서 호출 하는 방식으로 containerd를 호출 하 고 runhcs 수 있습니다.

## <a name="runhcs"></a>runhcs

`runhcs`은 `runc`포크입니다.  `runc`와 마찬가지로 `runhcs`은 OCI (Open Container 이니셔티브) 형식에 따라 패키지 된 응용 프로그램을 실행 하기 위한 명령줄 클라이언트 이며 Open Container 이니셔티브 사양의 규격 구현입니다.

Runc와 runhcs의 기능적 차이점은 다음과 같습니다.

* `runhcs`은 Windows에서 실행 됩니다.  [HCS](containerd.md#hcs) 와 통신 하 여 컨테이너를 만들고 관리 합니다.
* `runhcs` 다양 한 컨테이너 유형을 실행할 수 있습니다.

  * Windows 및 Linux [hyper-v 격리](../manage-containers/hyperv-container.md)
  * Windows 프로세스 컨테이너 (컨테이너 이미지는 컨테이너 호스트와 일치 해야 함)

**사용:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>`는 시작 중인 컨테이너 인스턴스의 이름입니다. 이름은 컨테이너 호스트에서 고유 해야 합니다.

번들 디렉터리 (`-b bundle`사용)는 선택 사항입니다.  
Runc와 마찬가지로 컨테이너는 번들을 사용 하 여 구성 됩니다. 컨테이너의 번들은 컨테이너의 OCI 사양 파일인 "app.config"를 사용 하는 디렉터리입니다.  "번들"의 기본값은 현재 디렉터리입니다.

OCI 사양 파일 "config.xml"에는 올바르게 실행 하기 위한 두 개의 필드가 있어야 합니다.

* 컨테이너의 스크래치 공간에 대 한 경로입니다.
* 컨테이너의 계층 디렉터리에 대 한 경로입니다.

Runhcs에서 사용할 수 있는 컨테이너 명령은 다음과 같습니다.

* 컨테이너를 만들고 실행 하는 도구
  * **실행** -컨테이너를 만들고 실행 합니다.
  * 컨테이너 만들기 **만들기**

* 컨테이너에서 실행 되는 프로세스를 관리 하는 도구:
  * **시작** -생성 된 컨테이너에서 사용자 정의 프로세스를 실행 합니다.
  * **exec** 는 컨테이너 내에서 새 프로세스를 실행 합니다.
  * **일시 중지** 일시 중지는 컨테이너 내의 모든 프로세스를 일시 중단 합니다.
  * **resume** 은 이전에 일시 중지 된 모든 프로세스를 다시 시작 합니다.
  * **ps** ps는 컨테이너 내에서 실행 중인 프로세스를 표시 합니다.

* 컨테이너 상태를 관리 하는 도구
  * **상태** 출력 컨테이너의 상태를 출력 합니다.
  * **kill** 은 지정 된 신호 (기본값: SIGTERM)를 컨테이너의 init 프로세스로 보냅니다.
  * **delete** 는 분리 된 컨테이너에서 자주 사용 되는 컨테이너에서 보유 한 모든 리소스를 삭제 합니다.

다중 컨테이너로 간주할 수 있는 유일한 명령은 **목록**입니다.  지정 된 루트를 사용 하 여 runhcs에서 시작한 실행 중이거나 일시 중지 된 컨테이너를 나열 합니다.

### <a name="hcs"></a>HCS

GitHub에서 HCS와 상호 작용 하는 데 사용할 수 있는 두 가지 래퍼가 있습니다. HCS는 C API 이므로 래퍼를 사용 하면 상위 수준 언어에서 HCS를 쉽게 호출할 수 있습니다.  

* [hcsshim](https://github.com/microsoft/hcsshim) -Hcsshim는 Go로 작성 되었으며 runhcs의 기반이 됩니다.
AppVeyor에서 최신 버전을 얻거나 직접 빌드 하세요.
* [dotnet](https://github.com/microsoft/dotnet-computevirtualization) -----HCS evirtualization은 C#

HCS (직접 또는 래퍼를 통해)를 사용 하려는 경우 또는 HCS에 Rust/Haskell/Insert 언어 래퍼를 만들려는 경우 주석을 남겨 두세요.

HCS에 대 한 자세한 내용을 보려면 [John Stark 'S DockerCon 프레젠테이션을](https://www.youtube.com/watch?v=85nCF5S8Qok)시청 하세요.

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 지원은 서버 2019/Windows 10 1809 이상 에서만 사용할 수 있습니다.  Windows 용 containerd도 적극적으로 개발 하 고 있습니다.
> 개발/테스트 전용입니다.

OCI 사양은 단일 컨테이너를 정의 하지만 [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (container runtime interface)는 pod 라는 공유 샌드박스 환경에서 작업으로 컨테이너를 설명 합니다.  Pod는 하나 이상의 컨테이너 작업을 포함할 수 있습니다.  Pod let 컨테이너 orchestrator Kubernetes 및 Service Fabric 메시는 메모리 및 Vnet 같은 일부 공유 리소스와 동일한 호스트에 있어야 하는 그룹화 된 작업을 처리 합니다.

containerd/cri는 pod에 대해 다음과 같은 호환성 매트릭스를 설정 합니다.

| 호스트 OS | 컨테이너 OS | 격리성 | Pod 지원 여부 |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | 예-진정한 다중 컨테이너 pod을 지원 합니다. |
|  | Windows Server 2019/1809 | `process`* 또는 `hyperv` | 예. 각 워크 로드 컨테이너 OS가 유틸리티 VM OS와 일치 하는 경우 진정한 다중 컨테이너 pod을 지원 합니다. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | 부분-컨테이너 OS가 유틸리티 VM OS와 일치 하는 경우 유틸리티 VM 마다 단일 프로세스 격리 컨테이너를 지원할 수 있는 pod 샌드박스를 지원 합니다. |

\*Windows 10 호스트는 Hyper-v 격리만 지원 합니다.

CRI 사양에 대 한 링크:

* [Runpodsandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod 사양
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -워크 로드 사양

![Containerd 기반 컨테이너 환경](media/containerd-platform.png)

RunHCS 및 containerd는 모두 Windows 시스템 서버 2016 이상에서 관리할 수 있지만 Pod (컨테이너 그룹)을 지원 하려면 Windows의 컨테이너 도구에 대 한 주요 변경 내용이 필요 합니다.  CRI 지원은 Windows Server 2019/Windows 10 1809 이상에서 사용할 수 있습니다.
