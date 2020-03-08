---
title: Windows 컨테이너 오케스트레이션 개요
description: Windows 컨테이너 orchestrator에 대해 알아봅니다.
keywords: Docker, 컨테이너
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853914"
---
# <a name="windows-container-orchestration-overview"></a>Windows 컨테이너 오케스트레이션 개요

컨테이너는 크기가 작고 응용 프로그램 방향은 agile 배달 환경 및 마이크로 서비스 기반 아키텍처에 적합 합니다. 그러나 컨테이너 및 마이크로 서비스를 사용 하는 환경에서는 수백 또는 수천 개의 구성 요소가 추적을 유지할 수 있습니다. 몇 수십 개의 가상 머신 또는 물리적 서버를 수동으로 관리할 수 있지만, 자동화 하지 않고 프로덕션 규모의 컨테이너 환경을 제대로 관리할 수 있는 방법은 없습니다. 이 작업은 오 케 스트레이 터를 자동화 하 고 관리 하며 서로 상호 작용 하는 프로세스를 포함 하는 orchestrator에 속해야 합니다.

Orchestrator 다음 작업을 수행 합니다.

- 일정: 컨테이너 이미지와 리소스 요청이 지정 된 경우 오 케 스트레이 터는 컨테이너를 실행 하는 데 적합 한 컴퓨터를 찾습니다.
- 선호도/선호도 방지: 컨테이너 집합을 성능을 위해 서로 가까이 실행할지, 아니면 가용성이 떨어져 있는지를 지정 합니다.
- 상태 모니터링: 컨테이너 장애를 감시하고 이를 자동으로 다시 예약합니다.
- 장애 조치 (Failover): 각 컴퓨터에서 실행 되는 항목을 추적 하 고 실패 한 컴퓨터의 컨테이너를 정상 노드로 다시 예약 합니다.
- 크기 조정: 컨테이너 인스턴스를 추가 하거나 제거 하 여 수요를 수동으로 또는 자동으로 일치 시킵니다.
- 네트워킹: 여러 호스트 컴퓨터에서 통신할 수 있도록 컨테이너를 조정 하는 오버레이 네트워크를 제공 합니다.
- 서비스 검색: 컨테이너가 호스트 컴퓨터 간에 이동하고 IP 주소를 변경하는 경우에도 컨테이너를 서로 자동으로 찾을 수 있습니다.
- 조정된 애플리케이션 업그레이드: 애플리케이션 작동 중지 시간을 피하도록 컨테이너 업그레이드를 관리하고 문제가 발생하는 경우 롤백합니다.

## <a name="orchestrator-types"></a>Orchestrator 유형

Azure는 AKS (Azure Kubernetes Service) 및 Service Fabric의 두 가지 컨테이너 orchestrator를 제공 합니다.

[AKS (Azure Kubernetes Service)](/azure/aks/) 를 사용 하면 컨테이너 화 된 응용 프로그램을 실행 하도록 미리 구성 된 가상 머신의 클러스터를 간편 하 게 만들고 구성 하 고 관리할 수 있습니다. 이를 통해 기존 기술을 사용 하 고 광범위 하 고 증가 하는 커뮤니티 전문 지식을 활용 하 여 Microsoft Azure에서 컨테이너 기반 응용 프로그램을 배포 하 고 관리할 수 있습니다. AKS를 사용 하 여 Kubernetes 및 Docker 이미지 형식을 통해 응용 프로그램 이식성을 유지 하면서 Azure의 엔터프라이즈급 기능을 활용할 수 있습니다.

[Azure Service Fabric](/azure/service-fabric/)은 신뢰할 수 있는 확장 가능한 마이크로 서비스 및 컨테이너를 쉽게 패키징하고 배포하고 관리할 수 있는 분산 시스템 플랫폼입니다. Service Fabric은 클라우드 원시 애플리케이션을 배포하고 관리하는 데 있어 중요한 과제를 해결합니다. 개발자와 관리자는 복잡한 인프라 문제를 방지하고, 확장 가능하고 신뢰할 수 있으며 관리가 용이한 중요 업무의 까다로운 작업을 구현하는 데 집중할 수 있습니다. Service Fabric은 컨테이너에서 실행되는 이러한 엔터프라이즈급, 계층 1, 클라우드 규모 애플리케이션을 구축하고 관리하는 차세대 플랫폼을 나타냅니다.

## <a name="getting-started"></a>시작

Azure Kubernetes service 배포를 시작 하려면 [Kubernetes 설치 가이드](../kubernetes/getting-started-kubernetes-windows.md)를 참조 하세요.

Azure Service Fabric 배포를 시작 하려면 [Service Fabric 빠른](/azure/service-fabric/service-fabric-quickstart-containers.md)시작을 참조 하세요.
