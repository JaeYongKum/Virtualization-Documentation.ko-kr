---
title: Windows 컨테이너 정보
description: 컨테이너는 온-프레미스 및 클라우드에서 다양 한 환경에 걸친 Windows 앱을 포함 하 여 앱을 패키지화 및 실행 하기 위한 기술입니다. 이 항목에서는 Docker 및 Azure Kubernetes 서비스를 사용 하는 것을 포함 하 여 컨테이너에서 앱을 개발 하 고 배포 하는 Microsoft, Windows, Azure 도움말에 대해 설명 합니다.
keywords: Docker, 컨테이너
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: acce214cc8991f20c979b6dbe636590416841cb9
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254293"
---
# <a name="windows-and-containers"></a>Windows 및 컨테이너

컨테이너는 온-프레미스 및 클라우드에서 다양 한 환경에서 Windows 및 Linux 응용 프로그램을 패키지화 하 고 실행 하기 위한 기술입니다. 컨테이너는 앱을 더 쉽게 개발, 배포 및 관리할 수 있도록 하는 간단 하 고 격리 된 환경을 제공 합니다. 컨테이너는 빠르게 시작 하 고 중지 하므로 수요 변화에 신속 하 게 적응할 필요가 있는 앱에 적합 합니다. 또한 컨테이너의 간단한 특성을 통해 인프라의 밀도와 활용도를 높이는 데 유용한 도구를 만들 수 있습니다.

