---
title: "중첩된 가상화"
description: "중첩된 가상화"
keywords: windows 10, Hyper-V
author: johncslack
ms.date: 12/18/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: d5b8e888b62495c98c0691dc0d62deecf7c1eb6e
ms.sourcegitcommit: 6eefb890f090a6464119630bfbdc2794e6c3a3df
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/18/2017
---
# <a name="run-hyper-v-in-a-virtual-machine-with-nested-virtualization"></a>가상 컴퓨터에서 중첩된 가상화를 사용하여 Hyper-V 실행

중첩된 가상화는 Hyper-V 가상 컴퓨터(VM) 내에서 Hyper-V를 실행할 수 있는 기능입니다. 가상 컴퓨터에서 Visual Studio 휴대폰 에뮬레이터를 실행하거나 일반적으로 몇 가지 호스트가 필요한 구성을 테스트하는 데 도움이 됩니다.

![](./media/HyperVNesting.png)

## <a name="prerequisites"></a>필수 구성 요소

* Hyper-V 호스트 및 게스트는 Windows Server 2016/Windows 10 1주년 업데이트 이상이어야 합니다.
* VM 구성 버전 8.0 이상.
* VT-x 및 EPT 기술을 사용하는 Intel 프로세서 - 중첩은 현재 **Intel 전용**입니다.
* 두 번째 수준 가상 컴퓨터에 대한 가상 네트워킹에는 몇 가지 차이점이 있습니다. "중첩된 가상 컴퓨터 네트워킹"을 참조하세요.


## <a name="configure-nested-virtualization"></a>중첩된 가상화 구성

1. 가상 컴퓨터를 만듭니다. 필요한 OS 및 VM 버전은 위의 필수 조건을 참조하세요.
2. 가상 컴퓨터가 꺼짐 상태일 때 물리적 Hyper-V 호스트에서 다음 명령을 실행합니다. 그러면 가상 컴퓨터에서 중첩된 가상화를 사용할 수 있습니다.

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 가상 컴퓨터를 시작합니다.
4. 물리적 서버에서 하는 것처럼 가상 컴퓨터 내에 Hyper-V를 설치합니다. Hyper-V 설치에 대한 자세한 내용은 [Hyper-V 설치](../quick-start/enable-hyper-v.md)를 참조하세요.

## <a name="disable-nested-virtualization"></a>중첩된 가상화 사용 안 함
중지된 가상 컴퓨터에 대해 중첩된 가상화를 사용하지 않도록 설정하려면 다음 PowerShell 명령을 사용합니다.
```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## <a name="dynamic-memory-and-runtime-memory-resize"></a>동적 메모리 및 런타임 메모리 크기 조정
Hyper-V가 가상 컴퓨터 내에 실행되고 있는 경우 가상 컴퓨터를 꺼야만 해당 메모리를 조정할 수 있습니다. 즉, 동적 메모리를 사용하도록 설정한 경우에도 메모리 양이 변동되지 않습니다. 동적 메모리를 사용하도록 설정하지 않은 가상 컴퓨터의 경우 켜져 있을 때 메모리 양을 조정하려고 하면 항상 실패합니다. 

중첩된 가상화를 사용하도록 설정하는 것만으로는 동적 메모리 또는 런타임 메모리의 크기가 조정되지 않습니다. 이러한 비호환성은 Hyper-V가 VM에서 실행 중인 경우에만 발생합니다.

## <a name="networking-options"></a>네트워킹 옵션

중첩된 가상 컴퓨터에는 두 가지 네트워킹 옵션이 있습니다. 

1. MAC 주소 스푸핑
2. NAT 네트워킹

### <a name="mac-address-spoofing"></a>MAC 주소 스푸핑
네트워크 패킷을 두 가상 스위치를 통해 전송하려면 첫 번째 수준의 가상 스위치에서 MAC 주소 스푸핑을 사용하도록 설정해야 합니다. 이 작업은 다음 PowerShell 명령으로 수행합니다.

``` PowerShell
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="network-address-translation-nat"></a>NAT(네트워크 주소 변환)
두 번째 옵션은 NAT(네트워크 주소 변환)를 사용합니다. 이 방법은 공용 클라우드 환경과 같이 MAC 주소 스푸핑이 가능하지 않은 경우에 적합합니다.

먼저 호스트 가상 컴퓨터("중간" VM)에서 가상 NAT 스위치를 만들어야 합니다. IP 주소는 예일 뿐이며 환경에 따라 다릅니다.

``` PowerShell
New-VMSwitch -Name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```

그런 다음 넷 어댑터에 IP 주소를 할당합니다.

``` PowerShell
Get-NetAdapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```

중접된 가상 컴퓨터 각각에는 IP 주소와 게이트웨이가 할당되어 있어야 합니다. 게이트웨이 IP는 이전 단계의 NAT 어댑터를 가리켜야 합니다. DNS 서버를 할당하고자 할 수도 있습니다.

``` PowerShell
Get-NetAdapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## <a name="how-nested-virtualization-works"></a>중첩된 가상화 작동 원리

최신 프로세서에는 가상화를 보다 빠르고 더 안전하게 만드는 하드웨어 기능이 포함됩니다. Hyper-V는 가상 컴퓨터를 실행하기 위해 프로세서 확장을 사용합니다(예: Intel VT-x 및 AMD-V). 일반적으로 Hyper-V가 시작되면 이러한 프로세서 기능을 사용하여 다른 소프트웨어를 방지합니다.  이렇게 하면 가상 컴퓨터가 Hyper-V를 실행하지 않도록 합니다.

중첩된 가상화를 통해 이 하드웨어 지원을 게스트 가상 컴퓨터에서 사용할 수 있습니다.

아래 다이어그램은 중첩 없는 Hyper-V를 보여 줍니다.  Hyper-V 하이퍼바이저는 하드웨어 가상화 기능(주황색 화살표)을 전체 제어하며 이 기능들을 게스트 운영 체제에 노출하지 않습니다.

![](./media/HVNoNesting.png)

반대로 아래 다이어그램은 중첩된 가상화가 활성화된 Hyper-V를 보여 줍니다. 이 경우 Hyper-V는 해당 가상 컴퓨터에 하드웨어 가상화 확장을 제공합니다. 중첩을 활성화하면 게스트 가상 컴퓨터는 자체 하이퍼바이저를 설치하고 자체 게스트 VM을 실행할 수 있습니다.

## <a name="3rd-party-virtualization-apps"></a>타사 가상화 앱

Hyper-V 외의 가상화 응용 프로그램은 Hyper-V 가상 컴퓨터에서 지원되지 않으며 실패할 확률이 높습니다. 하드웨어 가상화 확장이 필요한 모든 소프트웨어가 여기에 포함됩니다.
