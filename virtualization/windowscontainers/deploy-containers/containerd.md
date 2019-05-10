---
title: Windows 컨테이너 플랫폼
description: 새 컨테이너 빌딩 블록 사용할 수 있는 Windows에 대해 자세히 알아봅니다.
keywords: LCOW, linux 컨테이너, docker, 컨테이너, containerd, cri, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: f8bfd60af18731537c2ce02ca7abdb081f3c7369
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620761"
---
# <a name="container-platform-tools-on-windows"></a>Windows의 컨테이너 플랫폼 도구

Windows 컨테이너 플랫폼 확장! Docker 다른 컨테이너 플랫폼 도구를 빌드하는 것 이제 첫 번째 부분 컨테이너 여정 했습니다.

* [containerd/cri](https://github.com/containerd/cri) -새에서 Windows Server 2019/Windows 10 1809 합니다.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -runc Windows 컨테이너 호스트 만들어냅니다.
* [hcs](https://docs.microsoft.com/virtualization/api/) -호스트 계산 서비스 + shim 간편 하 게 사용 하 여 쉽게 수행할 수 있도록 합니다.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

이 문서에서는 각 컨테이너 플랫폼 도구 뿐만 아니라 Windows 및 Linux 컨테이너 플랫폼에 대해 설명 합니다.

## <a name="windows-and-linux-container-platform"></a>Windows 및 Linux 컨테이너 플랫폼

Linux 환경에서 Docker와 같은 컨테이너 관리 도구 보다 세밀 하 게 컨테이너 도구 집합을 기반으로 만들어진: [runc](https://github.com/opencontainers/runc) 및 [containerd](https://containerd.io/)합니다.

![Linux에서 docker 아키텍처](media/docker-on-linux.png)

`runc` 만들고 [OCI 컨테이너 런타임 사양](https://github.com/opencontainers/runtime-spec)에 따라 컨테이너를 실행 하기 위한 Linux 명령줄 도구가입니다.

`containerd` 다운로드 하 고 컨테이너 이미지 컨테이너 실행 및 관리를 압축을 푸는에서 컨테이너 수명 주기를 관리 하는 디먼이입니다.

Windows에서 다른 접근 방식을 했습니다.  Windows 컨테이너를 지원 하기 위해 Docker를 사용 하 여 작업을 착수 했을 때 HCS (호스트 계산 서비스)에 직접 구축 합니다.  [이 블로그 게시물](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) 매우 다양 한 이유는 HCS 구축 및 이유 것이 방법을 사용 하는 데 걸린 컨테이너 처음에 대 한 정보.

![Windows에서 초기 Docker 엔진 아키텍처](media/hcs.png)

이 시점에서 Docker는 여전히는 HCS를 직접 호출합니다. 하지만 그 이후에 컨테이너 관리 도구가 Windows 컨테이너와 컨테이너 호스트 containerd 및 Linux에서 runc에서 호출 하는 방법은 containerd runhcs를 호출할 수 Windows를 포함 하도록 확장 합니다.

## <a name="runhcs"></a>runhcs

`runhcs` 분기는 `runc`.  마찬가지로 `runc`, `runhcs` 명령줄 클라이언트 패키지 열기 (Initiative) 형식에 따라 응용 프로그램을 실행 하기 위한 이며 열기 컨테이너 이니셔티브 사양 준수 구현 합니다.

Runc와 runhcs 기능 차이점은 다음과 같습니다.

* `runhcs` Windows에서 실행 됩니다.  [HCS](containerd.md#hcs) 를 만들고 컨테이너 관리와 통신 합니다.
* `runhcs` 다양 한 다른 컨테이너 형식 실행할 수 있습니다.

  * Windows 및 Linux [Hyper-v 격리](../manage-containers/hyperv-container.md)
  * Windows 컨테이너 (컨테이너 이미지는 컨테이너 호스트 일치 해야 합니다)를 처리 합니다.

**사용량:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 시작 하는 컨테이너 인스턴스에 대 한 사용자 이름이입니다. 이름은은 컨테이너 호스트에서 고유 해야 합니다.

번들 디렉터리 (사용 하 여 `-b bundle`)는 선택 사항입니다.  
Runc와 마찬가지로 컨테이너 번들을 사용 하 여 구성 됩니다. 컨테이너의 번들은 컨테이너의 OCI 사양 파일을 사용 하 여 디렉터리 "config.json".  "번들"에 대 한 기본값은 현재 디렉터리입니다.

OCI 사양 파일, "config.json"가 두 개의 필드가 제대로 실행 되도록 해야 합니다.

* 컨테이너의 스크래치 공간에 대 한 경로
* 컨테이너의 계층 디렉터리의 경로

Runhcs에서 사용할 수 있는 컨테이너 명령은 다음과 같습니다.

* 만들고 컨테이너를 실행 하는 도구
  * 만들고 컨테이너 실행 **실행**
  * 컨테이너 만들기 **만들기**

* 컨테이너에서 실행 중인 프로세스를 관리 하기 위한 도구:
  * 생성된 된 컨테이너에서 사용자 정의 하는 프로세스를 실행 **시작**
  * **exec** 컨테이너 내에서 새 프로세스를 실행합니다.
  * **일시 중지** 일시 중지 컨테이너 내에서 모든 프로세스를 일시 중단
  * 이전에 일시 중지 된 모든 프로세스를 다시 시작 **을 다시 시작**
  * **ps** ps는 컨테이너 내에서 실행 중인 프로세스를 표시 합니다.

* 컨테이너의 상태를 관리 하기 위한 도구
  * 컨테이너의 상태를 출력 하는 **상태**
  * 지정된 된 신호를 보내는 **중단** (기본값: SIGTERM) 컨테이너의 초기화 프로세스에
  * 분리 된 컨테이너를 사용 하 여 자주 사용 되는 컨테이너에서 보유 한 모든 리소스를 삭제 **삭제**

다중 컨테이너 간주 될 수 있는 유일한 명령 **목록**입니다.  지정 된 루트 runhcs 시작 되지 않는 실행 또는 일시 중지 컨테이너를 나열 합니다.

### <a name="hcs"></a>HCS

사용할 수 있는 두 개의 래퍼는 HCS 인터페이스 github 에합니다. HCS는 C API 이므로 래퍼 쉽게 HCS는 더 높은 수준의 언어에서 호출할 수 있습니다.  

* [hcsshim](https://github.com/microsoft/hcsshim) -HCSShim Go에 작성 이며 runhcs의 기초가 됩니다.
AppVeyor의 최신 버전을 선택 하거나 직접 작성 합니다.
* [dotnet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet computevirtualization C# 래퍼는 HCS입니다.

HCS (직접 또는 래퍼를 통해)를 사용 하려면 한 녹/Haskell/InsertYourLanguage 래퍼는 HCS 하려면, 설명을 두십시오.

에 대 한 하위 수준에 HCS [John Stark DockerCon 프레젠테이션](https://www.youtube.com/watch?v=85nCF5S8Qok)확인 하세요.

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 지원은 Server 2019/Windows 10 1809에서에서 사용할 수 있는 이상만입니다.  도 여전히 적극적으로 Windows에 대 한 containerd를 개발 하는 것입니다.
> 개발자/테스트 합니다.

[CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (컨테이너 런타임 인터페이스) 공유 샌드박스에서 workload(s)로 컨테이너에 설명 OCI 사양 단일 컨테이너를 정의 하는 동안 환경에 포드를 호출 합니다.  포드는 하나 이상의 컨테이너 워크 로드를 포함할 수 있습니다.  포드가는 Kubernetes 및 서비스 패브릭 메시 처리 메모리 및 vNETs와 같은 일부 공유 리소스를 사용 하 여 동일한 호스트에 있어야 하는 그룹화 된 워크 로드와 같은 컨테이너 오 케 스트레이 터를 사용 합니다.

containerd/cri의 다음 호환성 매트릭스를 포드를 사용할 수 있습니다.

| 호스트 OS | 컨테이너 OS | 격리 | 포드 지원 하나요? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | 예-true 다중 컨테이너 포드를 지원 합니다. |
|  | Windows Server 2019/1809 | `process`* 또는 `hyperv` | 예-각 워크 로드 컨테이너 OS 유틸리티 VM OS를 일치 하는 경우 true 다중 컨테이너 포드를 지원 합니다. |
|  | WindowsServer 2016</br>Windows Server 1709</br>Windows Server 1803 | `hyperv` | 부분-지원 포드 컨테이너 OS 유틸리티 VM OS를 일치 하는 경우 유틸리티 VM 당 하나의 프로세스 격리 된 컨테이너를 지원할 수 있는 샌드박스 합니다. |

\*Windows 10 호스트에 Hyper-v 격리만 지원

CRI 사양에 대 한 링크.

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -포드 사양
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -워크 로드 사양

![Containerd 기반 컨테이너 환경](media/containerd-platform.png)

Windows Server 2016 이상 시스템에서 runHCS 및 containerd 관리할 수, 포드 (컨테이너의 그룹)을 지 원하는 필요한 Windows의 컨테이너 도구에 대 한 주요 변경 사항.  CRI 지원은 사용 가능한 Windows Server 2019/Windows 10 1809 이상입니다.
