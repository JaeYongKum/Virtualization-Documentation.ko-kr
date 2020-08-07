---
title: Windows 컨테이너 네트워킹
description: Windows 컨테이너 내의 네트워크 격리 및 보안.
keywords: Docker, 컨테이너
author: jmesser81
ms.date: 03/27/2018
ms.topic: conceptual
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 5c60406c0cc839a84e25ff12abf53439c6a208cb
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985387"
---
# <a name="network-isolation-and-security"></a>네트워크 격리 및 보안

## <a name="isolation-with-network-namespaces"></a>네트워크 네임 스페이스와 격리

각 컨테이너 끝점은 자체 __네트워크 네임 스페이스__에 배치 됩니다. 관리 호스트 vNIC 및 호스트 네트워크 스택은 기본 네트워크 네임 스페이스에 있습니다. 동일한 호스트의 컨테이너 간에 네트워크 격리를 적용 하기 위해 각 Windows Server 컨테이너에 대해 네트워크 네임 스페이스를 만들고 컨테이너에 대 한 네트워크 어댑터를 설치할 Hyper-v 격리 상태에서 컨테이너를 실행 합니다. Windows Server 컨테이너는 호스트 vNIC를 사용하여 가상 스위치에 연결합니다. Hyper-v 격리는 가상 VM NIC (유틸리티 VM에 노출 되지 않음)를 사용 하 여 가상 스위치에 연결 합니다.

![text](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>네트워크 보안

사용 되는 컨테이너 및 네트워크 드라이버에 따라 포트 Acl은 Windows 방화벽과 [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/)의 조합에 의해 적용 됩니다.

### <a name="windows-server-containers"></a>Windows Server 컨테이너

Windows 호스트의 방화벽 (네트워크 네임 스페이스와 함께 지원) 및 VFP를 사용 합니다.

* 기본 아웃 바운드: 모두 허용
* 기본 인바운드: 모든 (TCP, UDP, ICMP, IGMP) 원치 않는 네트워크 트래픽 허용
  * 이러한 프로토콜이 아닌 다른 모든 네트워크 트래픽을 거부 합니다.

  >[!NOTE]
  >Windows Server, 버전 1709 및 Windows 10 크리에이터 크리에이터 업데이트 이전에는 기본 인바운드 규칙이 모두 거부 였습니다. 이러한 이전 릴리스를 실행 하는 사용자는 ``docker run -p`` (포트 전달)를 사용 하 여 인바운드 허용 규칙을 만들 수 있습니다.

### <a name="hyper-v-isolation"></a>Hyper-V 격리

Hyper-v 격리에서 실행 되는 컨테이너에는 자체 격리 된 커널이 있으므로 다음 구성을 사용 하 여 Windows 방화벽의 고유한 인스턴스를 실행 합니다.

* 기본값은 Windows 방화벽 (유틸리티 VM에서 실행) 및 VFP 모두에서 허용 됩니다.

![text](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes Pod

[Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)에서는 끝점이 연결 된 인프라 컨테이너가 먼저 생성 됩니다. 인프라 및 작업자 컨테이너를 포함 하 여 동일한 pod에 속한 컨테이너는 공통 네트워크 네임 스페이스 (동일한 IP 및 포트 공간)를 공유 합니다.

![text](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>기본 포트 Acl 사용자 지정

기본 포트 Acl을 수정 하려는 경우 호스트 네트워킹 서비스 설명서를 먼저 읽어 보세요 (곧 링크 추가). 다음 구성 요소 내에서 정책을 업데이트 해야 합니다.

>[!NOTE]
>투명 및 NAT 모드의 Hyper-v 격리의 경우 현재 기본 포트 Acl을 mk-reprogram 수 없습니다. 이는 테이블의 "X"에 의해 반영 됩니다.

| 네트워크 드라이버 | Windows Server 컨테이너 | Hyper-V 격리  |
| -------------- |-------------------------- | ------------------- |
| 투명 | Windows 방화벽 | X |
| NAT | Windows 방화벽 | X |
| L2Bridge | 모두 | VFP |
| L2Tunnel | 모두 | VFP |
| 오버레이  | 모두 | VFP |