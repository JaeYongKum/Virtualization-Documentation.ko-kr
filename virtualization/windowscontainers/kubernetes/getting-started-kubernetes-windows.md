---
title: Windows의 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: V 1.14를 사용 하 여 Windows 노드를 Kubernetes 클러스터에 가입 합니다.
keywords: kubernetes, 1.14, windows, 시작 하기
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 18734f102042ec951255061dcd82229e18d29a15
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998390"
---
# <a name="kubernetes-on-windows"></a>Windows의 Kubernetes

이 페이지는 windows 노드를 Linux 기반 클러스터에 가입 하 여 Windows에서 Kubernetes를 시작 하기 위한 개요 역할을 합니다. Windows Server [버전 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)의 Kubernetes 1.14 릴리스를 사용 하면 사용자가 Windows의 Kubernetes에서 다음 기능을 활용할 수 있습니다.

- **오버레이 네트워킹**: vxlan 모드에서 Flannel를 사용 하 여 가상 오버레이 네트워크 구성
    - [KB4489899](https://support.microsoft.com/help/4489899) 가 설치 된 windows server 2019 또는 [Windows Server vnext Insider preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) 빌드 18317 +
    - 기능 게이트를 사용 하는 `WinOverlay` 경우 Kubernetes v 1.14 (이상)가 필요 합니다.
    - Flannel v 0.11.0 필요 (이상)
- **단순화 된 네트워크 관리**: 노드 간 자동 경로 관리를 위해 호스트 게이트웨이 모드에서 Flannel를 사용 합니다.
- **확장성 향상**: [Windows Server 컨테이너에 대 한 vnics를 deviceless](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716)하 여 더욱 빠르고 안정적인 컨테이너 시작 시간을 즐기세요.
- **Hyper-v 격리 (alpha)**: 강화 된 보안을 위해 커널 모드 격리를 사용 하 여 [hyper-v 격리](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) 를 조정 합니다. 자세한 내용은 [Windows 컨테이너 유형을](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types)입력 하세요.
    - 기능 게이트를 사용할 수 있는 `HyperVContainer` Kubernetes v 1.10 (이상)이 필요 합니다.
- **저장소 플러그 인**: Windows 컨테이너에 대 한 SMB 및 iSCSI 지원과 함께이 [볼륨 저장소 플러그](https://github.com/Microsoft/K8s-Storage-Plugins) 인을 사용 합니다.

>[!TIP]
>Azure에서 클러스터를 배포 하려는 경우 원본 공개 AKS 도구를 사용 하면이 작업이 간편 합니다. 자세한 내용은 단계별 [연습](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)을 참조 하세요.

## <a name="prerequisites"></a>필수 구성 요소

### <a name="plan-ip-addressing-for-your-cluster"></a>클러스터에 대 한 IP 주소 지정 계획

<a name="definitions"></a>Kubernetes 클러스터는 pods 및 서비스에 대 한 새 서브넷을 도입할 때 환경의 다른 기존 네트워크와 충돌 하지 않도록 하는 것이 중요 합니다. Kubernetes를 성공적으로 배포 하기 위해 확보 해야 하는 모든 주소 공간은 다음과 같습니다.

| 서브넷/주소 범위 | 설명 | 기본값 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**서비스 서브넷** | Pods에서 네트워크 토폴로지에 대 한 caring 없이 uniformally 액세스 서비스를 사용 하는 라우팅할 수 없는 전용 가상 서브넷입니다. 노드에서 실행되는 `kube-proxy`에 의해 라우팅할 수 있는 주소 공간으로 또는 공간에서 변환됩니다. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**클러스터 서브넷** |  클러스터의 모든 pods에서 사용 하는 글로벌 서브넷입니다. 각 노드에는 pods에서 사용할 수 있는 것 보다 작은/24 개의 서브넷이 할당 됩니다. 클러스터에 사용 되는 모든 pods을 수용할 수 있을 만큼 커야 합니다. *최소* 서브넷 크기를 계산 하려면: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Pods의 5 노드 클러스터에 대 한 예: 노드당 100 `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **Kubernetes DNS 서비스 IP** | 클러스터 서비스 검색 & DNS 확인에 사용 되는 "kube-dns" 서비스의 IP 주소입니다. | "10.96.0.10" |

> [!NOTE]
> Docker를 설치할 때 기본적으로 만들어지는 또 다른 Docker 네트워크 (NAT)가 있습니다. 대신 클러스터 서브넷에서 Ip를 할당할 때 Windows에서 Kubernetes를 작동 하는 것은 필요 하지 않습니다.

## <a name="what-you-will-accomplish"></a>수행할 작업

이 가이드에서는 다음을 수행합니다.

> [!div class="checklist"]
> * [Kubernetes 마스터](./creating-a-linux-master.md) 노드를 만들었습니다.  
> * [네트워크 솔루션](./network-topologies.md)을 선택 했습니다.  
> * [Windows worker 노드](./joining-windows-workers.md) 또는 [Linux 작업자 노드에](./joining-linux-workers.md) 참가 했습니다.  
> * [샘플 Kubernetes 리소스](./deploying-resources.md)를 배포 했습니다.  
> * [일반적인 문제 및 실수](./common-problems.md) 검토

## <a name="next-steps"></a>다음 단계

이 섹션에서는 지금 Windows에 Kubernetes를 배포 하는 데 필요한 중요 한 필수 조건 & 가정을 설명 했습니다. Kubernetes 마스터를 설정 하는 방법을 계속 학습 합니다.

>[!div class="nextstepaction"]
>[Kubernetes 마스터 만들기](./creating-a-linux-master.md)