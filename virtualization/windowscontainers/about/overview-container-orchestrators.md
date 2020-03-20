---
title: Windows 컨테이너 오케스트레이션 개요
description: Windows 컨테이너 Orchestrator에 대해 알아봅니다.
keywords: Docker, 컨테이너
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853914"
---
# <a name="windows-container-orchestration-overview"></a>Windows 컨테이너 오케스트레이션 개요

작은 크기와 애플리케이션 방향으로 인해 컨테이너는 빠른 전달 환경과 마이크로 서비스 기반 아키텍처에 적합합니다. 하지만 컨테이너와 마이크로 서비스를 사용하는 환경에는 추적해야 하는 구성 요소가 수백 또는 수천 개 있을 수 있습니다. 수십 개의 가상 머신이나 실제 서버는 수동으로 관리할 수 있지만, 프로덕션 규모의 컨테이너 환경은 자동화 없이 제대로 관리할 수 있는 방법이 없습니다. 이런 작업은 Orchestrator에게 맡겨야 합니다. Orchestrator는 다수의 컨테이너 및 이들이 서로 상호 작용하는 방식을 자동화하고 관리하는 프로세스입니다.

Orchestrator는 다음 작업을 수행합니다.

- 일정 예약: 컨테이너 이미지와 리소스 요청이 지정되면, Orchestrator는 컨테이너를 실행하기에 적합한 머신을 찾습니다.
- 선호도/선호도 방지: 컨테이너 세트가 성능을 위해 서로 가까이 있어야 하는지 또는 가용성을 위해 멀리 떨어져 있어야 하는지를 지정합니다.
- 상태 모니터링: 컨테이너 오류를 감시하고 자동으로 일정을 조정합니다.
- 장애 조치(failover): 각 머신에서 실행되는 항목을 계속 추적하고 오류가 발생한 머신에서 정상 노드로 컨테이너 일정을 조정합니다.
- 크기 조정: 필요에 맞게 수동 또는 자동으로 컨테이너 인스턴스를 추가 또는 제거합니다.
- 네트워킹: 여러 호스트 머신과 통신할 수 있도록 컨테이너를 조정하는 오버레이 네트워크를 제공합니다.
- 서비스 검색: 컨테이너가 호스트 머신 간에 이동하고 IP 주소를 변경하더라도 컨테이너가 자동으로 서로를 찾을 수 있습니다.
- 조정된 애플리케이션 업그레이드: 애플리케이션 중단 시간을 방지하고 문제가 발생하는 경우 롤백이 가능하도록 컨테이너 업그레이드를 관리합니다.

## <a name="orchestrator-types"></a>Orchestrator 유형

Azure는 두 가지 컨테이너 Orchestrator를 제공합니다. AKS(Azure Kubernetes Service) 및 Service Fabric이 그것입니다.

[AKS(Azure Kubernetes Service)](/azure/aks/)를 사용하면 컨테이너화된 애플리케이션을 실행하도록 미리 구성된 가상 머신 클러스터를 간단하게 만들고 구성하고 관리할 수 있습니다. 이를 통해 기존 기술을 사용하고 점차 커지고 있는 대규모의 커뮤니티 전문 지식을 활용하여 컨테이너 기반 애플리케이션을 Microsoft Azure에 배포하고 관리할 수 있습니다. AKS를 사용하면 Kubernetes 및 Docker 이미지 형식을 통해 애플리케이션 이동성을 유지하면서 Azure의 엔터프라이즈급 기능을 활용할 수 있습니다.

[Azure Service Fabric](/azure/service-fabric/)은 신뢰할 수 있는 확장 가능한 마이크로 서비스 및 컨테이너를 쉽게 패키징하고 배포하고 관리할 수 있는 분산 시스템 플랫폼입니다. Service Fabric은 클라우드 원시 애플리케이션을 배포하고 관리하는 데 있어 중요한 과제를 해결합니다. 개발자와 관리자는 복잡한 인프라 문제를 방지하고, 확장 가능하고 신뢰할 수 있으며 관리가 용이한 중요 업무의 까다로운 작업을 구현하는 데 집중할 수 있습니다. Service Fabric은 컨테이너에서 실행되는 이러한 엔터프라이즈급, 계층 1, 클라우드 규모 애플리케이션을 구축하고 관리하는 차세대 플랫폼을 나타냅니다.

## <a name="getting-started"></a>시작

Azure Kubernetes Service 배포를 시작하려면 [Kubernetes 설치 가이드](../kubernetes/getting-started-kubernetes-windows.md)를 참조하세요.

Azure Service Fabric 배포를 시작하려면 [Service Fabric 빠른 시작](/azure/service-fabric/service-fabric-quickstart-containers.md)을 참조하세요.
