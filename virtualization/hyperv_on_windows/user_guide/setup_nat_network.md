---
title: NAT 네트워크 설정
description: NAT 네트워크 설정
keywords: windows 10, hyper-v
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
---

# NAT 네트워크 설정

Windows 10 Hyper-V에서는 가상 네트워크에 대해 기본 NAT(Network Address Translation)를 허용합니다.

이 가이드에서는 다음을 안내합니다.
* NAT 네트워크 만들기
* 기존 가상 컴퓨터를 새 네트워크에 연결
* 가상 컴퓨터가 올바르게 연결되어 있는지 확인

요구 사항:
* Windows 빌드 14295 이상
* Hyper-V 역할 사용(지침은 [여기](../quick_start/walkthrough_create_vm.md) 참조)

> **참고:**  현재 Hyper-V에서는 NAT 네트워크를 하나만 만들 수 있습니다.

## NAT 개요
NAT는 호스트 컴퓨터의 IP 주소와 포트를 사용하여 네트워크 리소스에 대한 액세스 권한을 가상 컴퓨터에 제공합니다.

NAT(Network Address Translation)는 외부 IP 주소 및 포트를 훨씬 더 큰 내부 IP 주소 집합에 매핑하여 IP 주소를 절약하도록 설계된 네트워킹 모드입니다.  기본적으로 NAT 스위치는 NAT 매핑 테이블을 사용하여 IP 주소 및 포트 번호에서 네트워크의 장치(가상 컴퓨터, 컴퓨터, 컨테이너 등)와 연결된 올바른 내부 IP 주소로 트래픽을 라우팅합니다.

또한 NAT를 사용하면 여러 가상 컴퓨터에서 동일한(내부) 통신 포트가 필요한 응용 프로그램을 고유한 외부 포트에 매핑하여 호스트할 수 있습니다.

이러한 모든 이유로 NAT 네트워킹은 컨테이너 기술에 매우 일반적입니다([컨테이너 네트워킹](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking) 참조).


## NAT 가상 네트워크 만들기
새로운 NAT 네트워크 설정 과정을 살펴보겠습니다.

1.  관리자 권한으로 PowerShell 콘솔을 엽니다.  

