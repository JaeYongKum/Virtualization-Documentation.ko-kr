---
title: Windows 컨테이너 네트워킹
description: Windows 컨테이너 네트워크의 아키텍처를 간단하게 소개합니다.
keywords: Docker, 컨테이너
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 6cf35208cfcec313cfdd17e6ecef9c72050b85ad
ms.sourcegitcommit: 1715411ac2768159cd9c9f14484a1cad5e7f2a5f
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/26/2019
ms.locfileid: "9263480"
---
# <a name="windows-container-networking"></a>Windows 컨테이너 네트워킹
> ***보증의 부인: 일반 Docker 네트워킹 명령, 옵션 및 구문에 대한 내용은 [Docker 컨테이너 네트워킹](https://docs.docker.com/engine/userguide/networking/)을 참조하세요.*** [아래](#unsupported-features-and-network-options)에 설명된 경우를 제외한 모든 Docker 네트워킹 명령은 Linux에서와 동일한 구문을 통해 Windows에서 지원됩니다. 그러나 Windows와 Linux의 네트워크 스택이 서로 다르므로 일부 Linux 네트워크 명령은 Windows에서 지원되지 않습니다(예: ifconfig).


## <a name="basic-networking-architecture"></a>기본 네트워킹 아키텍처
이 항목에서는 Docker가 Windows에서 호스트 네트워크를 만들고 관리하는 방식에 대해 개략적으로 설명합니다. Windows 컨테이너는 네트워킹에 있어서 가상 컴퓨터와 유사하게 작동합니다. 각 컨테이너에는 Hyper-V 가상 스위치(vSwitch)에 연결되는 가상 네트워크 어댑터(vNIC)가 있습니다. Windows는 Docker를 통해 만들 수 있는 5가지 [네트워킹 드라이버 또는 모드](./network-drivers-topologies.md)인 *nat*, *overlay*, *transparent*, *l2bridge*, *l2tunnel*을 지원합니다. 실제 네트워크 인프라와 단일 및 다중 호스트 네트워킹 요구 사항에 따라 가장 적합한 네트워크 드라이버를 선택해야 합니다.


![텍스트](media/windowsnetworkstack-simple.png)


Docker 엔진은 처음으로 실행되면 내부 vSwitch와 `WinNAT`이라고 하는 Windows 구성 요소를 사용하는 기본 NAT 네트워크 'nat'를 만듭니다. 호스트에 PowerShell 또는 Hyper-V 관리자를 통해 만든 기존 외부 vSwitch가 있는 경우 이러한 외부 스위치 역시 *transparent* 네트워크 드라이버를 사용하여 Docker에 제공되며 ``docker network ls`` 명령을 실행할 때 볼 수 있습니다.  


![텍스트](media/docker-network-ls.png)


> - ***내부*** vSwitch는 컨테이너 호스트의 네트워크 어댑터에 직접 연결되지 않습니다. 

> - ***외부*** vSwitch는 컨테이너 호스트의 네트워크 어댑터에 _직접_ 연결됩니다.  


![텍스트](media/get-vmswitch.png)


'nat' 네트워크는 Windows에서 실행되는 컨테이너의 기본 네트워크입니다. 특정 네트워크 구성을 구현하는 플래그 또는 인수 없이 Windows에서 실행되는 모든 컨테이너는 기본 'nat' 네트워크에 연결되고, 'nat' 네트워크 내부 접두사 IP 범위의 IP 주소가 자동으로 할당합니다. 'nat'에 사용되는 기본 내부 IP 접두사는 172.16.0.0/16입니다. 


## <a name="container-network-management-with-host-network-service"></a>호스트 네트워크 서비스를 통해 컨테이너 네트워크 관리

HNS(호스트 네트워킹 서비스)와 HCS(호스트 계산 서비스)가 함께 작동하여 컨테이너를 만들고 네트워크에 끝점을 연결합니다.

### <a name="network-creation"></a>네트워크 만들기
  - HNS는 각 네트워크에 대해 Hyper-V 가상 스위치를 생성
  - HNS는 필요에 따라 NAT 및 IP 풀을 생성

### <a name="endpoint-creation"></a>끝점 생성
  - HNS는 컨테이너 끝점마다 네트워크 네임스페이스를 만듭니다.
  - HNS/HCS는 네트워크 네임스페이스 내에 v(m)NIC 배치
  - HNS는 (vSwitch) 포트 생성
  - HNS는 끝점에 IP 주소, DNS 정보, 경로 등(네트워킹 모드에 따라 다름)을 할당합니다.

### <a name="policy-creation"></a>정책 만들기
  - 기본 NAT 네트워크: HNS는 해당되는 Windows 방화벽 허용 규칙에 따라 WinNAT 포트 전달 규칙/매핑을 생성합니다.
  - 다른 모든 네트워크: HNS는 정책 생성을 위해 가상 필터링 플랫폼(VFP)을 이용합니다.
    - 여기에는 부하 분산, ACL, 캡슐화 등이 포함되어 있습니다.
    - **곧 게시될** Microsoft의 HNS API 및 스키마를 찾아보세요.


![텍스트](media/HNS-Management-Stack.png)


 ## <a name="unsupported-features-and-network-options"></a>지원되지 않는 기능과 네트워크 옵션
 다음 네트워킹 옵션은 현재 **하지** Windows에서 지원 됩니다.
   * L2bridge, NAT 및 오버레이 네트워크에 연결 된 Windows 컨테이너 IPv6 스택을 통해 통신 하는 것을 지원 하지 않습니다.
   * IPsec 통해 통신 암호화 된 컨테이너입니다.
   * 컨테이너에 대 한 HTTP 프록시 지원 합니다.
   * Hyper-v 컨테이너를 실행 하도록 끝점을 연결 (핫 애드).
   * 투명 네트워크 드라이버를 통해 Azure 가상화 인프라에 네트워킹.

 | 명령        | 지원되지 않는 옵션   |
 | ---------------|:--------------------:|
 | ``docker run``|   ``--ip6``, ``--dns-option`` |
 | ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |