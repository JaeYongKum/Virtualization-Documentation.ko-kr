---
title: Windows 컨테이너 정보
description: 컨테이너는 온-프레미스 및 클라우드의 다양한 환경에서 Windows 앱을 포함한 앱을 패키징하고 실행하는 기술입니다. 이 항목에서는 Docker 및 Azure Kubernetes Service를 사용하는 방법을 포함하여 컨테이너에서 앱을 개발하고 배포하는 데 Microsoft, Windows 및 Azure가 어떤 도움이 되는지 설명합니다.
keywords: Docker, 컨테이너
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 4fad299db2c897a6be860ef0cc71e80969c75357
ms.sourcegitcommit: 8dedb887b038fbff872327f51c7416454b301b86
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/06/2019
ms.locfileid: "74909413"
---
# <a name="windows-and-containers"></a>Windows 및 컨테이너

컨테이너는 온-프레미스 및 클라우드의 다양한 환경에서 Windows 및 Linux 애플리케이션을 패키징하고 실행하기 위한 기술입니다. 컨테이너는 앱을 보다 쉽게 개발, 배포 및 관리할 수 있게 해주는 격리된 경량 환경을 제공합니다. 컨테이너는 빠르게 시작되고 중지되므로 수요 변화에 맞게 신속하게 조정해야 하는 앱에 적합합니다. 컨테이너의 경량 특성을 이용해 인프라의 밀도 및 사용률을 높이는 데 유용한 도구로 사용할 수도 있습니다.