2. 내부 스위치 만들기  

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. [New-NetIPAddress](https://technet.microsoft.com/en-us/library/hh826150.aspx)를 사용하여 NAT 게이트웨이를 구성합니다.  

  다음은 일반 명령입니다.
  ``` PowerShell
  New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
  ```

  게이트웨이를 구성하려면 네트워크에 대한 약간의 정보가 필요합니다.  
  * **IPAddress** -- NAT 게이트웨이 IP는 NAT 게이트웨이 IP로 사용할 IPv4 또는 IPv6 주소를 지정합니다.  
    일반적인 형식은 a.b.c.1(예: 172.16.0.1)입니다.  최종 위치가 반드시 .1일 필요는 없지만 일반적으로 그렇습니다(접두사 길이를 기반).

    일반적인 게이트웨이 IP는 192.168.0.1입니다.  

  * **PrefixLength** -- NAT 서브넷 접두사 길이는 NAT 로컬 서브넷 크기(서브넷 마스크)를 정의합니다.
    서브넷 접두사 길이는 0에서 32 사이의 정수 값입니다.

    0은 전체 인터넷을 매핑하고, 32는 매핑된 IP를 하나만 허용합니다.  일반적인 값의 범위는 NAT에 연결해야 하는 IP 수에 따라 24에서 12 사이입니다.

    일반적인 PrefixLength는 24로, 서브넷 마스크 255.255.255.0입니다.

  * **InterfaceIndex** -- ifIndex는 위에서 만든 가상 스위치의 인터페이스 색인입니다.

    인터페이스 색인은 다음 명령을 실행하여 찾을 수 있습니다. `Get-NetAdapter`

    출력은 다음과 같이 표시됩니다.

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    내부 스위치에 `vEthernet (SwitchName)` 같은 이름과 `Hyper-V Virtual Ethernet Adapter` 같은 인터페이스 설명이 지정됩니다.

  다음을 실행하여 NAT 게이트웨이를 만듭니다.

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

4. [New-NetNat](https://technet.microsoft.com/en-us/library/dn283361(v=wps.630).aspx)를 사용하여 NAT 네트워크를 구성합니다.  

  다음은 일반 명령입니다.

  ``` PowerShell
  New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
  ```

  게이트웨이를 구성하려면 네트워크 및 NAT 게이트웨이에 대한 정보를 제공해야 합니다.  
  * **이름** -- NATOutsideName은 NAT 네트워크의 이름을 설명합니다.  이 정보는 NAT 네트워크를 제거하는 데 사용됩니다.

  * **InternalIPInterfaceAddressPrefix** -- NAT 서브넷 접두사는 위의 NAT 게이트웨이 IP 접두사와 위의 NAT 서브넷 접두사 길이를 모두 설명합니다.

    일반적인 형식은 a.b.c.0/NAT 서브넷 접두사 길이입니다.

    이 예제에서는 위의 192.168.0.0/24를 사용합니다.

  예제에서 다음을 실행하여 NAT 네트워크를 설정합니다.

  ``` PowerShell
  New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
  ```

축하합니다.  이제 가상 NAT 네트워크를 만들었습니다.  NAT 네트워크에 가상 컴퓨터를 추가하려면 [이러한 지침](setup_nat_network.md#connect-a-virtual-machine)을 따르세요.

## 가상 컴퓨터 연결

새 NAT 네트워크에 가상 컴퓨터를 연결하려면 [NAT 네트워크 설정](setup_nat_network.md#create-a-nat-virtual-network) 섹션의 첫 번째 단계에서 만든 내부 스위치를 VM 설정 메뉴를 사용하여 가상 컴퓨터에 연결합니다.


## 문제 해결

이 워크플로에서는 호스트에 다른 NAT가 없다고 가정합니다. 그러나 경우에 따라 여러 응용 프로그램이나 서비스에서 NAT를 사용합니다. Windows(WinNAT)는 내부 NAT 서브넷 접두사를 하나만 지원하므로 여러 NAT를 만들려고 하면 시스템이 알 수 없는 상태로 전환됩니다.

### 문제 해결 단계
1. NAT가 하나만 있는지 확인합니다.

  ``` PowerShell
  Get-NetNat
  ```
2. NAT가 이미 있으면 삭제합니다.

  ``` PowerShell
  Get-NetNat | Remove-NetNat
  ```

3. NAT에 대해 하나의 "내부" vmSwitch만 있는지 확인합니다. 4단계를 위해 vSwitch의 이름을 기록합니다.

  ``` PowerShell
  Get-VMSwitch
  ```

4. 오래된 NAT의 개인 IP 주소(예: NAT 기본 게이트웨이 IP 주소 - 일반적으로 *.1)가 어댑터에 계속 할당되어 있는지 확인합니다.

  ``` PowerShell
  Get-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)"
  ```

5. 오래된 개인 IP 주소가 사용 중인 경우 삭제합니다.  
   ``` PowerShell
  Remove-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)" -IPAddress <IPAddress>
  ```

## 여러 응용 프로그램에서 동일한 NAT 사용

일부 시나리오에서는 여러 응용 프로그램이나 서비스에서 동일한 NAT를 사용해야 합니다. 이 경우 여러 응용 프로그램/서비스에서 더 큰 NAT 내부 서브넷 접두사를 사용할 수 있도록 다음 워크플로를 따라야 합니다.

**_예를 들어 동일한 호스트에서 Windows 컨테이너 기능과 함께 사용되는 Docker 4 Windows - Docker Beta - Linux VM에 대해 자세히 설명합니다. 이 워크플로는 변경될 수 있습니다._**

1. C:\> net stop docker
2. Docker4Windows MobyLinux VM 중지
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *기존의 컨테이너 네트워크를 제거합니다(즉, vSwitch와 NetNat를 삭제하고 정리함).*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24(이 서브넷은 Windows 컨테이너 기능에 사용됨) *nat라는 내부 vSwitch를 만듭니다.*  
   *IP 접두사가 10.0.76.0/24인 “nat”라는 NAT 네트워크를 만듭니다.*  

6. Remove-NetNAT  
   *DockerNAT와 nat NAT 네트워크를 둘 다 제거합니다(내부 vSwitch는 유지).*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17(D4W와 컨테이너에서 공유할 더 큰 NAT 네트워크를 만듦)  
   *더 큰 접두사 10.0.0.0/17을 사용하는 DockerNAT라는 NAT 네트워크를 만듭니다.*  

8. Docker4Windows 실행(MobyLinux.ps1)  
   *내부 vSwitch DockerNAT를 만듭니다.*  
   *IP 접두사가 10.0.75.0/24인 “DockerNAT”라는 NAT 네트워크를 만듭니다.*  

9. Net start docker  
   *Docker는 사용자 정의 NAT 네트워크를 기본값으로 사용하여 Windows 컨테이너를 연결합니다.*  

결과적으로 DockerNAT와 nat라는 두 개의 내부 vSwitch가 만들어집니다. Get-NetNat를 실행하여 확인한 NAT 네트워크(10.0.0.0/17) 하나만 있어야 합니다. Windows 컨테이너의 IP 주소는 Windows HNS(호스트 네트워크 서비스)가 10.0.76.0/24 서브넷에서 할당합니다. 기존 MobyLinux.ps1 스크립트에 따라 Docker 4 Windows의 IP 주소는 10.0.75.0/24 서브넷에서 할당됩니다.


## 참조
[NAT 네트워크](https://en.wikipedia.org/wiki/Network_address_translation)에 대해 자세히 알아보세요.


<!--HONumber=May16_HO5-->


