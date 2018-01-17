---
title: "Windows 컨테이너 네트워킹"
description: "Windows 컨테이너에 대한 네트워킹을 구성합니다."
keywords: "Docker, 컨테이너"
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 394aa58c3421e512d005f59d5bd30667f1c26f16
ms.sourcegitcommit: 6eefb890f090a6464119630bfbdc2794e6c3a3df
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/18/2017
---
# <a name="windows-container-networking"></a>Windows 컨테이너 네트워킹
> ***일반 Docker 네트워킹 명령, 옵션 및 구문에 대한 내용은 [Docker 컨테이너 네트워킹](https://docs.docker.com/engine/userguide/networking/)을 참조하세요.*** 이 문서에 설명된 경우를 제외한 모든 Docker 네트워킹 명령은 Linux와 동일한 구문을 통해 Windows에서 지원됩니다. 그러나 Windows와 Linux의 네트워크 스택이 서로 다르므로 일부 Linux 네트워크 명령은 Windows에서 지원되지 않습니다(예: ifconfig).

## <a name="basic-networking-architecture"></a>기본 네트워킹 아키텍처
이 항목에서는 Docker가 Windows에서 네트워크를 만들고 관리하는 방식에 대해 개략적으로 설명합니다. Windows 컨테이너는 네트워킹에 있어서 가상 컴퓨터와 유사하게 작동합니다. 각 컨테이너에는 Hyper-V 가상 스위치(vSwitch)에 연결되는 가상 네트워크 어댑터(vNIC)가 있습니다. Windows는 Docker를 통해 만들 수 있는 5가지 네트워킹 드라이버 또는 모드인 *nat*, *overlay*, *transparent*, *l2bridge* 및 *l2tunnel*을 지원합니다. 실제 네트워크 인프라와 단일 및 다중 호스트 네트워킹 요구 사항에 따라 가장 적합한 네트워크 드라이버를 선택해야 합니다.

<figure>
  <img src="media/windowsnetworkstack-simple.png">
</figure>  

Docker 엔진은 처음으로 실행되면 내부 vSwitch와 `WinNAT`이라고 하는 Windows 구성 요소를 사용하는 기본 NAT 네트워크 'nat'를 만듭니다. 호스트에 PowerShell 또는 Hyper-V 관리자를 통해 만든 기존 외부 vSwitch가 있는 경우 이러한 외부 스위치 역시 *transparent* 네트워크 드라이버를 사용하여 Docker에 제공되며 ``docker network ls`` 명령을 실행할 때 볼 수 있습니다.  

<figure>
  <img src="media/docker-network-ls.png">
</figure>

> - ***내부*** vSwitch는 컨테이너 호스트의 네트워크 어댑터에 직접 연결되지 않습니다. 

> - ***외부*** vSwitch는 컨테이너 호스트의 네트워크 어댑터에 _직접_ 연결됩니다.  

<figure>
  <img src="media/get-vmswitch.png">
</figure>

'nat' 네트워크는 Windows에서 실행되는 컨테이너의 기본 네트워크입니다. 특정 네트워크 구성을 구현하는 플래그 또는 인수 없이 Windows에서 실행되는 모든 컨테이너는 기본 'nat' 네트워크에 연결되고, 'nat' 네트워크 내부 접두사 IP 범위의 IP 주소가 자동으로 할당합니다. 'nat'에 사용되는 기본 내부 IP 접두사는 172.16.0.0/16입니다. 


## <a name="windows-container-network-drivers"></a>Windows 컨테이너 네트워크 드라이버  

Windows에서 Docker를 사용하여 만든 기본 'nat' 네트워크를 활용하는 것 외에도 사용자는 사용자 지정 컨테이너 네트워크를 정의할 수 있습니다. 사용자 정의 네트워크는 Docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) 명령을 사용하여 만들 수 있습니다. Windows에서 다음과 같은 유형의 네트워크 드라이버를 사용할 수 있습니다.

- **nat** – 'nat' 드라이버를 사용하여 만든 네트워크에 연결된 컨테이너는 사용자가 지정한(``--subnet``) IP 접두사의 IP 주소를 받습니다. 컨테이너 호스트에서 컨테이너 끝점으로의 포트 전달/매핑이 지원됩니다.
> 참고: 이제 Windows 10 크리에이터스 업데이트에서 여러 NAT 네트워크가 지원됩니다! 