![컨테이너를 클라우드 또는 온-프레미스에서 실행하여 거의 모든 언어로 작성된 모놀리식 앱 또는 마이크로서비스를 지원하는 방법을 보여주는 그래픽입니다.](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Microsoft 컨테이너 에코시스템

Microsoft는 컨테이너에서 앱을 개발하고 배포하는 데 도움이 되는 다양한 도구와 플랫폼을 제공합니다.

- Windows에 기본 제공되는 컨테이너 기능을 사용하는 [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)으로 개발하고 테스트하려면 <strong>Windows 10에서 Windows 기반 또는 Linux 기반 컨테이너를 실행합니다</strong>. 또한 [Windows Server에서 컨테이너를 기본적으로 실행](../quick-start/set-up-environment.md?tabs=Windows-Server)할 수도 있습니다.
- Docker, Docker Compose, Kubernetes, Helm 및 기타 유용한 기술에 대한 지원을 포함하는 [Visual Studio](https://docs.microsoft.com/visualstudio/containers/overview) 및 [Visual Studio Code의 강력한 컨테이너 지원 기능](https://code.visualstudio.com/docs/azure/docker)을 사용하여 <strong>Windows 기반 컨테이너를 개발, 테스트, 게시 및 배포합니다</strong>.
- 다른 사람이 사용할 수 있도록 퍼블릭 DockerHub에 <strong>앱을 컨테이너 이미지로 게시</strong>하거나, 조직의 자체 개발 및 배포를 위한 프라이빗 [Azure Container Registry](https://azure.microsoft.com/services/container-registry/)에 게시하고 Visual Studio 및 Visual Studio Code에서 직접 내보내고 가져올 수 있습니다.
- 다른 클라우드나 <strong>Azure에서 대규모로 컨테이너 배포</strong>:

  - Azure Container Registry와 같은 컨테이너 레지스트리에서 앱(컨테이너 이미지)을 가져온 다음, [AKS(Azure Kubernetes Service)](https://docs.microsoft.com/azure/aks/intro-kubernetes)(Windows 기반 앱의 미리 보기) 또는 [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/)과 같은 오케스트레이터를 사용하여 대규모로 배포하고 관리합니다.
  - Azure Kubernetes Service는 Azure 가상 머신에 컨테이너를 배포하고 수십, 수백 또는 수천 개의 대규모 컨테이너를 관리합니다. Azure 가상 머신은 사용자 지정된 Windows Server 이미지(Windows 기반 앱을 배포하는 경우) 또는 사용자 지정된 Ubuntu Linux 이미지(Linux 기반 앱을 배포하는 경우)를 실행합니다.
- [AKS 엔진이 포함된 Azure Stack](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)(Linux 컨테이너의 경우 미리 보기 상태) 또는 [OpenShift가 포함된 Azure Stack](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack)을 사용하여 <strong>온-프레미스에서 컨테이너를 배포합니다</strong>. Windows Server에서 직접 Kubernetes를 설정([Windows 기반 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md) 참조)할 수도 있고, [RedHat OpenShift Container Platform에서 Windows 컨테이너](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821)를 실행하는 기능도 곧 지원될 예정입니다.

## <a name="how-containers-work"></a>컨테이너 작동 방법

컨테이너는 호스트 운영 체제에서 애플리케이션을 실행하기 위해 격리된 경량 사일로입니다. 컨테이너는 이 다이어그램에 표시된 것처럼 마치 호스트 운영 체제에 매립된 배관처럼 호스트 운영 체제의 커널 위에 빌드됩니다.

![컨테이너가 커널을 기반으로 실행되는 방법을 보여주는 아키텍처 다이어그램](media/container-diagram.svg)

컨테이너는 호스트 운영 체제의 커널을 공유하지만 이에 대한 무제한 액세스 권한을 얻지 않습니다. 대신 시스템의 격리된(가상화된 경우도 있음) 뷰를 가져옵니다. 예를 들어 컨테이너는 파일 시스템 및 레지스트리의 가상화된 버전에 액세스할 수 있지만 변경 내용은 컨테이너에만 영향을 주며 컨테이너가 중지되면 삭제됩니다. 컨테이너는 데이터를 저장하기 위해 [Azure Disk](https://azure.microsoft.com/services/storage/disks/) 또는 파일 공유([Azure Files](https://azure.microsoft.com/services/storage/files/) 포함)와 같은 영구 스토리지를 탑재할 수 있습니다.

컨테이너는 커널을 기반으로 빌드되지만 커널은 앱이 실행해야 하는 모든 API 및 서비스를 제공하지 않습니다. 이는 대부분 사용자 모드에서 커널을 기반으로 실행되는 시스템 파일(라이브러리)에서 제공합니다. 컨테이너는 호스트의 사용자 모드 환경과 격리되므로 이러한 사용자 모드 시스템 파일의 자체 복사본이 필요하며, 이는 기본 이미지라고 하는 항목에 패키징됩니다. 기본 이미지는 컨테이너가 빌드되는 기본 계층으로 사용되어 커널이 제공하지 않는 운영 체제 서비스를 제공합니다. 컨테이너 이미지에 대해서는 나중에 자세히 설명합니다.

## <a name="containers-vs-virtual-machines"></a>컨테이너와 가상 머신 비교

VM(가상 머신)은 이 다이어그램에 표시된 것처럼 컨테이너와 달리 자체 커널을 포함하는 완전한 운영 체제를 실행합니다.

![VM이 호스트 운영 체제와 별도로 완전한 운영 체제를 실행하는 방법을 보여주는 아키텍처 다이어그램](media/virtual-machine-diagram.svg)

컨테이너와 가상 머신은 각각 용도가 다릅니다. 실제로 대다수의 컨테이너 배포에서는 특히 클라우드에서 컨테이너를 실행할 때 가상 머신을 하드웨어에서 직접 실행하는 대신 호스트 운영 체제로 사용합니다.

이러한 보완 기술의 유사점과 차이점에 대한 자세한 내용은 [컨테이너와 가상 머신 비교](containers-vs-vm.md)를 참조하세요.

## <a name="container-images"></a>컨테이너 이미지

모든 컨테이너는 컨테이너 이미지에서 만들어집니다. 컨테이너 이미지는 로컬 머신 또는 원격 컨테이너 레지스트리에 있는 계층 스택으로 구성된 파일의 번들입니다. 컨테이너 이미지는 앱을 지원하는 데 필요한 사용자 모드 운영 체제 파일, 앱, 앱의 런타임 또는 종속 항목, 앱이 제대로 실행되는 데 필요한 기타 구성 파일 등으로 구성됩니다.

Microsoft는 사용자 고유의 컨테이너 이미지를 빌드하기 위한 시작 지점으로 사용할 수 있는 여러 이미지(기본 이미지라고 함)를 제공합니다.

* <strong>Windows</strong> - Windows API 및 시스템 서비스(서버 역할 제외)를 전부 포함합니다.
* <strong>Windows Server Core</strong> - Windows Server API(즉, 전체 .NET framework)의 일부를 포함하는 작은 이미지입니다. 또한 대부분의 서버 역할이 포함되어 있지만 팩스 서버가 포함되지 않는 경우도 있습니다.
* <strong>Nano 서버</strong> - .NET Core API 및 일부 서버 역할을 지원하는 가장 작은 Windows Server 이미지입니다.
* <strong>Windows 10 IoT Core</strong> - ARM 또는 x86/x64 프로세서를 실행하는 소형 사물 인터넷 디바이스에 하드웨어 제조업체에서 사용하는 Windows 버전입니다.

앞서 언급했듯이 컨테이너 이미지는 일련의 계층으로 구성됩니다. 각 계층은 함께 중첩될 때 컨테이너 이미지를 나타내는 파일 세트를 포함합니다. 컨테이너의 계층화되는 특성으로 인해 항상 기본 이미지를 대상으로 Windows 컨테이너를 빌드할 필요가 없습니다. 대신, 원하는 프레임워크가 이미 포함된 다른 이미지를 대상으로 지정할 수 있습니다. 예를 들어 .NET 팀은 .NET Core 런타임이 포함된 [.NET Core 이미지](https://hub.docker.com/_/microsoft-dotnet-core)를 게시합니다. 이를 통해 사용자는 .NET Core를 설치하는 프로세스를 복제할 필요가 없습니다. 대신 이 컨테이너 이미지의 계층을 다시 사용할 수 있습니다. .NET Core 이미지 자체는 Nano 서버에 기반하여 빌드됩니다.

자세한 내용은 [컨테이너 기본 이미지](../manage-containers/container-base-images.md)를 참조하세요.

## <a name="container-users"></a>컨테이너 사용자

### <a name="containers-for-developers"></a>개발자용 컨테이너

컨테이너는 개발자가 더 품질 높은 앱을 더 신속하게 구축하여 제공하는 데 도움이 됩니다. 개발자는 컨테이너를 사용하여 여러 환경에 동일하게 몇 초 안에 배포되는 컨테이너 이미지를 만들 수 있습니다. 컨테이너는 호스트 파일 시스템에 영향을 주지 않고 개발 환경을 부트스트랩하고 팀 간에 코드를 공유하는 데 필요한 간편한 메커니즘으로 사용됩니다.

컨테이너는 이식 가능하고 다용도로 사용되며, 다양한 언어로 작성된 앱을 실행할 수 있고, Windows 10, 버전 1607 이상 또는 Windows Server 2016 이상을 실행하는 모든 머신과 호환됩니다. 개발자는 노트북 또는 데스크톱에서 로컬로 컨테이너를 만들고 테스트한 다음, 동일한 컨테이너 이미지를 회사의 프라이빗 클라우드, 퍼블릭 클라우드 또는 서비스 공급자에 배포할 수 있습니다. 컨테이너는 근본적으로 민첩하기 때문에 가상화된 대규모 클라우드 환경에서의 최신 앱 개발을 지원합니다.

### <a name="containers-for-it-professionals"></a>IT 전문가용 컨테이너

관리자는 컨테이너를 통해 업데이트 및 유지 관리가 용이하며 하드웨어 리소스를 더욱 완벽하게 활용하는 인프라를 만들 수 있습니다. IT 전문가들은 컨테이너를 사용하여 개발, QA, 프러덕션 팀에 표준화된 환경을 제공할 수 있습니다. 시스템 관리자들은 컨테이너를 사용하여 운영 체제 설치와 기반 인프라에서의 차이점을 골라낼 수 있습니다.

## <a name="container-orchestration"></a>컨테이너 오케스트레이션

오케스트레이터는 컨테이너 기반 환경을 설정할 때 인프라의 중요한 부분입니다. Docker 및 Windows를 사용하여 컨테이너 몇 개를 수동으로 관리할 수 있지만 앱은 오케스트레이터가 제공되는 5개, 10개 또는 수백 개의 컨테이너를 활용하는 경우가 많습니다.

컨테이너 오케스트레이터는 프로덕션 환경에서 대규모로 컨테이너를 관리하는 데 도움이 되도록 설계되었습니다. 오케스트레이터는 다음을 위한 기능을 제공합니다.

- 대규모 배포
- 워크로드 예약
- 상태 모니터링
- 노드 장애 발생 시 장애 조치
- 확장 또는 축소
- 네트워킹
- 서비스 검색
- 앱 업그레이드 조정
- 클러스터 노드 선호도

Windows 컨테이너에서 사용할 수 있는 다양한 오케스트레이터가 있습니다. Microsoft에서 제공하는 옵션은 다음과 같습니다.
- [AKS(Azure Kubernetes Service)](https://docs.microsoft.com/azure/aks/intro-kubernetes) - 관리형 Azure Kubernetes 서비스 사용
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) - 관리형 서비스 사용
- [AKS Engine을 사용하는 Azure Stack](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) - 온-프레미스의 Azure Kubernetes Service 사용
- [Windows 기반 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md) - Windows에 Kubernetes 직접 설정

## <a name="try-containers-on-windows"></a>Windows에서 컨테이너 사용해 보기

Windows Server 또는 Windows 10에서 컨테이너를 시작하려면 다음을 참조하세요.
> [!div class="nextstepaction"]
> [시작: 컨테이너를 위한 환경 구성](../quick-start/set-up-environment.md)

자신의 시나리오에 적합한 Azure 서비스를 결정하는 데 도움이 필요한 경우 [Azure 컨테이너 서비스](https://azure.microsoft.com/product-categories/containers/) 및 [애플리케이션을 호스트하는 데 사용할 Azure 서비스 선택](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree)을 참조하세요.
