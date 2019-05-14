---
title: Windows 컨테이너 네트워킹
description: Windows 컨테이너용 네트워크 드라이버 및 토폴로지.
keywords: Docker, 컨테이너
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 40e877c8999574f21ecb9586c3f2bc012607177f
ms.sourcegitcommit: 40b929dbc72aa308d8e46765ac61616a35b31791
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/14/2019
ms.locfileid: "9634392"
---
# <a name="windows-container-network-drivers"></a>Windows 컨테이너 네트워크 드라이버  

Windows에서 Docker를 사용하여 만든 기본 'nat' 네트워크를 활용하는 것 외에도 사용자는 사용자 지정 컨테이너 네트워크를 정의할 수 있습니다. 사용자 정의 네트워크는 Docker CLI[`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) 명령을 사용하여 만들 수 있습니다. Windows에서 다음과 같은 유형의 네트워크 드라이버를 사용할 수 있습니다.

- **nat** – 'nat' 드라이버를 통해 생성된 네트워크에 연결된 컨테이너는 *내부* Hyper-V 스위치에 연결되고 사용자 지정(``--subnet``) IP 접두사에서 IP 주소를 수신합니다. 컨테이너 호스트에서 컨테이너 끝점으로의 포트 전달/매핑이 지원됩니다.
  
  >[!NOTE]
  > Windows Server 2019에서 (또는 위에) 만들어진 NAT 네트워크 다시 부팅 한 후 더 이상 유지 됩니다.

  > Windows 10 크리에이터 스 업데이트가 설치 되어 있는 경우 (이상) 여러 NAT 네트워크가 지원 됩니다.
  
- **transparent** – 'transparent' 드라이버를 통해 생성된 네트워크에 연결된 컨테이너가 *외부* Hyper-V 스위치를 통해 실제 네트워크에 직접 연결됩니다. 실제 네트워크의 IP는 외부 DHCP 서버를 사용하여 정적으로(사용자가 지정한 ``--subnet`` 옵션이 필요함) 또는 동적으로 할당할 수 있습니다.
  
  >[!NOTE]
  >다음 요구 사항으로 인해 컨테이너 호스트 투명 네트워크를 통해 연결 지원 되지 않습니다 Azure Vm에서.
  
  > 필요 합니다:이 모드를 사용할 경우 가상화 시나리오에서 (컨테이너 호스트는 VM) _MAC 주소 스푸핑이 필요_합니다.

- **overlay** - Docker 엔진이 [Swarm 모드](../manage-containers/swarm-mode.md)에서 실행 중일 때, 오버레이 네트워크에 연결된 컨테이너는 여러 컨테이너 호스트에서 동일한 네트워크에 연결된 다른 컨테이너들과 통신이 가능합니다. Swarm 클러스터에 만들어지는 각 오버레이 네트워크는 개인 IP 접두사에 의해 정의된 자체 IP 서브넷으로 만들어집니다. 오버레이 네트워크 드라이버는 VXLAN 캡슐화를 사용합니다. **적합한 네트워크 제어 평면(Flannel 또는 OVN)을 사용할 때 Kubernetes에서도 사용할 수 있습니다.**
  > 필요: 귀하의 환경이 오버레이 네트워크를 만들기 위해 이러한 필요한 [필수 구성 요소](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) 있는지 확인 합니다.

  > [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217), Windows 10 크리에이터 스 업데이트 또는 이후 릴리스를 사용 하 여 Windows Server 2016에서는 필요 합니다.

  >[!NOTE]
  >Docker EE 18.03를 실행 하는 Windows Server 2019 이상에서 Docker Swarm에서 만든 오버레이 네트워크에 대 한 아웃 바운드 연결 VFP NAT 규칙을 활용 합니다. 즉, 1 IP 주소를 수신 하는 thata 컨테이너를 제공 합니다. 또한는 같은 도구 ICMP 기반 의미 `ping` 또는 `Test-NetConnection` 디버깅 상황에서 TCP/UDP 옵션을 사용 하 여 구성 해야 합니다.

- **l2bridge** - 'l2bridge' 드라이버를 사용하여 만든 네트워크에 연결된 컨테이너는 컨테이너 호스트와 동일한 IP 서브넷에 있게 될 것이며, *외부* Hyper-V 스위치를 통해 실제 네트워크에 연결됩니다. IP 주소는 컨테이너 호스트와 동일한 접두사에서 정적으로 할당해야 합니다. 호스트의 모든 컨테이너 끝점은 수신 및 송신 시 계층 2 주소 변환(MAC 다시 쓰기) 작업으로 인해 호스트와 동일한 MAC 주소를 갖게 됩니다.
  > Windows Server 2016, Windows 10 크리에이터 스 업데이트 이상이 필요 합니다:이 필요합니다.

  > 필요: 외부 연결에 대 한 경우 [OutboundNAT 정책](./advanced.md#specify-outboundnat-policy-for-a-network) 입니다.

- **l2tunnel** - 그러나 l2bridge와 유사하게 _이 드라이버는 Microsoft 클라우드 스택에서만 사용해야 합니다_. 컨테이너에서 제공한 패킷은 SDN 정책이 적용되는 가상화 호스트로 전송됩니다.


## <a name="network-topologies-and-ipam"></a>네트워크 토폴로지 및 IPAM

아래 표에서는 각 네트워크 드라이버의 내부(컨테이너 간) 및 외부 연결에 대한 네트워크 연결을 제공하는 방법을 보여 줍니다.

### <a name="networking-modesdocker-drivers"></a>네트워킹 모드/Docker 드라이버

  | Docker Windows 네트워크 드라이버 | 일반적인 os | 컨테이너-컨테이너 (단일 노드) | 컨테이너-외부 (단일 노드 + 다중 노드) | 컨테이너-컨테이너 (다중 노드) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT(기본)** | 개발자에게 적합 | <ul><li>동일 서브넷: Hyper-V 가상 스위치를 통해 브리지된 연결</li><li> 크로스 서브넷: 지원 되지 않습니다 (오직 하나만 NAT 내부 접두사)</li></ul> | 관리 vNIC를 통해 라우트(WinNAT에 바인딩) | 직접 지원 되지 않음: 호스트 통한 포트 노출이 필요 |
  | **transparent** | 개발자나 소규모 배포에 적합 | <ul><li>동일 서브넷: Hyper-V 가상 스위치를 통해 브리지된 연결</li><li>크로스 서브넷: 컨테이너 호스트를 통해 라우트</li></ul> | 컨테이너 호스트를 라우트 되어 (물리적) 네트워크 어댑터에 직접 액세스 | 컨테이너 호스트를 라우트 되어 (물리적) 네트워크 어댑터에 직접 액세스 |
  | **오버레이** | 다중 노드;에 적합 합니다. Docker swarm, Kubernetes에서 사용할 수 있는 필요 | <ul><li>동일 서브넷: Hyper-V 가상 스위치를 통해 브리지된 연결</li><li>크로스 서브넷: 네트워크 트래픽이 관리 vNIC를 통해 캡슐화 및 라우트</li></ul> | 직접 지원되지 않음 - NAT 네트워크에 연결된 두 번째 컨테이너 끝점이 필요 | 동일 서브넷/크로스 서브넷: VXLAN을 사용해 네트워크 트래픽이 캡슐화되고 관리 vNIC를 통해 라우트 |
  | **L2Bridge** | Kubernetes 및 Microsoft SDN에 사용 | <ul><li>동일 서브넷: Hyper-V 가상 스위치를 통해 브리지된 연결</li><li> 크로스 서브넷: 수신 및 송신 시 컨테이너 MAC 주소가 재작성되어 라우트</li></ul> | 수신 및 송신 시 컨테이너 MAC 주소가 재작성되어 라우트 | <ul><li>동일 서브넷: 브리지된 연결</li><li>크로스 서브넷: WSv1709 및 위에 관리 vNIC를 통해 라우트</li></ul> |
  | **L2Tunnel**| Azure에만 해당 | 동일/크로스 서브넷: 정책이 적용되는 물리적 호스트의 Hyper-V 가상 스위치에 고정 | 트래픽은 Azure 가상 네트워크 게이트웨이를 반드시 통해야 함 | 동일/크로스 서브넷: 정책이 적용되는 물리적 호스트의 Hyper-V 가상 스위치에 고정 |

### <a name="ipam"></a>IPAM

각 네트워킹 드라이버에 서로 다른 IP 주소가 할당됩니다. Windows는 HNS(호스트 네트워킹 서비스)를 사용하여 nat 드라이버에 대한 IPAM을 제공하고 Docker Swarm 모드(내부 KVS)에서 작동하여 오버레이에 대한 IPAM을 제공합니다. 다른 네트워크 드라이버는 외부 IPAM을 사용합니다.

| 네트워킹 모드/드라이버 | IPAM |
| -------------------------|:----:|
| NAT | 동적 IP 할당 및 내부 NAT 서브넷 접두사에서 호스트 네트워킹 서비스 (HNS) 지정 |
| transparent | 컨테이너 호스트의 네트워크 접두사 내의 IP 주소에서 정적 또는 동적(외부 DHCP 서버 사용) IP 할당 및 지정 |
| 오버레이 | Docker 엔진 Swarm 모드에서 관리되는 접두사에서 동적 IP 할당 및 HNS 통한 지정 |
| L2Bridge | (또한 할당 될 수 HNS 통한) 컨테이너 호스트의 네트워크 접두사 내의 IP 주소에서 정적 IP 할당 및 지정 |
| L2Tunnel | Azure에만 해당 - 플러그에서 동적 IP 할당 및 지정 |

### <a name="service-discovery"></a>서비스 검색

서비스 검색은 특정 Windows 네트워크 드라이버에만 지원됩니다.

|  | 로컬 서비스 검색  | 글로벌 서비스 검색 |
| :---: | :---------------     |  :---                |
| nat | 예 | Docker EE에서 예 |  
| overlay | 예 | Docker EE 또는 kube dns 예 |
| transparent | NO | 아니요 |
| l2bridge | 아니요 | Kube-dns에서 예 |
