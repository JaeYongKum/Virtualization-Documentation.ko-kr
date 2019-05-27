---
title: Windows 컨테이너 orchestrators 정보
description: Windows 컨테이너 orchestrators에 대해 자세히 알아보세요.
keywords: Docker, 컨테이너
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 1ccf63b0ae55501ba32f8bdd61994e7f8006b5e6
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674888"
---
# <a name="about-windows-container-orchestrators"></a>Windows 컨테이너 orchestrators 정보

컨테이너는 크기와 응용 프로그램 방향 때문에 민첩 한 배달 환경 및 microservice 기반 아키텍처에 완벽 합니다. 그러나 컨테이너와 마이크로 서버를 사용 하는 환경에는 수백 또는 수천 개의 구성 요소를 포함 하 여 추적할 수 있습니다. 수십 개 이상의 가상 컴퓨터 또는 물리적 서버를 수동으로 관리할 수 있지만 자동화를 사용 하지 않고 프로덕션 크기 컨테이너 환경을 제대로 관리할 방법이 없습니다. 이 작업은 다양 한 컨테이너를 자동화 하 고 관리 하며 서로 상호 작용 하는 방식에 대 한 orchestrator에 속해야 합니다.

Orchestrators는 다음 작업을 수행 합니다.

- 일정 지정: 컨테이너 이미지와 리소스 요청이 있는 경우 orchestrator는 컨테이너를 실행할 적절 한 컴퓨터를 찾습니다.
- 선호도/선호도 방지: 컨테이너 집합을 서로 가까이 실행할지 여부를 지정 합니다.
- 상태 모니터링: 컨테이너 장애를 감시하고 이를 자동으로 다시 예약합니다.
- 장애 조치 (Failover): 각 컴퓨터에서 실행 중인 항목을 추적 하 고 실패 한 컴퓨터에서 정상 노드로 컨테이너의 일정을 조정 합니다.
- 크기 조정: 수동으로 또는 자동으로 요청에 맞게 컨테이너 인스턴스를 추가 하거나 제거 합니다.
- 네트워킹: 여러 호스트 컴퓨터에서 통신 하도록 컨테이너를 조정 하는 오버레이 네트워크를 제공 합니다.
- 서비스 검색: 컨테이너가 호스트 컴퓨터 간에 이동하고 IP 주소를 변경하는 경우에도 컨테이너를 서로 자동으로 찾을 수 있습니다.
- 조정된 응용 프로그램 업그레이드: 응용 프로그램 작동 중지 시간을 피하도록 컨테이너 업그레이드를 관리하고 문제가 발생하는 경우 롤백합니다.

## <a name="orchestrator-types"></a>Orchestrator 유형

Azure는 두 가지 컨테이너 orchestrators Azure Kubernetes 서비스 (AKS) 및 서비스 패브릭을 제공 합니다.

[Azure Kubernetes 서비스 (AKS)](/azure/aks/) 를 사용 하면 containerized 응용 프로그램을 실행 하도록 미리 구성 된 가상 컴퓨터 클러스터를 만들고 구성 하 고 관리 하는 것이 간편 합니다. 이를 통해 Microsoft Azure에서 컨테이너 기반 응용 프로그램을 배포 하 고 관리 하기 위해 기존 기술을 사용 하 고 방대한 양의 커뮤니티 전문가의 본문에 그릴 수 있습니다. AKS를 사용 하 여 Kubernetes 및 Docker 이미지 형식을 통해 응용 프로그램 이식성을 유지 하면서 Azure의 엔터프라이즈 성적 기능을 활용할 수 있습니다.

[Azure Service Fabric](/azure/service-fabric/)은 신뢰할 수 있는 확장 가능한 마이크로 서비스 및 컨테이너를 쉽게 패키징하고 배포하고 관리할 수 있는 분산 시스템 플랫폼입니다. Service Fabric은 클라우드 원시 응용 프로그램을 배포하고 관리하는 데 있어 중요한 과제를 해결합니다. 개발자와 관리자는 복잡한 인프라 문제를 방지하고, 확장 가능하고 신뢰할 수 있으며 관리가 용이한 중요 업무의 까다로운 작업을 구현하는 데 집중할 수 있습니다. Service Fabric은 컨테이너에서 실행되는 이러한 엔터프라이즈급, 계층 1, 클라우드 규모 응용 프로그램을 구축하고 관리하는 차세대 플랫폼을 나타냅니다.

## <a name="getting-started"></a>시작하기

Azure Kubernetes 서비스 배포를 시작 하려면 [Kubernetes 설정 가이드](../kubernetes/getting-started-kubernetes-windows.md)를 참조 하세요.

Azure Service Fabric 배포를 시작 하려면 [서비스 패브릭 퀵 스타트](/azure/service-fabric/service-fabric-quickstart-containers.md)를 참조 하세요.