![클라우드 또는 온-프레미스에서 컨테이너를 실행 하는 방법을 보여 주는 그래픽으로, 거의 모든 언어로 작성 된 monolithic 앱 또는 마이크로 서비스를 지원 합니다.](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Microsoft 컨테이너 환경

Microsoft는 컨테이너에서 앱을 개발 하 고 배포 하는 데 도움이 되는 다양 한 도구와 플랫폼을 제공 합니다.

- Windows <strong>10에서 windows 기반 또는 Linux 기반 컨테이너를 실행</strong> 하면 [Docker 데스크톱](https://store.docker.com/editions/community/docker-ce-desktop-windows)을 사용 하 여 개발 및 테스트 하 고 windows에 기본 제공 되는 컨테이너 기능을 사용할 수 있습니다. [Windows Server에서 기본적으로 컨테이너를 실행할](../quick-start/set-up-environment.md?tabs=Windows-Server)수도 있습니다.
- Docker, Docker 작성, Kubernetes, 투구 및 기타 유용한 기능을 포함 하는 Visual Studio 및 [Visual Studio 코드](https://code.visualstudio.com/docs/azure/docker) [의 강력한 컨테이너 지원을](https://docs.microsoft.com/visualstudio/containers/overview) 사용 하 여 <strong>Windows 기반 컨테이너를 개발, 테스트, 게시 및 배포</strong> 합니다. 신기술.
- 다른 사용자가 사용할 수 있는 공용 DockerHub에 <strong>앱을 컨테이너 이미지로 게시</strong> 하 고, visual Studio 및 Visual studio 코드 내에서 직접 푸시 및 가져오기에 대 한 사설 [Azure 컨테이너 레지스트리](https://azure.microsoft.com/services/container-registry/) 를 배포 합니다. .
- Azure 또는 기타 클라우드의 <strong>크기 조정에서 컨테이너 배포</strong> :

  - Azure 컨테이너 레지스트리와 같은 컨테이너 레지스트리에서 앱 (컨테이너 이미지)을 가져온 다음 orchestrator ( [Azure Kubernetes Service) (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) (Windows 기반 앱의 미리 보기) 또는 azure 서비스 등의 조정자를 사용 하 여 확장 하 여 배포 하 고 관리 합니다. [ 패브릭](https://docs.microsoft.com/azure/service-fabric/).
  - Azure Kubernetes 서비스는 Azure 가상 컴퓨터에 컨테이너를 배포 하 고 크기가 수십 개, 수백 개 또는 수천 개인 경우를 통해 관리 합니다. Azure 가상 컴퓨터는 사용자 지정 Windows Server 이미지 (Windows 기반 앱을 배포 하는 경우) 또는 사용자 지정 Ubuntu Linux 이미지 (Linux 기반 앱을 배포 하는 경우)를 실행 합니다.
- AKS Engine (Linux 컨테이너로 미리 보기에서) 또는 [OpenShift를 사용 하는 Azure 스택의](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack) [azure stack](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) 을 사용 하 여 온 <strong>-프레미스 배포 컨테이너</strong> 입니다. Windows Server ( [windows의 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md)참조)에서 직접 Kubernetes를 설정할 수도 있으며, [Openshift 컨테이너 플랫폼 에서도 redhat Windows 컨테이너](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821) 를 실행 하는 데 대 한 지원 작업을 수행 하 고 있습니다.

## <a name="how-containers-work"></a>컨테이너 작동 방식

컨테이너는 호스트 운영 체제에서 응용 프로그램을 실행 하기 위한 격리 된 경량 사일로 (silo)입니다. 컨테이너는 호스트 운영 체제의 커널 (이 다이어그램에 표시 된 것 처럼 운영 체제의 덮여 있는 것으로 간주할 수 있음) 위에 빌드됩니다.

![컨테이너를 커널을 위에서 실행 하는 방법을 보여 주는 아키텍처 다이어그램](media/container-diagram.svg)

컨테이너에서 호스트 운영 체제의 커널을 공유 하는 동안 컨테이너는 unfettered에 대 한 액세스 권한을 받지 않습니다. 대신 컨테이너는 격리 된 상태로 시스템의 일부 경우에 대 한 가상화 – 뷰를 가져옵니다. 예를 들어 컨테이너는 파일 시스템 및 레지스트리의 가상화 된 버전에 액세스할 수 있지만, 변경 사항은 컨테이너에만 영향을 미치며 중지할 때는 취소 됩니다. 데이터를 저장 하기 위해 컨테이너는 [Azure Disk](https://azure.microsoft.com/services/storage/disks/) 또는 파일 공유 ( [azure Files](https://azure.microsoft.com/services/storage/files/)포함) 등의 영구 저장소를 탑재할 수 있습니다.

컨테이너는 커널의 맨 위에 빌드 되지만, 커널에서 앱을 실행 하는 데 필요한 모든 Api와 서비스를 제공 하지는 않으며,이는 대부분의 경우 사용자 모드에서 커널을 위쪽으로 실행 되는 시스템 파일 (라이브러리)에 의해 제공 됩니다. 컨테이너는 호스트의 사용자 모드 환경에서 격리 되므로 컨테이너에는 기본 이미지 라는 것으로 패키지 된 이러한 사용자 모드 시스템 파일의 고유한 복사본이 필요 합니다. 기본 이미지는 컨테이너가 빌드될 기본 계층으로 사용 되며 커널에서 제공 되지 않는 운영 체제 서비스를 제공 합니다. 그러나 나중에 컨테이너 이미지에 대해 자세히 알아보세요.

## <a name="containers-vs-virtual-machines"></a>컨테이너 대 가상 컴퓨터

컨테이너와 대조적으로 Vm (가상 머신)은이 다이어그램에 표시 된 대로 자체 커널을 포함 하 여 완전 한 운영 체제를 실행 합니다.

![Vm이 호스트 운영 체제 옆의 전체 운영 체제를 실행 하는 방법을 보여 주는 아키텍처 다이어그램](media/virtual-machine-diagram.svg)

컨테이너와 가상 컴퓨터에는 각각 사용이 포함 되며, 특히 클라우드에서 컨테이너를 실행할 때 컨테이너의 여러 배포는 하드웨어에서 직접 실행 하는 대신 호스트 운영 체제로 가상 컴퓨터를 사용 합니다.

이러한 보완적인 기술의 유사점과 차이점에 대 한 자세한 내용은 [컨테이너 vs 가상 컴퓨터](containers-vs-vm.md)를 참조 하세요.

## <a name="container-images"></a>컨테이너 이미지

컨테이너 이미지에서 모든 컨테이너가 만들어집니다. 컨테이너 이미지는 로컬 컴퓨터 또는 원격 컨테이너 레지스트리에 있는 계층의 스택으로 구성 된 파일 번들입니다. 컨테이너 이미지는 앱, 앱, 앱의 런타임 또는 종속성을 지 원하는 데 필요한 사용자 모드 운영 체제 파일과 앱이 제대로 실행 되는 데 필요한 기타 모든 기타 구성 파일으로 구성 됩니다.

Microsoft는 고유한 컨테이너 이미지를 만들기 위한 출발점으로 사용할 수 있는 기본 이미지 라는 여러 이미지를 제공 합니다.

* <strong>Windows</strong> -windows api 및 시스템 서비스 (서버 역할 제외)의 전체 집합을 포함 합니다.
* <strong>Windows Server Core</strong> -전체 .net framework와 같은 Windows server api의 하위 집합을 포함 하는 작은 이미지입니다. 또한 대부분의 서버 역할은 sadly 하지만 팩스 서버는 포함 하지 않습니다.
* <strong>Nano 서버</strong> -.Net Core api와 일부 서버 역할에 대 한 지원이 포함 된 최소 Windows Server 이미지입니다.
* <strong>Windows 10 IoT Core</strong> -ARM 또는 x86/x64 프로세서를 실행 하는 디바이스의 소규모 인터넷을 위한 하드웨어 제조업체에서 사용 하는 windows 버전입니다.

앞에서 언급 한 것 처럼 컨테이너 이미지는 일련의 레이어로 구성 됩니다. 각 계층에는 함께 겹친 경우 컨테이너 이미지를 나타내는 파일 집합이 포함 되어 있습니다. 컨테이너의 계층화 된 특성 때문에 항상 기본 이미지를 대상으로 지정 하 여 Windows 컨테이너를 빌드할 필요는 없습니다. 대신 원하는 프레임 워크를 이미가지고 있는 다른 이미지를 대상으로 할 수 있습니다. 예를 들어 .NET 팀은 .net core 런타임을 전달 하는 [.net core 이미지](https://hub.docker.com/_/microsoft-dotnet-core) 를 게시 합니다. 이 컨테이너 이미지의 레이어를 다시 사용할 수 있도록 사용자가 .NET core를 설치 하는 프로세스를 복제 하지 않아도 됩니다. .NET core 이미지 자체는 Nano 서버에 따라 빌드됩니다.

자세한 내용은 [컨테이너 기본 이미지](../manage-containers/container-base-images.md)를 참조 하세요.

## <a name="container-users"></a>컨테이너 사용자

### <a name="containers-for-developers"></a>개발자를 위한 컨테이너

컨테이너를 통해 개발자는 고품질 앱을 더 빠르게 빌드하고 제공할 수 있습니다. 컨테이너를 사용 하 여 개발자는 환경에 따라 초에 배포 하는 컨테이너 이미지를 만들 수 있습니다. 컨테이너는 팀 간에 코드를 공유 하 고 호스트 파일 시스템에 영향을 주지 않고 개발 환경을 부트스트랩 하는 간단한 메커니즘 역할을 합니다.

컨테이너는 이식 가능 하 고 다양 한 언어로 작성 된 앱을 실행할 수 있으며, Windows 10, 버전 1607 이상 또는 Windows 2016 이상을 실행 하는 컴퓨터와 호환 됩니다. 개발자는 자신의 랩톱이 나 데스크톱에서 컨테이너를 로컬로 만들고 테스트 한 다음 동일한 컨테이너 이미지를 해당 회사의 사설 클라우드, 공용 클라우드 또는 서비스 공급자에 배포할 수 있습니다. 컨테이너의 자연 민첩성은 가상화 된 대규모 클라우드 환경에서 최신 앱 개발 패턴을 지원 합니다.

### <a name="containers-for-it-professionals"></a>IT 전문가를 위한 컨테이너

컨테이너는 관리 하는 것이 더 쉬우며 하드웨어 리소스를 더 완벽 하 게 활용 하는 인프라를 만드는 데 도움이 됩니다. IT 전문가는 컨테이너를 사용 하 여 개발, QA, 프로덕션 팀에 게 표준화 된 환경을 제공할 수 있습니다. 컨테이너를 사용 하 여 시스템 관리자는 운영 체제 설치 및 기본 인프라의 차이를 추상화 합니다.

## <a name="container-orchestration"></a>컨테이너 오케스트레이션

컨테이너 기반 환경을 설정 하는 경우 Orchestrators는 인프라의 중요 한 부분입니다. Docker 및 Windows를 사용 하 여 몇 가지 컨테이너를 수동으로 관리할 수 있지만, 앱은 종종 orchestrators에 포함 된 다섯 개, 10 개 또는 수백 개의 컨테이너를 사용 합니다.

컨테이너 orchestrators는 확장성 및 프로덕션 환경에서 컨테이너를 관리 하는 데 도움이 되도록 작성 되었습니다. Orchestrators는 다음과 같은 기능을 제공 합니다.

- 배율에 배치
- 작업량 예약
- 상태 모니터링
- 노드에 장애가 발생 했을 때 장애 조치
- 확대 또는 축소
- 네트워킹
- 서비스 검색
- 앱 업그레이드 조정
- 클러스터 노드 선호도

Windows 컨테이너와 함께 사용할 수 있는 다양 한 orchestrators 있습니다. Microsoft에서 제공 하는 옵션은 다음과 같습니다.
- [Azure Kubernetes service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) -관리 되는 azure Kubernetes Service 사용
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) -관리 서비스 사용
- [AKS Engine을 사용 하는 Azure Stack](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) -온-프레미스 Azure Kubernetes Service 온-프레미스
- Windows [의 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md) -Windows에서 Kubernetes를 직접 설정 합니다.

## <a name="try-containers-on-windows"></a>Windows에서 컨테이너 체험

Windows Server 또는 Windows 10에서 컨테이너를 시작 하려면 다음을 참조 하세요.
> [!div class="nextstepaction"]
> [시작: 컨테이너에 대 한 환경 구성](../quick-start/set-up-environment.md)

시나리오에 적합 한 Azure 서비스를 결정 하는 데 도움이 필요한 경우 [azure 컨테이너 서비스](https://azure.microsoft.com/product-categories/containers/) 를 참조 하 고 [응용 프로그램을 호스팅하는 데 사용할 Azure 서비스를 선택](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree)합니다.
