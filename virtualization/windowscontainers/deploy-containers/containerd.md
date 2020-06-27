---
title: Windows 컨테이너 플랫폼
description: Windows에서 사용할 수 있는 새 컨테이너 구성 요소에 대해 자세히 알아보세요.
keywords: LCOW, linux 컨테이너, docker, 컨테이너, containerd, cri, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: conceptual
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: dd7ddbc3784eee67fd67bba20533d520e172ebd3
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192261"
---
# <a name="container-platform-tools-on-windows"></a>Windows의 컨테이너 플랫폼 도구

Windows 컨테이너 플랫폼이 확장되고 있습니다. Docker는 첫 번째 컨테이너였으며, 현재 다른 컨테이너 플랫폼 도구를 빌드하고 있습니다.

* [containerd/cri](https://github.com/containerd/cri) - Windows Server 2019/Windows 10 1809의 새 기능입니다.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - runc에 해당하는 Windows 컨테이너 호스트입니다.
* [hcs](https://docs.microsoft.com/virtualization/api/) - 호스트 컴퓨팅 서비스와 편리한 shim을 통합하여 더 쉽게 사용할 수 있습니다.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

이 문서에서는 Windows 및 Linux 컨테이너 플랫폼과 각 컨테이너 플랫폼 도구에 대해 설명합니다.

## <a name="windows-and-linux-container-platform"></a>Windows 및 Linux 컨테이너 플랫폼

Linux 환경에서 Docker 같은 컨테이너 관리 도구는 보다 세분화된 컨테이너 도구 세트인 [runc](https://github.com/opencontainers/runc) 및 [containerd](https://containerd.io/)를 기반으로 합니다.

![Linux의 Docker 아키텍처](media/docker-on-linux.png)

`runc`는 [OCI 컨테이너 런타임 사양](https://github.com/opencontainers/runtime-spec)에 따라 컨테이너를 만들고 실행하기 위한 Linux 명령줄 도구입니다.

`containerd`는 컨테이너 이미지를 다운로드하고 압축을 푸는 과정부터 컨테이너를 실행하고 감독하는 과정까지 컨테이너의 전체 수명 주기를 관리하는 디먼입니다.

Windows에서는 다른 방법을 사용했습니다.  저희는 Docker를 사용하여 Windows 컨테이너 지원을 시작할 때 HCS(호스트 컴퓨팅 서비스)에 직접 빌드했습니다.  [이 블로그 게시물](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332)에는 저희가 HCS를 빌드한 이유와 이 방법을 컨테이너에 처음으로 사용한 이유가 자세히 설명되어 있습니다.

![Windows의 초기 Docker 엔진 아키텍처](media/hcs.png)

현재 Docker는 여전히 HCS로 직접 호출합니다. 그러나 앞으로 Windows 컨테이너 및 Windows 컨테이너 호스트를 포함하도록 확장 중인 컨테이너 관리 도구는 Linux의 containerd 및 runc에서 호출하는 방식으로 containerd 및 runhcs로 호출할 수 있습니다.

## <a name="runhcs"></a>runhcs

`runhcs`는 `runc`에서 갈라져 나왔습니다.  `runc`와 마찬가지로 `runhcs`는 OCI(Open Container Initiative) 형식에 따라 패키징된 애플리케이션을 실행하는 명령줄 클라이언트이며, Open Container Initiative 사양에 부합하는 방식으로 구현되었습니다.

runc와 runhcs의 기능적 차이점은 다음과 같습니다.

* `runhcs`는 Windows에서 실행됩니다.  [HCS](containerd.md#hcs)와 통신하여 컨테이너를 만들고 관리합니다.
* `runhcs`는 다음과 같은 다양한 유형의 컨테이너를 실행할 수 있습니다.

  * Windows 및 Linux [Hyper-V 격리](../manage-containers/hyperv-container.md)
  * Windows 프로세스 컨테이너(컨테이너 이미지가 컨테이너 호스트와 일치해야 함)

**사용법:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>`는 여러분이 시작하려는 컨테이너 인스턴스의 이름입니다. 이 이름은 컨테이너 호스트에서 고유해야 합니다.

번들 디렉터리(`-b bundle` 사용)는 선택 사항입니다.
runc와 마찬가지로 컨테이너는 번들을 사용하여 구성됩니다. 컨테이너의 번들은 컨테이너의 OCI 사양 파일 "app.config"가 포함된 디렉터리입니다.  "번들"의 기본값은 현재 디렉터리입니다.

OCI 사양 파일 "config.xml"에는 올바른 실행을 위한 다음 2개 필드가 있어야 합니다.

* 컨테이너의 스크래치 공간에 대한 경로
* 컨테이너의 계층 디렉터리에 대한 경로

runhcs에서 사용할 수 있는 컨테이너 명령은 다음과 같습니다.

* 컨테이너를 만들고 실행하는 도구
  * **run** - 컨테이너를 만들고 실행합니다.
  * **create** - 컨테이너를 만듭니다.

* 컨테이너에서 실행되는 프로세스를 관리하는 도구:
  * **start** - 만든 컨테이너에서 사용자 정의 프로세스를 실행합니다.
  * **exec** - 컨테이너 내부에서 새 프로세스를 실행합니다.
  * **pause** - 컨테이너 내부의 모든 프로세스를 일시 중지합니다.
  * **resume** - 이전에 일시 중지된 모든 프로세스를 다시 시작합니다.
  * **ps** - 컨테이너 내부에서 실행 중인 프로세스를 표시합니다.

* 컨테이너 상태를 관리하는 도구
  * **state** - 컨테이너의 상태를 출력합니다.
  * **kill** - 지정된 신호(기본값: SIGTERM)를 컨테이너의 초기화 프로세스에 전달합니다.
  * **delete** - 분리된 컨테이너에서 자주 사용하는 컨테이너에 들어 있는 모든 리소스를 삭제합니다.

다중 컨테이너로 간주할 수 있는 유일한 명령은 **list**입니다.  이 명령은 runhcs에서 지정된 루트를 사용하여 시작한 현재 실행 중이거나 일시 중지된 컨테이너를 나열합니다.

### <a name="hcs"></a>HCS

GitHub에는 HCS와 상호 작용하는 데 사용할 수 있는 두 가지 래퍼가 있습니다. HCS는 C API이므로 래퍼를 사용하면 고급 언어에서 HCS를 쉽게 호출할 수 있습니다.

* [hcsshim](https://github.com/microsoft/hcsshim) - HCSShim은 Go로 작성되었으며 runhcs의 기반이 됩니다.
AppVeyor에서 최신 버전을 다운로드하거나 직접 빌드할 수 있습니다.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) - dotnet-computevirtualization은 HCS용 C# 래퍼입니다.

HCS를 사용하려면(직접 또는 래퍼를 통해) 또는 HCS 주변에 Rust/Haskell/InsertYourLanguage를 만들려면 주석을 남겨 주세요.

HCS에 대한 자세한 내용은 [John Stark의 DockerCon 프레젠테이션](https://www.youtube.com/watch?v=85nCF5S8Qok)을 시청하세요.

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 지원은 Server 2019/Windows 10 1809 이상에서만 사용할 수 있습니다.  현재 Windows용 containerd도 적극적으로 개발하고 있습니다.
> 개발/테스트 전용입니다.

OCI 사양은 단일 컨테이너를 정의하지만, [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto)(컨테이너 런타임 인터페이스)는 컨테이너를 Pod라는 공유 샌드박스 환경의 워크로드로 설명합니다.  Pod는 하나 이상의 컨테이너 워크로드를 포함할 수 있습니다.  Pod를 사용하면 메모리 및 vNET 같은 일부 공유 리소스와 동일한 호스트에 있어야 하는 그룹화된 작업을 Kubernetes 및 Service Fabric Mesh 같은 컨테이너 오케스트레이터에서 처리할 수 있습니다.

다음은 containerd/cri가 Pod에 대해 지원하는 호환성 매트릭스입니다.

| 호스트 OS | 컨테이너 OS | 격리 | Pod 지원 여부 |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | 예 - 진정한 다중 컨테이너 Pod를 지원합니다. |
|  | Windows Server 2019/1809 | `process`* 또는 `hyperv` | 예 - 각 워크로드 컨테이너 OS가 유틸리티 VM OS와 일치하는 경우 진정한 다중 컨테이너 Pod를 지원합니다. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | 부분적으로 지원 - 컨테이너 OS가 유틸리티 VM OS와 일치하는 경우 유틸리티 VM마다 단일 프로세스 격리 컨테이너를 지원할 수 있는 Pod 샌드박스를 지원합니다. |

\*Windows 10 호스트는 Hyper-V 격리만 지원

CRI 사양의 링크:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - Pod 사양
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - 워크로드 사양

![containerd 기반 컨테이너 환경](media/containerd-platform.png)

runHCS와 containerd 둘 다 Windows 시스템 Server 2016 이상을 관리할 수 있지만, Pod(컨테이너 그룹)을 지원하려면 Windows의 컨테이너 도구를 대폭 변경해야 합니다.  CRI 지원은 Windows Server 2019/Windows 10 1809 이상에서 사용할 수 있습니다.
