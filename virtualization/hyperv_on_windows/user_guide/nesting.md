---
title: "중첩된 가상화"
description: "중첩된 가상화"
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.sourcegitcommit: 26a8adb426a7cf859e1a9813da2033e145ead965
ms.openlocfilehash: d17413fc572e59ec21ff513ef5de994c6716aa08

---

# 가상 컴퓨터에서 중첩된 가상화를 사용하여 Hyper-V 실행

중첩된 가상화는 Hyper-V 가상 컴퓨터 내에서 Hyper-V를 실행할 수 있는 기능입니다. 즉, 중첩된 가상화를 통해 Hyper-V 호스트 자체를 가상화할 수 있습니다. 중첩된 가상화의 몇 가지 사용 사례로 가상화된 컨테이너 호스트에서 Hyper-V 컨테이너 실행, 가상화된 환경에서 Hyper-V 랩 실행, 개별 하드웨어 없이 다중 컴퓨터 시나리오 테스트 등을 들 수 있습니다. 이 문서에서는 소프트웨어 및 하드웨어 필수 조건 및 구성 단계를 자세히 설명하고 문제 해결 정보를 제공합니다. Windows 참가자 미리 보기 빌드 14361 이상에서 Hyper-V를 실행하는 경우 [Nested Virtualization Preview for Windows Insiders: Builds 14361+(Windows 참가자에 대해 중첩된 가상화 미리 보기: 빌드 14361+)](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting#nested-virtualization-preview-for-windows-insiders-builds-14361-)을 참조하세요.

## 필수 구성 요소

- 빌드 10565 이상을 실행 중인 Windows 참가자 빌드(Windows Server 2016 또는 Windows 10)를 실행하는 Hyper-V 호스트
- 두 하이퍼바이저(부모 및 자식)가 모두 동일한 Windows 빌드(10565 이상)를 실행하고 있어야 합니다.
- Intel VT-x 및 EPT 기술을 사용하는 Intel 프로세서

## 중첩된 가상화 구성

먼저 가상 컴퓨터를 만듭니다. **가상 컴퓨터를 켜지 마세요**.. 자세한 내용은 [가상 컴퓨터 만들기](../quick_start/walkthrough_create_vm.md)를 참조하세요.

가상 컴퓨터를 만들었으면 물리적 Hyper-V 호스트에서 다음 명령을 실행합니다. 그러면 가상 컴퓨터에서 중첩된 가상화를 사용할 수 있습니다.

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
중첩된 Hyper-V 호스트를 실행할 때는 가상 컴퓨터에서 동적 메모리를 사용하지 않도록 설정해야 합니다. 이 설정은 가상 컴퓨터의 속성에서 하거나 다음 PowerShell 명령을 사용하여 구성할 수 있습니다.
```none
Set-VMMemory -VMName <VMName> -DynamicMemoryEnabled $false
```

이러한 단계를 완료했으면 가상 컴퓨터를 시작하고 Hyper-V를 설치할 수 있습니다. Hyper-V 설치에 대한 자세한 내용은 [Hyper-V 설치]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)를 참조하세요.

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


## 알려진 문제

- 장치 가드가 활성화된 호스트는 가상화 확장을 게스트에 공개할 수 없습니다.
- VBS(가상화 기반 보안)를 사용하도록 설정한 가상 컴퓨터에서 동시에 중첩된 가상화를 사용하도록 설정할 수 없습니다. 중첩된 가상화를 사용하려면 먼저 VBS를 사용하지 않도록 설정해야 합니다.
- 중첩된 가상화가 가상 컴퓨터에서 활성화되면 다음과 같은 기능을 해당 VM과 더 이상 호환할 수 없습니다.  
  * 런타임 메모리 크기 조정 및 동적 메모리
  * 검사점
  * Hyper-V를 사용하도록 설정한 가상 컴퓨터를 실시간 마이그레이션할 수 없습니다.

## FAQ 및 문제 해결

가상 컴퓨터가 시작되지 않으면 어떻게 해야 합니까?

