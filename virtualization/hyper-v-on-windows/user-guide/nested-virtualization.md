---
title: "중첩된 가상화"
description: "중첩된 가상화"
keywords: windows 10, hyper-v
author: theodthompson
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
translationtype: Human Translation
ms.sourcegitcommit: e714d4dc22c0049d3365d4a4f3c11d072f46a161
ms.openlocfilehash: 7d16fcf22187ae3ace25fe1bedbc02f3c6b63eb8
ms.lasthandoff: 02/14/2017

---

# 가상 컴퓨터에서 중첩된 가상화를 사용하여 Hyper-V 실행

중첩된 가상화는 Hyper-V 가상 컴퓨터 내에서 Hyper-V를 실행할 수 있는 기능입니다. 즉, 중첩된 가상화를 통해 Hyper-V 호스트 자체를 가상화할 수 있습니다. 중첩된 가상화의 몇 가지 사용 사례로 가상화된 컨테이너 호스트에서 Hyper-V 컨테이너 실행, 가상화된 환경에서 Hyper-V 랩 실행, 개별 하드웨어 없이 다중 컴퓨터 시나리오 테스트 등을 들 수 있습니다. 이 문서에서는 소프트웨어 및 하드웨어 필수 조건, 구성 단계 및 제한 사항을 자세히 설명합니다. 

## 필수 구성 요소

- Windows Server 2016 또는 Windows 10 1주년 업데이트를 실행하는 Hyper-V 호스트
- Windows Server 2016 또는 Windows 10 1주년 업데이트를 실행하는 Hyper-V VM
- 구성 버전 8.0 이상을 사용하는 Hyper-V VM
- VT-x 및 EPT 기술을 사용하는 Intel 프로세서

## 중첩된 가상화 구성

1. 가상 컴퓨터를 만듭니다. 필요한 OS 및 VM 버전은 위의 필수 조건을 참조하세요.
2. 가상 컴퓨터가 꺼짐 상태일 때 물리적 Hyper-V 호스트에서 다음 명령을 실행합니다. 그러면 가상 컴퓨터에서 중첩된 가상화를 사용할 수 있습니다.

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 가상 컴퓨터를 시작합니다.
4. 물리적 서버에서 하는 것처럼 가상 컴퓨터 내에 Hyper-V를 설치합니다. Hyper-V 설치에 대한 자세한 내용은 [Hyper-V 설치](../quick-start/enable-hyper-v.md)를 참조하세요.

## 중첩된 가상화 사용 안 함
중지된 가상 컴퓨터에 대해 중첩된 가상화를 사용하지 않도록 설정하려면 다음 PowerShell 명령을 사용합니다.
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## 동적 메모리 및 런타임 메모리 크기 조정
Hyper-V가 가상 컴퓨터 내에 실행되고 있는 경우 가상 컴퓨터를 꺼야만 해당 메모리를 조정할 수 있습니다. 즉, 동적 메모리를 사용하도록 설정한 경우에도 메모리 양이 변동되지 않습니다. 동적 메모리를 사용하도록 설정하지 않은 가상 컴퓨터의 경우 켜져 있을 때 메모리 양을 조정하려고 하면 항상 실패합니다. 

중첩된 가상화를 사용하도록 설정하는 것만으로는 동적 메모리 또는 런타임 메모리의 크기가 조정되지 않습니다. 이러한 비호환성은 Hyper-V가 VM에서 실행 중인 경우에만 발생합니다.

## 네트워킹 옵션
중첩된 가상 컴퓨터에서는 MAC 주소 스푸핑 및 NAT 모드의 두 가지 네트워킹 옵션을 지정합니다.

### MAC 주소 스푸핑
네트워크 패킷을 두 가상 스위치를 통해 전송하려면 첫 번째 수준의 가상 스위치에서 MAC 주소 스푸핑을 사용하도록 설정해야 합니다. 이 작업은 다음 PowerShell 명령으로 수행합니다.

```none
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### Network Address Translation
두 번째 옵션은 NAT(네트워크 주소 변환)를 사용합니다. 이 방법은 공용 클라우드 환경과 같이 MAC 주소 스푸핑이 가능하지 않은 경우에 적합합니다.

먼저 호스트 가상 컴퓨터("중간" VM)에서 가상 NAT 스위치를 만들어야 합니다. IP 주소는 예일 뿐이며 환경에 따라 다릅니다.
```none
new-vmswitch -name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```
그런 다음 넷 어댑터에 IP 주소를 할당합니다.
```none
get-netadapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```
중접된 가상 컴퓨터 각각에는 IP 주소와 게이트웨이가 할당되어 있어야 합니다. 게이트웨이 IP는 이전 단계의 NAT 어댑터를 가리켜야 합니다. DNS 서버를 할당할 수도 있습니다.
```none
get-netadapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## 타사 가상화 앱
Hyper-V 외의 가상화 응용 프로그램은 Hyper-V 가상 컴퓨터에서 지원되지 않으며 실패할 확률이 높습니다. 하드웨어 가상화 확장이 필요한 모든 소프트웨어가 여기에 포함됩니다.

