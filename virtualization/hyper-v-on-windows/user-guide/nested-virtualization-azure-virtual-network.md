---
title: Azure 가상 네트워크에 있는 리소스와 직접 통신 하는 중첩 된 Vm 구성
description: 중첩된 가상화
keywords: windows 10, hyper-v가 설치, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: c39a2e5639d013a0aba15150117b7ab18ea32f97
ms.sourcegitcommit: 1aef193cf56dd0870139b5b8f901a8d9808ebdcd
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001659"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Azure 가상 네트워크에 있는 리소스와 통신 하는 중첩 된 Vm 구성

배포 하 고 Azure 내에 중첩 된 가상 컴퓨터 구성에 대 한 원래 지침 NAT 스위치를 통해 이러한 Vm에 액세스 하는 필요 합니다. 이 인해 몇 가지 제한이 있습니다.

1. 중첩 된 Vm 온-프레미스 리소스에 액세스할 수 없는 또는 Azure 가상 네트워크 내에서 합니다.
2. 온-프레미스 리소스 또는 리소스 Azure 내에서 여러 게스트 동일한 포트를 공유할 수 없는 NAT를 통해 중첩 된 Vm 액세스할만 수 있습니다.

설정 양식이의 RRAS를 사용 하 여 배포를 통해이 문서는 일부 사용자 정의 된 경로 및 중첩 된 Vm 동작 하 고 직접 Azure 내 VNet에 배포 된 가상 컴퓨터와 같은 통신을 허용 하도록 "부동" 주소 공간을 합니다.

이 가이드를 하세요 시작 하기 전에:

1. 읽기는 [여기에 제공 된 지침에](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization) 중첩 된 가상화 중첩 가능한 Vm을 만들고 해당 Vm 내에서 Hyper-v 역할을 설치 합니다. Hyper-v 역할 설정 지난 진행 하지 마십시오.
2. 구현 하기 전에이 전체 문서를 참조 합니다.

이 가이드에서는 대상 환경에 대 한 다음을 가정 합니다.

1. 우리는 ExpressRoute에 연결 되는 허브 [허브 및 스포크 토폴로지](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)작동 합니다.
1. 스포크 네트워크가 10.0.0.0/23는 두 개의 /24 서브넷으로 형성 된 곳의 주소 공간을 할당 됩니다.
    * 10.0.0.0/24 서브넷 Hyper-v 호스트 상주 합니다.
    * 10.0.1.0/24 – "부동" 서브넷이 됩니다. 이 주소 공간은 중첩 된 Vm에서 사용 하는 있고 온-프레미스에 다시 경로 알림을 처리 하도록 합니다.
    * 스포크 VNet 정교 "스포크" 라고 합니다.

1. 허브 네트워크 IP 범위와 관련 된 하지만 이름이 "허브"입니다.
1. 우리의 Hyper-v 10.0.0.4/24의 주소를 할당 됩니다.
1. DNS 서버 10.0.0.10/24에 있는,이 요구 사항이 있지만 연습 과정의 가정 합니다.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>여기서는 높은 수준의 개요 수행 및 이유

* 배경: 중첩 된 Vm은 DHCP에서에서 받지 내부 또는 외부 스위치를 구성 하는 경우에 해당 호스트에 연결 되어 있는 VNet. 
  * 즉, Hyper-v 호스트 DHCP를 제공 해야 합니다.
* 우리는 Hyper-v 호스트에서 마법사 사용에 대 한 Ip 블록을 할당 합니다.  Hyper-v 호스트의 VNet에 현재 할당 된 임대 인식 하지 않으므로는 호스트에에서 게 할당 된 IP 이미 존재 하는 상황을 방지 하기 위해 Hyper-v 호스트에서 방금 사용할 Ip 블록 할당 해야 합니다. 이렇게 하면 중복 IP 시나리오를 방지할 수 있습니다.
  * 블록을 선택 하는 Ip 서브넷에 있는 Hyper-v 호스트 VNet 내에 해당 됩니다.
  * 있는 기존 서브넷에 해당 하는 이유는 ExpressRoute 위로 다시 BGP 광고를 처리 하는 것입니다. 사용 하 여 Hyper-v 호스트에 대 한 IP 범위를 방금 만든 것 이면 일련의 클라이언트가 수 있도록 정적 경로 만들려고 했습니다 온-프레미스 중첩 된 가상 컴퓨터와 통신할 수 있습니다. 이 의미는 중첩 된 Vm에 대 한 IP 범위를 구성 하 고 다음 클라이언트 해당 범위에 대 한 Hyper-v 호스트에 지시 하는 데 필요한 모든 경로 만들 수 하드 요구 사항이 아닙니다.
