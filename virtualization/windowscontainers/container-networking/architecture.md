---
title: Windows 컨테이너 네트워킹
description: Gentle는 Windows 컨테이너 네트워크의 아키텍처를 소개 합니다.
keywords: Docker, 컨테이너
author: daschott
ms.author: jgerend
ms.date: 08/13/2020
ms.topic: overview
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 71228abea66084f9de1ee29daaff1d3d3f1ffc40
ms.sourcegitcommit: 160405a16d127892b6e2897efa95680f29f0496a
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/22/2020
ms.locfileid: "90990876"
---
# <a name="windows-container-networking"></a>Windows 컨테이너 네트워킹

>[!IMPORTANT]
>일반 docker 네트워킹 명령, 옵션 및 구문에 대 한 [Docker 컨테이너 네트워킹](https://docs.docker.com/engine/userguide/networking/) 을 참조 하세요. [지원 되지 않는 기능 및 네트워크 옵션](#unsupported-features-and-network-options)에 설명 된 경우를 제외 하 고 모든 Docker 네트워킹 명령은 Linux의 구문과 동일한 구문을 사용 하 여 Windows에서 지원 됩니다. 그러나 Windows 및 Linux 네트워크 스택은 서로 다르며 Windows에서 일부 Linux 네트워크 명령 (예: ifconfig)이 지원 되지 않는 것을 알 수 있습니다.

## <a name="basic-networking-architecture"></a>기본 네트워킹 아키텍처

이 항목에서는 Docker가 Windows에서 호스트 네트워크를 만들고 관리 하는 방법에 대 한 개요를 제공 합니다. Windows 컨테이너는 네트워킹에 관해서 가상 컴퓨터와 유사하게 작동합니다. 각 컨테이너에는 가상 네트워크 어댑터 (vNIC)가 있으며,이는 Hyper-v 가상 스위치 (vSwitch)에 연결 됩니다. Windows에서는 Docker를 통해 만들 수 있는 5 가지 [네트워킹 드라이버 또는 모드](./network-drivers-topologies.md) ( *nat*, *오버레이*, *투명*, *l2bridge*및 *l2tunnel*)를 지원 합니다. 실제 네트워크 인프라 및 단일 vs 다중 호스트 네트워킹 요구 사항에 따라 사용자의 요구에 가장 적합 한 네트워크 드라이버를 선택 해야 합니다.

![text](media/windowsnetworkstack-simple.png)

Docker 엔진을 처음 실행 하면 기본 NAT 네트워크 (' nat ')가 생성 됩니다 .이 네트워크는 내부 vSwitch와 이라는 Windows 구성 요소를 사용 `WinNAT` 합니다. PowerShell 또는 Hyper-v 관리자를 통해 만든 기존 외부 vSwitches 호스트에 있는 경우 *투명* 네트워크 드라이버를 사용 하 여 Docker에도 사용할 수 있으며 명령을 실행할 때 볼 수 있습니다 ``docker network ls`` .

![text](media/docker-network-ls.png)

- **내부** vSwitch는 컨테이너 호스트의 네트워크 어댑터에 직접 연결 되지 않는 항목입니다.
- **외부** vSwitch는 컨테이너 호스트의 네트워크 어댑터에 직접 연결 된 vSwitch입니다.

![text](media/get-vmswitch.png)

' Nat ' 네트워크는 Windows에서 실행 되는 컨테이너에 대 한 기본 네트워크입니다. 특정 네트워크 구성을 구현 하기 위한 플래그 또는 인수 없이 Windows에서 실행 되는 모든 컨테이너는 기본 ' nat ' 네트워크에 연결 되며 ' nat ' 네트워크의 내부 접두사 IP 범위에서 IP 주소가 자동으로 할당 됩니다. ' N a m e '에 사용 되는 기본 내부 IP 접두사는 172.16.0.0/16입니다.

## <a name="container-network-management-with-host-network-service"></a>호스트 네트워크 서비스를 사용 하 여 컨테이너 네트워크 관리

HNS (호스트 네트워킹 서비스)와 HCS (호스트 계산 서비스)는 함께 작동 하 여 컨테이너를 만들고 네트워크에 끝점을 연결 합니다.

### <a name="network-creation"></a>네트워크 만들기

- HNS는 각 네트워크에 대 한 Hyper-v 가상 스위치를 만듭니다.
- HNS는 필요에 따라 NAT 및 IP 풀을 만듭니다.

### <a name="endpoint-creation"></a>끝점 만들기

- HNS는 컨테이너 엔드포인트 당 네트워크 네임 스페이스를 만듭니다.
- HNS/HCS 네트워크 네임 스페이스 내에 v (m) NIC 배치
- HNS에서 (vSwitch) 포트를 만듭니다.
- HNS는 끝점에 IP 주소, DNS 정보, 경로 등 (네트워킹 모드에 따름)을 할당 합니다.

### <a name="policy-creation"></a>정책 만들기

- 기본 NAT 네트워크: HNS는 해당 Windows 방화벽 허용 규칙을 사용 하 여 WinNAT 포트 전달 규칙/매핑을 만듭니다.
- 다른 모든 네트워크: HNS는 VFP (가상 필터링 플랫폼)를 활용 하 여 정책 만들기
    - 여기에는 부하 분산, Acl, 캡슐화 등이 포함 됩니다.
    - [여기](/windows-server/networking/technologies/hcn/hcn-top) 에 게시 된 HNS api 및 스키마를 찾습니다.

![text](media/HNS-Management-Stack.png)

## <a name="unsupported-features-and-network-options"></a>지원 되지 않는 기능 및 네트워크 옵션

다음 네트워킹 옵션은 현재 Windows에서 지원 **되지 않습니다** .

- L2bridge, NAT 및 오버레이 네트워크에 연결 된 Windows 컨테이너는 IPv6 스택을 통한 통신을 지원 하지 않습니다.
- IPsec을 통해 암호화 된 컨테이너 통신.
- 컨테이너에 대 한 HTTP 프록시 지원.
- [호스트 모드](https://docs.docker.com/ee/ucp/interlock/config/host-mode-networking/) 네트워킹
- 투명 한 네트워크 드라이버를 통해 가상화 된 Azure 인프라에서 네트워킹.

| 명령        | 지원 되지 않는 옵션   |
|---------------|:--------------------:|
| ``docker run``|   ``--ip6``, ``--dns-option`` |
| ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |