---
title: Windows 컨테이너 네트워킹
description: Windows 컨테이너의 네트워크 드라이버 및 토폴로지
keywords: Docker, 컨테이너
author: jmesser81
ms.date: 03/27/2018
ms.topic: conceptual
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: c7be67f3c114e7f6b122dffff399b2c292c26369
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192173"
---
# <a name="windows-container-network-drivers"></a>Windows 컨테이너 네트워크 드라이버

Windows에서 Docker에서 만든 기본 ' nat ' 네트워크를 활용 하는 것 외에도 사용자는 사용자 지정 컨테이너 네트워크를 정의할 수 있습니다. 사용자 정의 네트워크는 Docker CLI 명령을 사용 하 여 만들 수 있습니다 [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) . Windows에서는 다음 네트워크 드라이버 유형을 사용할 수 있습니다.

- **nat** – ' nat ' 드라이버로 만든 네트워크에 연결 된 컨테이너는 *내부* hyper-v 스위치에 연결 되 고 사용자 지정 () ip 접두사에서 ip 주소를 수신 ``--subnet`` 합니다. 컨테이너 호스트에서 컨테이너 엔드포인트으로의 포트 전달/매핑이 지원됩니다.

  >[!NOTE]
  > Windows Server 2019 이상에서 만든 NAT 네트워크는 다시 부팅 후 더 이상 지속 되지 않습니다.

  > Windows 10 크리에이터 업데이트 (이상)를 설치한 경우 여러 NAT 네트워크가 지원 됩니다.

- **투명** – ' 투명 ' 드라이버로 만든 네트워크에 연결 된 컨테이너는 *외부* hyper-v 스위치를 통해 실제 네트워크에 직접 연결 됩니다. 실제 네트워크의 Ip를 정적으로 할당 하거나 (사용자 지정 옵션 필요 ``--subnet`` ) 외부 DHCP 서버를 사용 하 여 동적으로 할당할 수 있습니다.

  >[!NOTE]
  >다음 요구 사항으로 인해 투명 한 네트워크를 통해 컨테이너 호스트를 연결 하는 것은 Azure Vm에서 지원 되지 않습니다.

  > 필요:이 모드가 가상화 시나리오에서 사용 되는 경우 (컨테이너 호스트는 VM) _MAC 주소 스푸핑이 필요_합니다.