* Hyper-v 내에서 내부 스위치를 만듭니다 하 고 새로 생성된 된 인터페이스 DHCP에 대 한 제외 설정 범위 내에서 IP 주소가 할당은 다음 합니다. 이 IP 주소는 중첩 된 Vm에 대 한 기본 게이트웨이 될 되며 될 경로 내부 스위치와이 VNet에 연결 된 호스트의 NIC 사이 하는 데 사용 합니다.
* 호스트 라우터도 전환 하는 호스트에서 라우팅 및 원격 액세스 역할을 설치 하겠습니다.  이것은 호스트 외부 리소스와는 중첩 된 Vm 간의 통신을 허용 하는 데 필요 합니다.
* 다른 리소스 이러한 중첩 된 Vm에 액세스 하는 방법을 지시 합니다. 이 작업에 중첩 된 Vm에 있는 IP 범위에 대 한 정적 경로 포함 하는 사용자 정의 경로 테이블을 만드는 것입니다. Hyper-v에 대 한 정적 경로이 IP 주소를 가리킵니다.
* 온-프레미스에서 오는 클라이언트는 중첩 된 Vm에 도달 하는 방법을 알 수 있도록이 UDR 게이트웨이 서브넷에 배치 다음 됩니다.
* 또한 중첩 된 Vm에 대 한 연결을 요구 하는 Azure 내에서 다른 서브넷에이 UDR을 배치 됩니다.
* 여러 Hyper-v 호스트에 대 한 추가 "부동" 서브넷 만들고는 UDR에 추가 정적 경로 추가 합니다.
* Hyper-v 호스트 역할을 해제할 때는 삭제/재활용 우리의 "부동" 서브넷 및 우리의 UDR에서 정적 경로 제거 하거나 이것이 마지막 Hyper-v 호스트의 UDR을 모두 제거 합니다.

## <a name="creating-our-virtual-switch"></a>우리의 가상 스위치 만들기

1. 관리 모드에서 PowerShell을 엽니다.
2. 내부 스위치를 만듭니다. `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. 새로 만든된 인터페이스는 IP를 할당 합니다. `New-NetIPAddress –IPAddress 10.0.1.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>설치 및 DHCP를 구성 합니다.

*많은 사람들이 중첩 된 가상화 사용 먼저 시도 하는 경우이 구성 요소를 누락. 달리에서 온-프레미스에 게스트 Vm 호스트에 상주 하는 네트워크에서 DHCP를 받게 됩니다 Azure에서 중첩 된 Vm 제공 해야 합니다 DHCP은 통해 호스트에서 실행 합니다. 하거나 하는 정적 IP 주소를 할당 중첩 된 모든 VM에 상관 없는 확장 가능 해야 합니다.*

1. DHCP 역할을 설치 합니다. `Install-WindowsFeature DHCP -IncludeManagementTools`
2. DHCP 범위를 만듭니다. `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.1.2 -EndRange 10.0.1.254 -SubnetMask 255.255.255.0`
3. 범위에 대 한 DNS 및 기본 게이트웨이 옵션을 구성 합니다. `Set-DhcpServerV4OptionValue -DnsServer 10.0.0.10 -Router 10.0.1.1`
    * 올바른 DNS 서버를 입력 해야 합니다. 이 경우 Windows DNS를 서비스 하는 10.0.0.0/24 네트워크에서 서버를 준비 발생 합니다.

## <a name="installing-remote-access"></a>원격 액세스 설치

* 서버 관리자를 열고 "추가 역할 및 기능"를 선택 합니다.
* "서버 역할"에 도달할 때까지 "다음"를 선택 합니다.
* "원격 액세스"를 확인 하 고 "역할 서비스"에 도달할 때까지 "다음"을 클릭 합니다.
* "라우팅" 확인 "기능 추가"를 선택한 다음 "다음" 한 다음 "설치"를 선택 합니다. 마법사를 완료 하 고 설치를 완료 될 때까지 기다립니다.

## <a name="configuring-remote-access"></a>원격 액세스 구성

* 서버 관리자를 열고 "도구"를 선택 하 고 "라우팅 및 원격 액세스"를 선택 합니다.
* 라우팅 및 원격 액세스 관리 패널의 오른쪽에서 아이콘 옆에 있는 서버 이름으로 보고,이 마우스 오른쪽 단추로 클릭 하 됩니다 "구성 및 사용 라우팅 및 원격 액세스"를 선택 합니다.
* 마법사에서 선택 "다음", "사용자 지정 구성"에 대 한 방사형 단추를 선택 하 고 "다음"를 선택 합니다.
* "NAT" 확인 "LAN 라우팅" 한 다음 선택 "다음"와 다음 ""입니다. 서비스를 시작 하 고 이렇게 묻는 메시지가 나타납니다 하는 경우.
* 이제 "IPv4" 노드로 이동 하 고 "NAT" 노드를 사용할 수 있도록 확장 합니다.
* 마우스 오른쪽 단추로 클릭 "NAT", "새 인터페이스"를 선택 하 고 세 가지 옵션을 사용 하 여 표시 해야 합니다. 
* Hyper-v 가상 스위치에 대 한 다음이 옵션 중 두 가지, "Ethernet"에 대 한 옵션을 표시 되어야,이 옵션을 선택 합니다. 즉, Azure VNet에 직접 연결 되어 있는 인터페이스를 선택 합니다.

## <a name="creating-a-route-table-within-azure"></a>Azure 내 경로 테이블 만들기

깊이 읽기를 만들고 Azure 내에서 경로 관리에 대 한 자세한 내용은 [이 문서](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal) 참조 하세요.

* 이동 https://portal.azure.com.
* 왼쪽 위 모서리에 있는 "리소스를 만들고"를 선택 합니다.
* 검색 필드에 "경로 테이블"를 입력 하 고 적중 횟수를 입력 합니다.
* 상위 결과 경로 테이블,이 항목을 선택 및 "만들기"를 선택 합니다.
* 경로 테이블 이름, 여기서에서 이름을 지정한 것 "경로-에 대 한-중첩-Vm".
* Hyper-v 호스트에 있는 동일한 구독을 선택 했는지 확인 합니다.
* 새 리소스 그룹 만들기는 기존 선택 및 Hyper-v 호스트에 있는 동일한 지역에서 경로 테이블을 만들면 지역 되는지 확인 합니다.
* "만들기"를 선택 합니다.

## <a name="configuring-the-route-table"></a>경로 테이블 구성

* 방금 만든 경로 테이블으로 이동 합니다. 포털의 상단에서 검색 창에서 경로 테이블의 이름을 검색 하 여 설정할 수 있습니다.
* 선택 하면 경로 테이블에서 이동 하려면 "경로" 블레이드 내 합니다.
* "추가"를 선택 합니다.
* 이름을 지정 하 여 경로, "중첩 Vm"으로 진행 합니다.
* 주소에 대 한 접두사는 "부동" 서브넷에 대 한 IP 범위를 입력합니다. 이 경우 10.0.1.0/24 것입니다.
* "다음 홉 유형은"에 대 한 "가상 어플라이언스"를 선택 하 고, 10.0.0.4는 되며 다음 "확인"을 선택 하는 Hyper-v 호스트에 대 한 IP 주소를 입력 합니다.
* 이제에서 블레이드 선택 "서브넷" 내 됩니다 "경로" 바로 아래입니다.
* 선택 "연결" 다음 "허브" 가상 네트워크를 선택 하 고 "GatewaySubnet"을 선택 고 "확인"를 선택 합니다.
* Hyper-v 호스트에 있는 중첩 된 Vm에 액세스 해야 하는 다른 서브넷: 서브넷에 대 한이 동일한 프로세스를 수행 합니다.

## <a name="conclusion"></a>결론

이제 Hyper-v 호스트에 가상 컴퓨터 (32 비트 VM도!) 배포와 하는 온-프레미스에서 및 Azure 내에서 액세스할 수 있어야 합니다.