1. 동적 메모리가 OFF인지 확인합니다.
2. 지원하는 Intel 프로세서를 사용하는지 확인합니다.
3. 관리자 권한 프롬프트로 호스트 컴퓨터에서 [이 PowerShell 스크립트](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1)를 실행합니다.

## 사용자 의견

Windows 피드백 앱, [가상화 포럼](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv) 또는 [GitHub](https://github.com/Microsoft/Virtualization-Documentation)를 통해 추가 문제를 보고하세요.

##Windows 참가자에 대해 중첩된 가상화 미리 보기: 빌드 14361+
몇 달 전에 빌드 10565와 함께 Hyper-V 중첩된 가상화의 초기 미리 보기를 발표했습니다. 이 기능의 뛰어난 성능을 보며 흥분했었고, 이제 Windows 참가자와 업데이트를 공유되어 기쁩니다.

###중첩된 가상화에 대해 새 VM 버전 필요
빌드 14361부터 버전 8.0에서는 VM에 중첩된 가상화를 사용해야 합니다. 이전 호스트에서 만든 중첩된 가상화를 사용하는 VM에 대한 버전 업데이트가 필요합니다. 

####VM 버전 업데이트
중첩된 가상화를 계속 사용하면 VM 버전을 8.0으로 업데이트해야 합니다. 즉 저장된 상태를 제거하고 VM을 종료해야 합니다. 다음 PowerShell cmdlet은 VM 버전을 업데이트합니다.
```none
Update-VMVersion -Name <VMName>
```
####중첩된 가상화 사용 안 함
VM을 업데이트하고 싶지 않은 경우, VM에서 부팅할 수 있도록 중첩된 가상화를 사용하지 않도록 설정할 수 있습니다.
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

###VM 버전 8.0의 새로운 동작 
이 미리 보기에는 중첩이 설정된 VM이 작동하는 방법에 대한 몇 가지 변경 사항이 있습니다.
-   검사점 생성 및 적용이 이제 중첩된 가상화를 사용하는 VM에 대해 작동합니다.
-   이제 중첩이 설정된 VM을 저장 및 시작할 수 있습니다.
-   중첩된 가상화를 사용하는 VM을 이제 가상화 기반 보안(Device Guard 및 Credential Guard 포함)을 사용하는 호스트에서 실행할 수 있습니다.
-   기존 제한 사항에 대한 오류 메시지가 향상되었습니다.

###기능 제한 사항
-   중첩된 가상화는 Hyper-V 가상 컴퓨터 내에서 Hyper-V를 실행하도록 설계되었습니다. 타사 가상화 응용 프로그램은 지원되지 않으며 Hyper-V VM에서 실패할 확률이 높습니다.
-   동적 메모리는 중첩된 가상화와 호환되지 않습니다. VM 내에서 Hyper-V를 실행하면 VM에서 런타임 시 메모리를 변경할 수 없습니다. 
-   런타임 메모리 크기 조정은 중첩된 가상화와 호환되지 않습니다. Hyper-V가 내부에서 실행되는 동안 VM 메모리 크기 조정은 실패합니다. 
-   중첩된 가상화는 Intel 시스템에서만 지원됩니다.

###알려진 문제
빌드 14361에서 다음과 같은 오류와 함께 2세대 VM이 부팅되지 않는 알려진 문제가 있습니다.
```none
“Cannot modify property without enabling VirtualizationBasedSecurityOptOut”
```
이 문제는 중첩된 가상화를 사용하지 않도록 설정하거나 가상화 기반 보안을 옵트아웃(opt out)하여 일시적으로 해결할 수 있습니다.
```none
Set-VMSecurity -VMName <vmname> -VirtualizationBasedSecurityOptOut $true
```

###피드백 환영
언제나처럼 Windows 피드백 앱을 통해 계속 피드백을 보내주세요. 질문이 있는 경우 문서 [GitHub](https://github.com/Microsoft/Virtualization-Documentation) 페이지에서 문제를 제출하세요. 



<!--HONumber=Jun16_HO4-->