- **오버레이** -docker 엔진이 [swarm 모드로](../manage-containers/swarm-mode.md)실행 되는 경우 오버레이 네트워크에 연결 된 컨테이너는 여러 컨테이너 호스트에서 동일한 네트워크에 연결 된 다른 컨테이너와 통신할 수 있습니다. Swarm 클러스터에 생성 된 각 오버레이 네트워크는 개인 IP 접두사에 의해 정의 되는 자체 IP 서브넷을 사용 하 여 만들어집니다. 오버레이 네트워크 드라이버는 VXLAN 캡슐화를 사용 합니다. **적절 한 네트워크 제어 평면 (예: Flannel)을 사용 하는 경우 Kubernetes와 함께 사용할 수 있습니다.**
  > 필요: 오버레이 네트워크를 만들기 위해 환경이 이러한 필수 [조건을](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) 충족 하는지 확인 합니다.

  > 필요: Windows Server 2019에서는 [KB4489899](https://support.microsoft.com/help/4489899)가 필요 합니다.

  > 필요: Windows Server 2016에서는 [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)가 필요 합니다.

  >[!NOTE]
  >Windows Server 2019에서 Docker Swarm에 의해 생성 된 오버레이 네트워크는 아웃 바운드 연결에 VFP NAT 규칙을 활용 합니다. 즉, 지정 된 컨테이너는 1 개의 IP 주소를 받습니다. 또한 또는와 같은 ICMP 기반 도구는 `ping` `Test-NetConnection` 디버깅 상황에서 TCP/UDP 옵션을 사용 하 여 구성 해야 합니다.

- **l2bridge** -네트워킹 모드와 유사 하 게 `transparent` ' l2bridge ' 드라이버로 만든 네트워크에 연결 된 컨테이너는 *외부* hyper-v 스위치를 통해 실제 네트워크에 연결 됩니다. L2bridge의 차이점은 컨테이너 끝점은 수신 및 송신에 대 한 계층 2 주소 변환 (MAC 다시 쓰기) 작업으로 인해 호스트와 동일한 MAC 주소를 갖게 된다는 것입니다. 클러스터링 시나리오에서이는 때때로 수명이 짧은 컨테이너의 MAC 주소를 학습 하는 스위치에 대 한 스트레스를 완화 하는 데 도움이 됩니다. L2bridge 네트워크는 다음과 같은 두 가지 방법으로 구성할 수 있습니다.
  1. L2bridge 네트워크가 컨테이너 호스트와 동일한 IP 서브넷으로 구성 되어 있습니다.
  2. L2bridge 네트워크는 새로운 사용자 지정 IP 서브넷으로 구성 됩니다.

  구성 2에서 사용자는 게이트웨이 역할을 하는 호스트 네트워크 구획에 끝점을 추가 하 고 지정 된 접두사에 대 한 라우팅 기능을 구성 해야 합니다.
  > 필요: Windows Server 2016, Windows 10 크리에이터 스 업데이트 또는 이후 버전이 필요 합니다.


- **l2bridge** -네트워킹 모드와 유사 하 게 `transparent` ' l2bridge ' 드라이버로 만든 네트워크에 연결 된 컨테이너는 *외부* hyper-v 스위치를 통해 실제 네트워크에 연결 됩니다. L2bridge의 차이점은 컨테이너 끝점은 수신 및 송신에 대 한 계층 2 주소 변환 (MAC 다시 쓰기) 작업으로 인해 호스트와 동일한 MAC 주소를 갖게 된다는 것입니다. 클러스터링 시나리오에서이는 때때로 수명이 짧은 컨테이너의 MAC 주소를 학습 하는 스위치에 대 한 스트레스를 완화 하는 데 도움이 됩니다. L2bridge 네트워크는 다음과 같은 두 가지 방법으로 구성할 수 있습니다.
  1. L2bridge 네트워크가 컨테이너 호스트와 동일한 IP 서브넷으로 구성 되어 있습니다.
  2. L2bridge 네트워크는 새로운 사용자 지정 IP 서브넷으로 구성 됩니다.

  구성 2에서 사용자는 게이트웨이 역할을 하는 호스트 네트워크 구획에 끝점을 추가 하 고 지정 된 접두사에 대 한 라우팅 기능을 구성 해야 합니다.
  >[!TIP]
  >L2bridge를 구성 하 고 설치 하는 방법에 대 한 자세한 내용은 [여기](https://techcommunity.microsoft.com/t5/networking-blog/l2bridge-container-networking/ba-p/1180923)에서 찾을 수 있습니다.

- **l2tunnel** -l2bridge와 유사 하지만 _이 드라이버는 Microsoft 클라우드 Stack (Azure) 에서만 사용 해야_합니다. 컨테이너에서 들어오는 패킷은 SDN 정책이 적용 되는 가상화 호스트에 전송 됩니다.


## <a name="network-topologies-and-ipam"></a>네트워크 토폴로지 및 IPAM

아래 표에서는 각 네트워크 드라이버에 대 한 내부 (컨테이너-컨테이너) 및 외부 연결에 대 한 네트워크 연결이 제공 되는 방법을 보여 줍니다.

### <a name="networking-modesdocker-drivers"></a>네트워킹 모드/Docker 드라이버

  | Docker Windows 네트워크 드라이버 | 일반적인 사용 용도 | 컨테이너-컨테이너 (단일 노드) | 컨테이너-외부 (단일 노드 + 다중 노드) | 컨테이너 간 (다중 노드) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (기본값)** | 개발자에 게 적합 | <ul><li>동일한 서브넷: Hyper-v 가상 스위치를 통한 브리지 연결</li><li> 서브넷 간: 지원 되지 않음 (NAT 내부 접두사 하나만)</li></ul> | 관리 vNIC를 통해 라우팅 (WinNAT에 바인딩) | 직접 지원 되지 않음: 호스트를 통해 포트를 노출 해야 함 |
  | **투명** | 개발자 또는 소규모 배포에 적합 | <ul><li>동일한 서브넷: Hyper-v 가상 스위치를 통한 브리지 연결</li><li>서브넷 간: 컨테이너 호스트를 통해 라우팅</li></ul> | (실제) 네트워크 어댑터에 직접 액세스 하 여 컨테이너 호스트를 통해 라우팅 | (실제) 네트워크 어댑터에 직접 액세스 하 여 컨테이너 호스트를 통해 라우팅 |
  | **오버레이** | 다중 노드에 적합 합니다. Kubernetes에서 사용할 수 있는 Docker Swarm에 필요 합니다. | <ul><li>동일한 서브넷: Hyper-v 가상 스위치를 통한 브리지 연결</li><li>서브넷 간: 네트워크 트래픽이 캡슐화 되 고 관리 vNIC를 통해 라우팅됩니다.</li></ul> | 직접 지원 되지 않음-windows server 2016의 NAT 네트워크에 연결 된 두 번째 컨테이너 끝점 또는 Windows Server 2019의 VFP NAT 규칙을 사용 해야 합니다.  | 동일/교차 서브넷: 네트워크 트래픽이 VXLAN를 사용 하 여 캡슐화 되 고 관리 vNIC를 통해 라우팅됩니다. |
  | **L2Bridge** | Kubernetes 및 Microsoft SDN에 사용 됩니다. | <ul><li>동일한 서브넷: Hyper-v 가상 스위치를 통한 브리지 연결</li><li> 서브넷 간: 수신 및 송신 및 라우팅에 대해 컨테이너 MAC 주소가 다시 작성 되었습니다.</li></ul> | 수신 및 송신 시 컨테이너 MAC 주소 다시 작성 | <ul><li>동일한 서브넷: 브리지 연결</li><li>서브넷 간: WSv1809 이상에서 관리 vNIC를 통해 라우팅</li></ul> |
  | **L2Tunnel**| Azure만 | 동일/교차 서브넷: 머리카락-실제 호스트의 Hyper-v 가상 스위치에 고정-정책이 적용 됨 | 트래픽이 Azure virtual network 게이트웨이를 통과 해야 함 | 동일/교차 서브넷: 머리카락-실제 호스트의 Hyper-v 가상 스위치에 고정-정책이 적용 됨 |

### <a name="ipam"></a>IPAM

IP 주소는 각 네트워킹 드라이버에 대해 다르게 할당 및 할당 됩니다. Windows는 HNS(호스트 네트워킹 서비스)를 사용하여 nat 드라이버에 대한 IPAM을 제공하고 Docker Swarm 모드(내부 KVS)에서 작동하여 오버레이에 대한 IPAM을 제공합니다. 다른 모든 네트워크 드라이버는 외부 IPAM을 사용 합니다.

| 네트워킹 모드/드라이버 | IPAM |
| -------------------------|:----:|
| NAT | 내부 NAT 서브넷 접두사에서 HNS (호스트 네트워킹 서비스)에의 한 동적 IP 할당 및 할당 |
| 투명 | 정적 또는 동적 (외부 DHCP 서버 사용) IP 할당 및 컨테이너 호스트의 네트워크 접두사 내의 IP 주소에서 할당 |
| 오버레이 | Docker 엔진 Swarm의 동적 IP 할당, HNS를 통한 관리 되는 접두사 및 할당 |
| L2Bridge | 컨테이너 호스트의 네트워크 접두사 내에서 IP 주소의 고정 IP 할당 및 할당 (HNS를 통해 할당 될 수도 있음) |
| L2Tunnel | Azure에만 해당-플러그 인에서 동적 IP 할당 및 할당 |

### <a name="service-discovery"></a>서비스 검색

서비스 검색은 특정 Windows 네트워크 드라이버에 대해서만 지원 됩니다.

|  | 로컬 서비스 검색  | 글로벌 서비스 검색 |
| :---: | :---------------     |  :---                |
| nat | YES | 예, Docker EE |
| 오버레이된 | YES | 예, Docker EE 또는 kube |
| transparent | 아니요 | 아니요 |
| l2bridge | 아니요 | 예 (kube 사용)-dns |
