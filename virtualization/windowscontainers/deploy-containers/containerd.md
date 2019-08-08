---
title: Windows 컨테이너 플랫폼
description: Windows에서 사용할 수 있는 새 컨테이너 문서 블록에 대해 자세히 알아보세요.
keywords: LCOW, linux 컨테이너, docker, 컨테이너, containerd, cri, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998420"
---
# <a name="container-platform-tools-on-windows"></a>Windows의 컨테이너 플랫폼 도구

Windows 컨테이너 플랫폼이 확장 중입니다! Docker는 컨테이너의 첫 번째 부분으로, 이제 다른 컨테이너 플랫폼 도구를 작성 하 고 있습니다.

* [containerd/cri](https://github.com/containerd/cri) -windows Server 2019/windows 10 1809의 새로운 기능.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -runc에 대응 하는 Windows 컨테이너 호스트입니다.
* [hcs](https://docs.microsoft.com/virtualization/api/) -호스트 계산 서비스 + 편리한 shim을 사용 하 여 쉽게 사용할 수 있도록 합니다.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-고 대 인 evirtualization](https://github.com/microsoft/dotnet-computevirtualization)

이 문서에서는 Windows, Linux 컨테이너 플랫폼 및 각 컨테이너 플랫폼 도구에 대해 설명 합니다.

## <a name="windows-and-linux-container-platform"></a>Windows 및 Linux 컨테이너 플랫폼

Linux 환경에서 Docker와 같은 컨테이너 관리 도구는 보다 세부적인 컨테이너 도구 집합 ( [runc](https://github.com/opencontainers/runc) 및 [containerd](https://containerd.io/))을 기반으로 합니다.

![Linux의 Docker 아키텍처](media/docker-on-linux.png)

`runc` 는 [OCI 컨테이너 런타임 사양](https://github.com/opencontainers/runtime-spec)에 따라 컨테이너를 만들고 실행 하기 위한 Linux 명령줄 도구입니다.

`containerd` 컨테이너 이미지를 다운로드 하 고 압축을 푸는 컨테이너 수명 주기를 관리 하는 데몬입니다.

Windows에서는 다른 방법을 사용 했습니다.  Windows 컨테이너를 지원 하기 위해 Docker 작업을 시작할 때 HCS (호스트 계산 서비스)에 직접 빌드 했습니다.  [이 블로그 게시물](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) 에는 HCS를 작성 한 이유와 처음에이 방법을 컨테이너에 사용 하는 이유가에 대 한 모든 정보가 들어 있습니다.

![Windows의 초기 Docker 엔진 아키텍처](media/hcs.png)

이 시점에서 Docker는 HCS에 직접 전화 합니다. 하지만 앞으로는 Windows 컨테이너를 포함 하도록 확장 된 컨테이너 관리 도구 및 Windows 컨테이너 호스트가 containerd로 전화를 걸고 runhcs에서 containerd 및 runc를 통해이를 호출할 수 있습니다.

## <a name="runhcs"></a>runhcs

`runhcs` 이 ( `runc`가) 포크입니다.  `runhcs` Like `runc`는 OCI (open container 이니셔티브) 형식에 따라 패키지 된 응용 프로그램을 실행 하기 위한 명령줄 클라이언트 이며 열려 있는 컨테이너 이니셔티브 사양의 규격 구현입니다.

Runc와 runhcs의 기능적 차이점은 다음과 같습니다.

* `runhcs` Windows에서 실행 됩니다.  [HCS](containerd.md#hcs) 와 통신 하 여 컨테이너를 만들고 관리 합니다.
* `runhcs` 다양 한 컨테이너 유형을 실행할 수 있습니다.

  * Windows 및 Linux [hyper-v 격리](../manage-containers/hyperv-container.md)
  * Windows 프로세스 컨테이너 (컨테이너 이미지가 컨테이너 호스트와 일치 해야 함)

**사용량:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 은 시작 하는 컨테이너 인스턴스의 이름입니다. 이름은 컨테이너 호스트에서 고유 해야 합니다.

번들 디렉터리 (using `-b bundle`)는 선택 사항입니다.  
Runc와 마찬가지로 컨테이너는 번들을 사용 하 여 구성 됩니다. 컨테이너의 번들은 컨테이너의 OCI 사양 파일인 "config.xml"이 포함 된 디렉터리입니다.  "번들"의 기본값은 현재 디렉토리입니다.

OCI 사양 파일인 "config.xml"에는 올바르게 실행 되는 두 개의 필드가 필요 합니다.

* 컨테이너의 스크래치 공간 경로
* 컨테이너의 layer 디렉터리에 대 한 경로입니다.

Runhcs에서 사용할 수 있는 컨테이너 명령에는 다음이 포함 됩니다.

* 컨테이너를 만들고 실행 하는 도구
  * **run** 은 컨테이너를 만들고 실행 합니다.
  * 컨테이너 만들기 **만들기**

* 컨테이너에서 실행 되는 프로세스를 관리 하는 도구:
  * **start** 만든 컨테이너에서 사용자 정의 프로세스를 실행 합니다.
  * **exec** 는 컨테이너 내에서 새 프로세스를 실행 합니다.
  * **일시 중지** 일시 중지 컨테이너 내의 모든 프로세스를 일시 중단 합니다.
  * **resume** 은 이전에 일시 중지 된 모든 프로세스를 다시 시작 합니다.
  * **ps** ps는 컨테이너 내에서 실행 되는 프로세스를 표시 합니다.

* 컨테이너의 상태를 관리 하는 도구
  * **state** 는 컨테이너의 상태를 출력 합니다.
  * **kill** 은 지정 된 신호 (DEFAULT: SIGTERM)를 컨테이너의 init 프로세스에 보냅니다.
  * **delete** 는 컨테이너에서 보유 한 리소스 중 분리 된 컨테이너에서 자주 사용 하는 리소스도 삭제 합니다.

다중 컨테이너로 간주할 수 있는 유일한 명령으로는 **목록**입니다.  지정 된 루트로 runhcs 하 여 시작한 실행 중이거나 일시 중지 된 컨테이너가 나열 됩니다.

### <a name="hcs"></a>HCS

HCS를 사용 하 여 인터페이스 하는 GitHub에는 두 가지 래퍼를 사용할 수 있습니다. HCS는 C API 이기 때문에 더 높은 수준의 언어에서 HCS를 쉽게 호출할 수 있도록 합니다.  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim은 runhcs의 기초가 되며 이동 중에도 작성 됩니다.
최신 버전을 AppVeyor 직접 만들어 보세요.
* [dotnet-](https://github.com/microsoft/dotnet-computevirtualization) 즉, HCS에 대 한 c # 래퍼를 통해 dotnet-즉, 모든 것이 사전 인 evirtualization입니다.

HCS (직접 또는 래퍼를 통해)를 사용 하거나 HCS에 대 한 Rust/Haskell/Insert관련 언어 래퍼를 만들려는 경우에는 의견을 남겨 주세요.

HCS를 더 자세히 보려면 [John 극명의 DockerCon 표현을](https://www.youtube.com/watch?v=85nCF5S8Qok)시청 하세요.

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 지원은 서버 2019/Windows 10 1809 이상 에서만 사용할 수 있습니다.  또한 여전히 Windows 용 containerd을 적극적으로 개발 하 고 있습니다.
> 개발/테스트만 가능 합니다.

OCI 사양에서는 단일 컨테이너를 정의 하 고 [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (컨테이너 런타임 인터페이스)는 pod 라는 공유 샌드박스 환경에서 컨테이너에 작업을 설명 합니다.  Pods는 하나 이상의 컨테이너 작업을 포함할 수 있습니다.  Pods orchestrators Kubernetes 및 Service Fabric 메시 핸들과 같은 컨테이너를 사용 하 여 메모리 및 vNETs 같은 일부 공유 리소스와 동일한 호스트에 있어야 하는 작업을 그룹화 합니다.

containerd/cri에서는 pods에 대해 다음과 같은 호환성 행렬을 사용할 수 있습니다.

| 호스트 OS | 컨테이너 OS | 격리가 | Pod 지원 여부 |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Yes-진정한 다중 컨테이너 pods를 지원 합니다. |
|  | Windows Server 2019/1809 | `process`* 또는 `hyperv` | Yes-각 작업 영역 컨테이너 OS가 유틸리티 VM OS와 일치 하는 경우 진정한 다중 컨테이너 pods를 지원 합니다. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | 부분-컨테이너 OS가 유틸리티 VM OS와 일치 하는 경우 유틸리티 VM 당 단일 프로세스 격리 컨테이너를 지원할 수 있는 pod 샌드박스를 지원 합니다. |

\ * Windows 10 호스트는 Hyper-v 격리만 지원 합니다.

CRI spec에 대 한 링크:

* [Runpodsandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod 사양
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -작업 사양

![Containerd 기반 컨테이너 환경](media/containerd-platform.png)

RunHCS 및 containerd는 모두 Windows 시스템 서버 2016 이상에서 관리할 수 있지만, Windows의 컨테이너 도구에 대 한 변경 사항은 Pods (컨테이너 그룹)을 지 원하는 데 필요 합니다.  CRI 지원은 Windows Server 2019/Windows 10 1809 이상에서 사용할 수 있습니다.