- **transparent** – 'transparent' 드라이버를 사용하여 만든 네트워크에 연결된 컨테이너는 실제 네트워크에 직접 연결됩니다. 실제 네트워크의 IP는 외부 DHCP 서버를 사용하여 정적으로(사용자가 지정한 ``--subnet`` 옵션이 필요함) 또는 동적으로 할당할 수 있습니다. 

- **overlay** - __새로 추가되었습니다!__  Docker 엔진이 [Swarm 모드](./swarm-mode.md)로 실행되면 오버레이 네트워크에 연결된 컨테이너는 여러 컨테이너 호스트에서 동일한 네트워크에 연결된 다른 컨테이너와 통신할 수 있습니다. Swarm 클러스터에 만들어지는 각 오버레이 네트워크는 개인 IP 접두사에 의해 정의된 자체 IP 서브넷으로 만들어집니다. 오버레이 네트워크 드라이버는 VXLAN 캡슐화를 사용합니다.
> [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217)이 설치된 Windows Server 2016 또는 Windows 10 크리에이터스 업데이트가 필요합니다. 

- **l2bridge** - 'l2bridge' 드라이버를 사용하여 만든 네트워크에 연결된 컨테이너는 컨테이너 호스트와 동일한 IP 서브넷에 배치됩니다. IP 주소는 컨테이너 호스트와 동일한 접두사에서 정적으로 할당해야 합니다. 호스트의 모든 컨테이너 끝점은 수신 및 송신 시 계층 2 주소 변환(MAC 다시 쓰기) 작업 때문에 동일한 MAC 주소를 갖게 됩니다.
> Windows Server 2016 또는 Windows 10 크리에이터스 업데이트가 필요합니다.

- **l2tunnel** - _이 드라이버는 Microsoft 클라우드 스택에만 사용해야 합니다._

> Microsoft SDN 스택을 사용하여 컨테이너 끝점을 오버레이 가상 네트워크에 연결하는 방법을 알아보려면 [가상 네트워크에 컨테이너 연결](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network) 항목을 참조하세요.

