---
title: Windows의 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: V1.12 사용 하 여 Kubernetes 클러스터에 Windows 노드를 가입합니다.
keywords: kubernetes, 1.12, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e43b2ac5b19d16721c1ba0dd1f34e339223bdaf
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178906"
---
# <a name="kubernetes-on-windows"></a>Windows의 Kubernetes #
이 페이지는 Linux 기반 클러스터에 Windows 노드를 가입 하 여 Windows의 Kubernetes를 사용 하 여 시작에 대 한 개요 역할을 합니다. Windows Server [버전 1803](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1803#kubernetes) 베타에서 Kubernetes 1.12 릴리스를 통해 사용자는 Windows의 Kubernetes에서 [최신 기능](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) 을 활용을 수행할 수 있습니다.

  - **간소화 된 네트워크 관리**: 호스트 게이트웨이 모드에서 Flannel를 사용 하 여 노드 간에 자동 경로 관리
  - **향상 된 확장성**: [Windows Server 컨테이너에 대 한 장치 없는 Vnic](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/) 덕분 빠르고 안정적이 되도록 컨테이너 시작 시간을 즐길 수
  - **hyper-v 격리 (알파)**: 향상 된 보안 ([Windows 컨테이너 형식](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types))에 대 한 커널 모드 격리를 사용 하 여 [hyper-v 설치 컨테이너](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) 조정 합니다.
  - **저장소 플러그 인**: Windows 컨테이너에 대 한 SMB 및 iSCSI 지원 [FlexVolume 저장소 플러그 인](https://github.com/Microsoft/K8s-Storage-Plugins) 을 사용

> [!TIP] 
> Azure에서 클러스터를 배포하려는 경우 오픈 소스 ACS-Engine 도구를 사용하면 간편합니다. 단계별 [연습](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md)을 볼 수 있습니다.

## <a name="prerequisites"></a>사전 요구 사항 ##

### <a name="plan-ip-addressing-for-your-cluster"></a>클러스터에 IP 주소 계획 ###
<a name="definitions"></a>Kubernetes 클러스터 포드 및 서비스에 대 한 새 서브넷 소개, 이므로 환경에서 다른 기존 네트워크와 충돌 중 확인 해야 합니다. 다음은 Kubernetes를 성공적으로 배포 하려면 확보 해야 하는 모든 주소 공간입니다.

| 서브넷 주소 범위 / | 설명 | 기본값 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**서비스 서브넷** | 라우팅할, 완전히 가상 서브넷 포드를 위해 포드에 없으며 네트워크 토폴로지에 대 한 고려 없이 서비스에 액세스 하는 데 사용 합니다. 노드에서 실행되는 `kube-proxy`에 의해 라우팅할 수 있는 주소 공간으로 또는 공간에서 변환됩니다. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**클러스터 서브넷** |  클러스터의 모든 포드에 의해 사용 되는 전역 서브넷입니다. 각 노드는 더 작은 /24 할당 된 서브넷 포드를 사용 하기 위해 여기에서 합니다. 클러스터에서 사용 되는 모든 포드를 수용 하기에 충분 이어야 합니다. *최소* 서브넷 크기를 계산 합니다. `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>노드당 100 포드는 5 개의 노드 클러스터에 대 한 예제: `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **Kubernetes DNS 서비스 IP** | DNS 해상도 및 클러스터 서비스 검색에 사용 되는 "kube dns" 서비스의 IP 주소입니다. | "10.96.0.10" |
> [!NOTE]
> Docker를 설치 하면 기본적으로 만들어졌으며 다른 Docker 네트워크 (NAT) 있습니다. Windows의 Kubernetes를 대신 클러스터 서브넷에서 Ip를 할당 하는 대로 작동 하는 데 필요 하지 않습니다.

### <a name="disable-anti-spoofing-protection"></a>스푸핑 방지 보호 사용 안 함 ###
> [!Important] 
> 성공적으로 Vm을 사용 하 여 오늘날 Windows의 Kubernetes를 배포 하려면 모든 사용자에 대해 필요한 대로이 섹션을 신중 하 게 읽어 바랍니다.

MAC 주소 스푸핑을 확인 하 고 Windows 컨테이너 호스트 Vm (게스트)에 대 한 가상화를 사용할 수 있습니다. 이 위해 (예: Hyper-v에 대 한) Vm을 호스팅하는 컴퓨터에서 관리자 권한으로 다음을 실행 해야 있습니다.

```powershell
Set-VMProcessor -VMName "<name>" -ExposeVirtualizationExtensions $true 
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```
> [!TIP]
> VMware 기반 제품 가상화 요구 사항에 맞게를 사용 하는 경우 MAC 스푸핑 요구 사항에 대해 [promiscuous 모드](https://kb.vmware.com/s/article/1004099) 활성화를 참조 하십시오.

>[!TIP]
> Azure IaaS Vm에서 Kubernetes를 직접 배포는,이 요구 사항에 대해 [중첩 된 가상화](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/) 를 지 원하는 Vm을 참조 하십시오.

## <a name="what-you-will-accomplish"></a>수행할 작업 ##

이 가이드에서는 다음을 수행합니다.

> [!div class="checklist"]
> * [Kubernetes 마스터](./creating-a-linux-master.md) 노드를 생성 합니다.  
> * [네트워크 솔루션](./network-topologies.md)을 선택 합니다.  
> * [Windows 작업자 노드](./joining-windows-workers.md) 또는 [Linux 작업자 노드](./joining-linux-workers.md) 를 가입 합니다.  
> * [샘플 Kubernetes 리소스](./deploying-resources.md)를 배포합니다.  
> * [일반적인 문제 및 실수](./common-problems.md) 검토

## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 중요 한 필수 조건 및 Windows의 Kubernetes 오늘날 성공적으로 배포 하는 데 필요한 가정 하는 방법에 대 한 가이드가 있습니다. Kubernetes 마스터를 설정 하는 방법을 알아보려면 계속 진행 합니다.

> [!div class="nextstepaction"]
> [Kubernetes 마스터 만들기](./creating-a-linux-master.md)