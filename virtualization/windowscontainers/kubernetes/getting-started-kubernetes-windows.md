---
title: Windows의 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: V1.14 사용 하 여 Kubernetes 클러스터에 Windows 노드를 가입합니다.
keywords: kubernetes, 1.14, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c380f5dc10430a94959718a5ce92f311603db733
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622928"
---
# <a name="kubernetes-on-windows"></a>Windows의 Kubernetes

이 페이지는 Linux 기반 클러스터에 Windows 노드를 가입 하 여 Windows에서 Kubernetes를 사용 하 여 시작에 대 한 개요 역할을 합니다. Windows Server [버전 1809에서](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)Kubernetes 1.14의 릴리스를 통해 사용자는 Windows에서 Kubernetes에서 다음 기능을 활용을 수행할 수 있습니다.

- **오버레이 네트워킹**: vxlan 모드에서 Flannel를 사용 하 여 가상 오버레이 네트워크를 구성 합니다.
    - 두 Windows Server 2019 [KB4489899](https://support.microsoft.com/help/4489899) 설치 또는 [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) 빌드 18317 + 필요
    - Kubernetes v1.14 필요 (이상)와 `WinOverlay` 기능 게이트 설정
    - Flannel v0.11.0 필요 (이상)
- **간소화 된 네트워크 관리**: 호스트 게이트웨이 모드에서 Flannel를 사용 하 여 노드 간에 자동 경로 관리 합니다.
- **향상 된 확장성**: [Windows Server 컨테이너에 대 한 장치 없는 Vnic](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716)덕분에 빠르고 안정적이 되도록 컨테이너 시작 시간을 개선 합니다.
- **Hyper-v 격리 (알파)**: 향상 된 보안을 위한 커널 모드 격리를 통해 [Hyper-v 격리](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) 를 조정 합니다. 자세한 내용은 [Windows 컨테이너 형식](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types)입니다.
    - Kubernetes v1.10 필요 (이상)와 `HyperVContainer` 기능 게이트 사용 하도록 설정 합니다.
- **저장소 플러그 인**: Windows 컨테이너에 대 한 SMB 및 iSCSI 지원 [FlexVolume 저장소 플러그 인](https://github.com/Microsoft/K8s-Storage-Plugins) 을 사용 합니다.

>[!TIP]
>Azure에서 클러스터를 배포 하려는 경우 오픈 소스 AKS 엔진 도구를 사용 하면 간편 합니다. 자세한 내용은 우리의 단계별 [연습](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)을 참조 하세요.

## <a name="prerequisites"></a>필수 구성 요소

### <a name="plan-ip-addressing-for-your-cluster"></a>클러스터에 IP 주소 계획

<a name="definitions"></a>Kubernetes 클러스터 포드 및 서비스에 대 한 새 서브넷 소개, 이므로 환경에서 다른 기존 네트워크와 충돌 중 확인 해야 합니다. 다음은 Kubernetes를 성공적으로 배포 하려면 확보 해야 하는 모든 주소 공간입니다.

| 서브넷 주소 범위 / | 설명 | 기본값 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**서비스 서브넷** | 액세스 하기 위해 포드에 서비스 없으며 네트워크 토폴로지에 대 한 고려 없이 포드에 의해 사용 되는 라우팅할, 완전히 가상 서브넷입니다. 노드에서 실행되는 `kube-proxy`에 의해 라우팅할 수 있는 주소 공간으로 또는 공간에서 변환됩니다. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**클러스터 서브넷** |  클러스터의 모든 포드에 의해 사용 되는 글로벌 서브넷입니다. 각 노드는 더 작은 /24 할당 포드를 사용 하기 위해 여기에서 서브넷 합니다. 클러스터에 사용 되는 모든 포드를 수용 하기에 충분 이어야 합니다. *최소* 서브넷 크기를 계산 합니다. `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>5 개의 노드 클러스터 노드당 100 포드에 대 한 예제: `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **Kubernetes DNS 서비스 IP** | DNS 확인 & 클러스터 서비스 검색에 사용 되는 "kube dns" 서비스의 IP 주소입니다. | "10.96.0.10" |

> [!NOTE]
> Docker를 설치 하면 기본적으로 만들어졌으며 다른 Docker 네트워크 (NAT) 있습니다. Windows에서 Kubernetes 대신 클러스터 서브넷에서 Ip를 할당 하는 대로 작동 하는 데 필요 하지 않습니다.

## <a name="what-you-will-accomplish"></a>수행할 작업

이 가이드에서는 다음을 수행합니다.

> [!div class="checklist"]
> * [Kubernetes 마스터](./creating-a-linux-master.md) 노드를 생성 합니다.  
> * [네트워크 솔루션](./network-topologies.md)을 선택 합니다.  
> * [Windows 작업자 노드](./joining-windows-workers.md) 또는 [Linux 작업자 노드](./joining-linux-workers.md) 를 가입 합니다.  
> * [샘플 Kubernetes 리소스](./deploying-resources.md)를 배포합니다.  
> * [일반적인 문제 및 실수](./common-problems.md) 검토

## <a name="next-steps"></a>다음 단계

이 섹션에서는 기술인 중요 한 필수 조건 성공적으로 현재 Windows에서 Kubernetes를 배포 하는 데 필요한 & 가정 합니다. Kubernetes 마스터를 설정 하는 방법을 알아보려면 계속 진행 합니다.

>[!div class="nextstepaction"]
>[Kubernetes 마스터 만들기](./creating-a-linux-master.md)