> Windows 10 크리에이터스 업데이트에는 실행 중인 컨테이너에 새 컨테이너 끝점을 추가하는 플랫폼 지원(즉, '핫 애드')이 도입되었습니다 이는 종단 간 [미해결 Docker pull 요청](https://github.com/docker/libnetwork/pull/1661)에 도움이 될 것입니다.

## <a name="network-topologies-and-ipam"></a>네트워크 토폴로지 및 IPAM
아래 표에서는 각 네트워크 드라이버의 내부(컨테이너 간) 및 외부 연결에 대한 네트워크 연결을 제공하는 방법을 보여 줍니다.

<figure>
  <img src="media/network-modes-table.png">
</figure>

### <a name="ipam"></a>IPAM 
각 네트워킹 드라이버에 서로 다른 IP 주소가 할당됩니다. Windows는 HNS(호스트 네트워킹 서비스)를 사용하여 nat 드라이버에 대한 IPAM을 제공하고 Docker Swarm 모드(내부 KVS)에서 작동하여 오버레이에 대한 IPAM을 제공합니다. 다른 네트워크 드라이버는 외부 IPAM을 사용합니다.

<figure>
  <img src="media/ipam.png">
</figure>

# <a name="details-on-windows-container-networking"></a>Windows 컨테이너 네트워킹에 대한 세부 정보

## <a name="isolation-namespace-with-network-compartments"></a>네트워크 구획으로 격리(네임스페이스)
각 컨테이너 끝점은 Linux의 네트워크 네임스페이스와 비슷한 자체 __네트워크 구획__에 배치됩니다. 관리 호스트 vNIC 및 호스트 네트워크 스택은 기본 네트워크 구획에 배치됩니다. 동일한 호스트에서 컨테이너 간에 네트워크 격리를 적용하려면 컨테이너에 대한 네트워크 어댑터가 설치되는 각 Windows Server 및 Hyper-V 컨테이너에 대해 네트워크 구획을 만듭니다. Windows Server 컨테이너는 호스트 vNIC를 사용하여 가상 스위치에 연결합니다. Hyper-V 컨테이너는 가상 VM NIC(유틸리티 VM에 노출되지 않음)를 사용하여 가상 스위치에 연결합니다. 

<figure>
  <img src="media/network-compartment-visual.png">
</figure>

```powershell 
Get-NetCompartment
```

## <a name="windows-firewall-security"></a>Windows 방화벽 보안

Windows 방화벽은 포트 ACL을 통해 네트워크 보안을 적용하는 데 사용됩니다.

> 참고: 기본적으로 오버레이 네트워크에 연결된 모든 컨테이너 끝점은 모두 허용 규칙을 갖습니다.   

<figure>
  <img src="media/windows-firewall-containers.png">
</figure>

## <a name="container-network-management-with-host-network-service"></a>호스트 네트워크 서비스를 통해 컨테이너 네트워크 관리

아래 그림은 HNS(호스트 네트워킹 서비스)와 HCS(호스트 계산 서비스)가 함께 작동하여 컨테이너를 만들고 네트워크에 끝점을 연결하는 방식을 보여 줍니다. 

<figure>
  <img src="media/HNS-Management-Stack.png">
</figure>

# <a name="advanced-network-options-in-windows"></a>Windows의 고급 네트워크 옵션
Windows 관련 기능 및 특징을 활용할 수 있도록 여러 가지 네트워크 드라이버 옵션이 지원됩니다. 

## <a name="switch-embedded-teaming-with-docker-networks"></a>네트워크를 사용하는 스위치 포함 팀

> 모든 네트워크 드라이버에 적용 

`-o com.docker.network.windowsshim.interface` 옵션으로 여러 네트워크 어댑터(쉼표로 구분)를 지정하여 Docker에서 사용할 컨테이너 호스트 네트워크를 만들 때 [스위치 포함 팀](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set)을 활용할 수 있습니다. 

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>네트워크의 VLAN ID 설정

> transparent 및 l2bridge 네트워크 드라이버에 적용 

네트워크의 VLAN ID를 설정하려면 `docker network create` 명령에 `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` 옵션을 사용합니다. 예를 들어 다음 명령을 사용하여 VLAN ID가 11인 투명 네트워크를 만들 수 있습니다.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
네트워크의 VLAN ID를 설정하면 해당 네트워크에 연결될 모든 컨테이너 끝점에 대해 VLAN 격리가 설정됩니다.

> vSwitch가 올바른 VLAN에서 액세스 모드로 vNIC(컨테이너 끝점)를 사용하여 태그가 지정된 트래픽을 모두 처리할 수 있도록 호스트 네트워크 어댑터(실제)가 트렁크 모드에 있는지 확인합니다.

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>HNS 서비스에 네트워크의 이름 지정

> 모든 네트워크 드라이버에 적용 

일반적으로 `docker network create`를 사용하여 컨테이너 네트워크를 만들면, 제공하는 네트워크 이름이 Docker 서비스에 사용되지만 HNS 서비스에는 사용되지 않습니다. 네트워크를 만들 경우 `docker network create` 명령에 `-o com.docker.network.windowsshim.networkname=<network name>` 옵션을 사용하여 HNS 서비스에서 이름을 제공하도록 지정할 수 있습니다. 예를 들어 다음 명령을 사용하여 HNS 서비스에 지정된 이름으로 투명 네트워크를 만들 수 있습니다.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>특정 네트워크 인터페이스에 네트워크 바인딩

> 'nat'을 제외한 모든 네트워크 드라이버에 적용  

Hyper-V 가상 스위치를 통해 연결된 네트워크를 특정 네트워크 인터페이스에 바인딩하려면 `docker network create` 명령에 `-o com.docker.network.windowsshim.interface=<Interface>` 옵션을 사용합니다. 예를 들어, 다음 명령을 사용하여 "Ethernet 2" 네트워크 인터페이스에 연결된 투명 네트워크를 만들 수 있습니다.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 참고: *com.docker.network.windowsshim.interface*의 값은 다음 명령으로 찾을 수 있는 네트워크 어댑터의 *이름*입니다.

>```
PS C:\> Get-NetAdapter
```
## Specify the DNS Suffix and/or the DNS Servers of a Network

> Applies to all network drivers 

Use the option, `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` to specify the DNS suffix of a network, and the option, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` to specify the DNS servers of a network. For example, you might use the following command to set the DNS suffix of a network to "example.com" and the DNS servers of a network to 4.4.4.4 and 8.8.8.8:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## VFP

The Virtual Filtering Platform (VFP) extension is a Hyper-V virtual switch, forwarding extension used to enforce network policy and manipulate packets. For instance, VFP is used by the 'overlay' network driver to perform VXLAN encapsulation and by the 'l2bridge' driver to perform MAC re-write on ingresss and egress. The VFP extension is only present on Windows Server 2016 and Windows 10 Creators Update. To check and see if this is running correctly a user run two commands:

```powershell
Get-Service vfpext

# This should indicate the extension is Running: True 
Get-VMSwitchExtension  -VMSwitchName <vSwitch Name> -Name "Microsoft Azure VFP Switch Extension"
```

## <a name="tips--insights"></a>팁과 고급 정보
다음은 커뮤니티에서 Windows 컨테이너 네트워킹에 대해 자주 묻는 질문을 바탕으로 작성한 유용한 팁과 고급 정보입니다.

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS를 사용하려면 컨테이너 호스트 컴퓨터에서 IPv6를 사용하도록 설정 
[KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217)에서 도입된 HNS를 사용하려면 Windows 컨테이너 호스트에서 IPv6를 사용하도록 설정해야 합니다. 아래와 같은 오류가 발생할 경우 호스트 컴퓨터에서 IPv6가 비활성화된 것이 원인일 수 있습니다.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
자동으로 이 문제를 감지/방지하도록 플랫폼 변경 작업을 진행 중입니다. 현재는 다음과 같은 해결 방법을 사용하여 호스트 컴퓨터에서 IPv6를 사용하도록 설정할 수 있습니다.

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition-instead-of-hns-internal-vswitch"></a>Moby Linux VM은 HNS 내부 vSwitch 대신 Windows용 Docker와 함께 DockerNAT 스위치([Docker CE](https://www.docker.com/community-edition) 제품)를 사용합니다. 
Windows 10에서 Windows용 Docker(Docker CE 엔진용 Windows 드라이버)는 'DockerNAT'라고 하는 내부 vSwitch를 사용하여 Moby Linux VM을 컨테이너 호스트에 연결합니다. Windows에서 Moby Linux VM을 사용하는 개발자는 호스트가 HNS 서비스에서 생성한 vSwitch(Windows 컨테이너에 사용되는 기본 스위치)가 아닌 DockerNAT vSwitch를 사용한다는 사실을 알고 있어야 합니다. 

#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>DHCP를 사용하여 가상 컨테이너 호스트에 IP를 할당하려면 MACAddressSpoofing을 사용하도록 설정 
컨테이너 호스트가 가상화되었으며 DHCP를 사용하여 IP를 할당하려는 경우 가상 컴퓨터의 네트워크 어댑터에서 MACAddressSpoofing을 사용하도록 설정해야 합니다. 그렇지 않으면 Hyper-V 호스트는 여러 MAC 주소를 사용하여 VM의 컨테이너에서 들어오는 네트워크 트래픽을 차단합니다. 다음 PowerShell 명령을 사용하여 MACAddressSpoofing을 사용하도록 설정할 수 있습니다.
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
사용자 하이퍼바이저로 VMware를 실행하는 경우 이를 작동하도록 하려면 promiscuous 모드를 사용하도록 설정해야 합니다. 자세한 내용은 [여기](https://kb.vmware.com/s/article/1004099)에서 확인할 수 있습니다.
#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>단일 컨테이너 호스트에 여러 투명 네트워크 만들기
투명 네트워크를 두 개 이상 만들려면 외부 Hyper-V 가상 스위치가 바인딩될 (가상) 네트워크 어댑터를 지정해야 합니다. 네트워크 인터페이스를 지정하려면 다음 구문을 사용합니다.
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>고정 IP 할당을 사용하는 경우 *--subnet* 및 *--gateway*를 지정해야 합니다.
고정 IP 할당을 사용하는 경우 먼저 네트워크를 만들 때 *--subnet* 및 *--gateway* 매개 변수를 지정해야 합니다. 서브넷 및 게이트웨이 IP 주소는 컨테이너 호스트, 즉 실제 네트워크에 대한 네트워크 설정과 동일해야 합니다. 예를 들어 다음과 같이 투명 네트워크를 만든 후 고정 IP 할당을 사용하여 해당 네트워크에서 끝점을 실행할 수 있습니다.

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>DHCP IP는 L2Bridge 네트워크에서 지원되지 않음
l2bridge 드라이버를 사용하여 만든 컨테이너 네트워크에서는 고정 IP 할당만 지원됩니다. 위에서 언급했듯이, *--subnet* 및 *--gateway* 매개 변수를 사용하여 고정 IP 할당에 대해 구성된 네트워크를 만들어야 합니다.

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>외부 vSwitch를 활용하는 네트워크마다 자체 네트워크 어댑터가 있어야 함
연결용 vSwitch(예: Transparent, L2 Bridge, L2 Transparent)를 사용하는 여러 네트워크를 동일한 컨테이너 호스트에 만드는 경우 네트워크마다 자체 네트워크 어댑터가 있어야 합니다. 

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>중지된 컨테이너 및 실행 중인 컨테이너에 IP 할당
고정 IP 할당은 컨테이너의 네트워크 어댑터에서 직접 수행되며, 컨테이너가 중지됨 상태인 경우에만 수행해야 합니다. Windows Server 2016에서는 컨테이너가 실행되는 동안 컨테이너 네트워크 어댑터의 핫 애드 또는 네트워크 스택 변경이 지원되지 않습니다.
> 참고: 이제 플랫폼에서 "핫 애드"를 지원하므로 이 동작은 Windows 10 크리에이터스 업데이트에서 변경됩니다. 이 [미해결 Docker pull 요청](https://github.com/docker/libnetwork/pull/1661)이 병합되면 이 기능을 통해 E2E를 사용할 수 있습니다.

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>기존 vSwitch(Docker에서 보이지 않음)가 투명 네트워크 생성을 차단할 수 있음
투명 네트워크를 만드는 중 오류가 발생하는 경우 Docker에서 자동으로 검색되지 않고 투명 네트워크가 컨테이너 호스트의 외부 네트워크 어댑터에 바인딩되지 못하게 방해하는 외부 vSwitch가 시스템에 있는 것일 수 있습니다. 

투명 네트워크를 만들 때 Docker는 네트워크에 대한 외부 vSwitch를 만든 다음 스위치를 (외부) 네트워크 어댑터에 바인딩하려고 합니다. 어댑터는 VM 네트워크 어댑터 또는 실제 네트워크 어댑터일 수 있습니다. vSwitch가 컨테이너 호스트에 이미 생성되고 *Docker에 표시되는 경우* Windows Docker 엔진은 새 스위치를 만드는 대신 해당 스위치를 사용합니다. 그러나 vSwitch가 대역 외에서 생성되고(Hyper-V 관리자 또는 PowerShell을 사용하여 컨테이너 호스트에서 생성) 아직 Docker에 표시되지 않는 경우, Windows Docker 엔진은 새 vSwitch를 만들려고 시도한 다음 새 스위치를 컨테이너 호스트 외부 네트워크 어댑터에 연결할 수 없습니다(네트워크 어댑터가 이미 대역 외에서 생성된 스위치에 연결되어 있기 때문).

예를 들어 Docker 서비스가 실행 중일 때 호스트에 새 vSwitch를 먼저 만든 다음 투명 네트워크를 만들려고 하면 이 문제가 발생합니다. 이 경우 Docker에서는 사용자가 만든 스위치를 인식할 수 없으며 투명 네트워크에 대한 새 vSwitch를 만듭니다.

이 문제를 해결하는 데는 다음과 같은 세 가지 방법이 있습니다.

* 물론 대역 외에서 만든 vSwitch를 삭제할 수 있습니다. 그러면 Docker에서 문제 없이 새 vSwitch를 만들고 호스트 네트워크 어댑터에 연결할 수 있습니다. 이 방법을 선택하기 전에 대역 외 vSwitch를 다른 서비스(예: Hyper-V)에서 사용하고 있지 않은지 확인하세요.
* 또는 대역 외에서 생성된 외부 vSwitch를 사용하려는 경우 Docker 및 HNS 서비스를 다시 시작하여 *Docker에 스위치가 표시되도록 하세요.*
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* 또 다른 방법으로는 '-o com.docker.network.windowsshim.interface' 옵션을 사용하여 투명 네트워크의 외부 vSwitch를 컨테이너 호스트에서 아직 사용하지 않는 특정 네트워크 어댑터(대역 외에서 생성된 vSwitch에서 사용하는 것 외의 네트워크 어댑터)에 바인딩하는 것입니다. '-o' 옵션은 이 문서의 [투명 네트워크](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network) 섹션에 자세히 설명되어 있습니다.


## <a name="unsupported-features-and-network-options"></a>지원되지 않는 기능과 네트워크 옵션 

다음 네트워킹 옵션은 Windows에서 지원되지 않으며 ``docker run``에 전달할 수 없습니다.
 * 컨테이너 연결(예: ``--link``) - _대안은 서비스 검색 사용_
 * IPv6 주소(예: ``--ip6``)
 * DNS 옵션(예: ``--dns-option``)
 * 다중 DNS 검색 도메인(예: ``--dns-search``)
 
다음 네트워킹 옵션 및 기능은 Windows에서 지원되지 않으며 ``docker network create``에 전달할 수 없습니다.
 * --aux-address
 * --internal
 * --ip-range
 * --ipam-driver
 * --ipam-opt
 * --ipv6 

다음 네트워킹 옵션은 Docker 서비스에서 지원되지 않습니다.
* 데이터 평면 암호화(예: ``--opt encrypted``) 


## <a name="windows-server-2016-work-arounds"></a>Windows Server 2016 해결 방법 

계속해서 새 기능을 추가하고 개발에 힘을 쏟고 있지만 일부 기능은 이전 플랫폼으로 이식하지 않을 예정입니다. 가장 좋은 방법은 최신 Windows 10 및 Windows Server 업데이트로 "갈아타는" 것입니다.  아래 섹션에는 Windows Server 2016 및 이전 버전의 Windows 10(즉, 1704 크리에이터스 업데이트 이전 버전)에 적용되는 몇 가지 해결 방법과 주의 사항이 나열되어 있습니다.

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>WS2016 컨테이너 호스트의 다중 NAT 네트워크

새 NAT 네트워크에 대한 파티션은 더 큰 내부 NAT 네트워킹 접두사 아래에 만들어야 합니다. PowerShell에서 다음 명령을 실행하고 "InternalIPInterfaceAddressPrefix" 필드를 참조하여 접두사를 찾을 수 있습니다.

```
PS C:\> Get-NetNAT
```

예를 들어 호스트의 NAT 네트워크 내부 접두사는 172.16.0.0/16일 수 있습니다. 이 경우 NAT 네트워크가 *172.16.0.0/16 접두사의 하위 집합인 경우* Docker를 사용하여 추가 NAT 네트워크를 만들 수 있습니다. 예를 들어 IP 접두사 172.16.1.0/24(게이트웨이, 172.16.1.1) 및 172.16.2.0/24(게이트웨이, 172.16.2.1)를 사용하여 NAT 네트워크를 두 개 만들 수 있습니다.

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

새로 만든 네트워크는 다음을 사용하여 나열할 수 있습니다.
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/)는 해당 네트워크를 사용할 컨테이너/서비스와 함께 컨테이너 네트워크를 정의하고 구성하는 데 사용할 수 있습니다. Compose '네트워크' 키는 컨테이너를 연결할 네트워크 정의에서 최상위 키로 사용됩니다. 예를 들어 아래 구문에서는 지정한 Compose 파일에 정의된 모든 컨테이너/서비스에 대해 ‘기본’ 네트워크가 되도록 Docker에서 만든 기존 NAT 네트워크를 정의합니다.

```
networks:
 default:
  external:
   name: "nat"
```

마찬가지로, 사용자 지정 NAT 네트워크를 정의하는 데 다음 구문을 사용할 수 있습니다.

> 참고: 아래 예에 정의된 '사용자 지정 NAT 네트워크'는 컨테이너 호스트의 기존 NAT 내부 접두사의 파티션으로 정의됩니다. 자세한 컨텍스트는 위의 '여러 NAT 네트워크' 섹션을 참조하세요.

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Docker Compose를 사용하여 컨테이너 네트워크 정의/구성에 대한 자세한 내용은 [Compose File reference](https://docs.docker.com/compose/compose-file/)(Compose 파일 참조)를 참조하세요.

### <a name="service-discovery"></a>서비스 검색
서비스 검색은 특정 Windows 네트워크 드라이버에만 지원됩니다.

|  | 로컬 서비스 검색  | 글로벌 서비스 검색 |
| :---: | :---------------     |  :---                |
| nat | 예 | 해당 없음 |  
| overlay | 예 | 예 |
| transparent | 아니요 | 아니요 |
| l2bridge | 아니요 | 아니요 |


