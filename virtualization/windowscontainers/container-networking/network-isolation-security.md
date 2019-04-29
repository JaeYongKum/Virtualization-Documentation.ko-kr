---
title: Windows 컨테이너 네트워킹
description: Windows 컨테이너 내 네트워크 격리 및 보안
keywords: Docker, 컨테이너
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 1c0a3fd25a5572604db59e0c68d8b4a3d84b00e9
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576314"
---
# <a name="network-isolation-and-security"></a>네트워크 격리 및 보안

## <a name="isolation-with-network-namespaces"></a>네트워크 네임 스페이스를 사용 하 여 격리

각 컨테이너 끝점은 자체 __네트워크 네임스페이스__에 배치됩니다. 관리 호스트 vNIC 및 호스트 네트워크 스택은 기본 네트워크 네임스페이스에 배치됩니다. 동일한 호스트에서 컨테이너 간에 네트워크 격리를 적용 하기 위해 네트워크 네임 스페이스에 대 한 각 Windows Server 컨테이너 만들어지고 컨테이너는 컨테이너에 대 한 네트워크 어댑터가 설치 되는 Hyper-v 격리에서 실행 합니다. Windows Server 컨테이너는 호스트 vNIC를 사용하여 가상 스위치에 연결합니다. Hyper-v 격리는 가상 VM NIC (유틸리티 VM에 노출 되지 않음)를 사용 하 여 가상 스위치에 연결 합니다.

![텍스트](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>네트워크 보안

사용되는 컨테이너 및 네트워크 드라이버에 따라 Windows 방화벽과 [VFP](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/)의 조합으로 포트 ACL이 적용됩니다.

### <a name="windows-server-containers"></a>Windows Server 컨테이너

이들은 Windows 호스트의 방화벽(네트워크 네임스페이스를 통해 인식)을 비롯해 VFP를 사용

* 기본 아웃바운드: 모두 허용
* 기본 인바운드: 원치 않는 모든 네트워크 트래픽(TCP, ICMP, UDP IGMP) 허용
  * 이러한 프로토콜에서 나오지 않은 기타 모든 네트워크 트래픽 거부

  >[!NOTE]
  >Windows Server, 버전 1709 및 Windows 10 Fall Creators Update, 이전 기본 인바운드 규칙에는 모든 거부 되었습니다. 이러한 이전 버전을 실행 하는 사용자가 사용 하 여 인바운드 허용 규칙을 만들 수 있습니다 ``docker run -p`` (포트 전달).

### <a name="hyper-v-isolation"></a>Hyper-V 격리

Hyper-v 격리 된 상태에서 실행 되는 컨테이너는 자체적으로 격리 된 커널을 있고 따라서 다음 구성을 사용 하 여 Windows 방화벽의 자체 인스턴스를 실행 합니다.

* Windows 방화벽(유틸리티 VM에서 실행 중)과 VFP 모두에서 모두 허용이 기본 적용

![텍스트](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes 포드

[Kubernetes 포드](https://kubernetes.io/docs/concepts/workloads/pods/pod/)를 인프라 컨테이너가 먼저 만들어집니다 끝점이 연결 합니다. 인프라 및 작업자 컨테이너 포함 하 여, 동일한 포드에 속하는 컨테이너는 일반적인 네트워크 네임 스페이스 (동일한 IP 및 포트 공간)을 공유 합니다.

![텍스트](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>기본 포트 ACL 사용자 지정

기본 포트 Acl을 먼저 호스트 네트워킹 서비스 설명서를 참조 하십시오 (링크를 곧 추가 될 예정)를 수정 해야 합니다. 다음 구성 요소 내에서 정책을 업데이트 해야 합니다.

>[!NOTE]
>투명 및 NAT 모드의 Hyper-v 격리를 하면 현재 다시 프로그래밍할 수 없습니다 기본 포트 Acl 합니다. 이는 테이블의 "X"에 반영되어 있습니다.

| 네트워크 드라이버 | Windows Server 컨테이너 | Hyper-V 격리  |
| -------------- |-------------------------- | ------------------- |
| 투명 | Windows 방화벽 | X |
| NAT | Windows 방화벽 | X |
| L2Bridge | 둘 모두 | VFP |
| L2Tunnel | 둘 모두 | VFP |
| 오버레이  | 둘 모두 | VFP |