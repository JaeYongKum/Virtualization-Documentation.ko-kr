---
title: Windows 컨테이너 네트워킹
description: Windows 컨테이너에 대 한 고급 네트워킹.
keywords: Docker, 컨테이너
author: jmesser81
ms.date: 03/27/2018
ms.topic: how-to
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 5278d1d0aefadf81905c843609e1c922f5ca446a
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192690"
---
# <a name="advanced-network-options-in-windows"></a>Windows의 고급 네트워크 옵션

Windows 관련 기능과 기능을 활용 하기 위해 몇 가지 네트워크 드라이버 옵션이 지원 됩니다.

## <a name="switch-embedded-teaming-with-docker-networks"></a>Docker 네트워크를 사용 하 여 포함 된 팀 전환

> 모든 네트워크 드라이버에 적용 됩니다.

옵션으로 여러 네트워크 어댑터 (쉼표로 구분)를 지정 하 여 Docker에서 사용할 컨테이너 호스트 네트워크를 만들 때 [스위치 포함 팀](https://docs.microsoft.com/windows-server/virtualization/hyper-v-virtual-switch/RDMA-and-Switch-Embedded-Teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set) 을 활용할 수 있습니다 `-o com.docker.network.windowsshim.interface` .

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>네트워크의 VLAN ID 설정

> 투명 및 l2bridge 네트워크 드라이버에 적용 됩니다.

네트워크에 대 한 VLAN ID를 설정 하려면 옵션을 `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` 명령에 사용 `docker network create` 합니다. 예를 들어 다음 명령을 사용 하 여 VLAN ID가 11 인 투명 네트워크를 만들 수 있습니다.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
네트워크에 대해 VLAN ID를 설정 하는 경우 해당 네트워크에 연결 되는 모든 컨테이너 끝점에 대해 VLAN 격리를 설정 하 게 됩니다.

> 호스트 네트워크 어댑터 (물리적)가 트렁크 모드에 있는지 확인 하 여 태그가 지정 된 모든 트래픽이 올바른 VLAN의 액세스 모드에서 vNIC (컨테이너 끝점) 포트를 사용 하 여 vSwitch에 의해 처리 되도록 합니다.

## <a name="specify-outboundnat-policy-for-a-network"></a>네트워크에 대 한 OutboundNAT 정책 지정

> L2bridge 네트워크에 적용 됩니다.

일반적으로를 `l2bridge` 사용 하 여 컨테이너 네트워크를 만들 때 `docker network create` 컨테이너 끝점에는 HNS OutboundNAT 정책이 적용 되지 않아 컨테이너가 외부 세계에 도달할 수 없게 됩니다. 네트워크를 만드는 경우 옵션을 사용 하 여 컨테이너에 `-o com.docker.network.windowsshim.enable_outboundnat=<true|false>` 외부 세계에 대 한 액세스를 제공 하는 OUTBOUNDNAT HNS 정책을 적용할 수 있습니다.

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true MyL2BridgeNetwork
```

NAT'ing 발생 하지 않으려는 대상 집합 (예: 컨테이너 간 연결이 필요한 경우)이 있는 경우에는 예외 항목을 지정 해야 합니다.

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true -o com.docker.network.windowsshim.outboundnat_exceptions=10.244.10.0/24
```

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>HNS 서비스에 대 한 네트워크의 이름을 지정 합니다.

> 모든 네트워크 드라이버에 적용 됩니다.

일반적으로를 사용 하 여 컨테이너 네트워크를 만들 때 `docker network create` 사용자가 제공 하는 네트워크 이름은 Docker 서비스에서 사용 되지만 HNS 서비스에서 사용 되지 않습니다. 네트워크를 만드는 경우 옵션을 사용 하 여 HNS 서비스에서 제공 하는 이름을 명령에 지정할 수 있습니다 `-o com.docker.network.windowsshim.networkname=<network name>` `docker network create` . 예를 들어, 다음 명령을 사용 하 여 HNS 서비스에 지정 된 이름을 사용 하 여 투명 한 네트워크를 만들 수 있습니다.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>네트워크를 특정 네트워크 인터페이스에 바인딩

> 'nat'을 제외한 모든 네트워크 드라이버에 적용

Hyper-v 가상 스위치를 통해 연결 된 네트워크를 특정 네트워크 인터페이스에 바인딩하려면 명령에 옵션을 사용 `-o com.docker.network.windowsshim.interface=<Interface>` `docker network create` 합니다. 예를 들어 다음 명령을 사용 하 여 "이더넷 2" 네트워크 인터페이스에 연결 된 투명 네트워크를 만들 수 있습니다.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 참고: *com.docker* 의 값은 다음에서 찾을 수 있는 네트워크 어댑터의 *이름*입니다.

```
PS C:\> Get-NetAdapter
```

## <a name="specify-the-dns-suffix-andor-the-dns-servers-of-a-network"></a>네트워크의 DNS 접미사 및/또는 DNS 서버를 지정 하십시오.

> 모든 네트워크 드라이버에 적용 됩니다.

옵션을 사용 하 여 `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` 네트워크의 dns 접미사를 지정 하 고 옵션을 사용 하 여 `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` 네트워크의 dns 서버를 지정 합니다. 예를 들어 네트워크의 DNS 접미사를 "example.com"로 설정 하 고 네트워크의 DNS 서버를 4.4.4.4 및 8.8.8.8로 설정 하려면 다음 명령을 사용할 수 있습니다.

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

자세한 내용은 [이 문서](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/)를 참조하세요.

## <a name="tips--insights"></a>팁과 고급 정보
커뮤니티에서 제공 하는 Windows 컨테이너 네트워킹에 대 한 일반적인 질문에 따라 유용한 팁과 정보를 제공 하는 목록을 제공 합니다.

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS를 사용 하려면 컨테이너 호스트 컴퓨터에서 i p v 6을 사용 하도록 설정 해야 합니다.
[KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217) HNS의 일부로 Windows 컨테이너 호스트에서 i p v 6을 사용 하도록 설정 해야 합니다. 아래와 같은 오류가 발생 하는 경우 호스트 컴퓨터에서 i p v 6을 사용 하지 않도록 설정할 가능성이 있습니다.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
이 문제를 자동으로 감지/방지 하기 위해 플랫폼 변경에 대해 작업 하 고 있습니다. 현재는 다음과 같은 해결 방법을 사용하여 호스트 컴퓨터에서 IPv6를 사용하도록 설정할 수 있습니다.

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```


#### <a name="linux-containers-on-windows"></a>Windows의 Linux 컨테이너

**새로 만들기:** Microsoft는 _Moby LINUX VM 없이_Linux 및 Windows 컨테이너를 나란히 실행할 수 있도록 노력 하 고 있습니다. 자세한 내용은 [Windows의 Linux 컨테이너에 대 한 블로그 게시물 (LCOW)](https://blog.docker.com/2017/11/docker-for-windows-17-11/) 을 참조 하세요. [시작](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10-linux)하는 방법은 다음과 같습니다.
> 참고: LCOW는 Moby Linux VM을 사용 중단 기본 HNS "nat" 내부 vSwitch를 활용 합니다.

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-ce"></a>Moby Linux Vm은 Windows용 Docker ( [DOCKER CE](https://www.docker.com/community-edition)제품)와 함께 DockerNAT 스위치를 사용 합니다.

Windows 10의 Windows용 Docker (Docker CE 엔진 용 Windows 드라이버)는 ' DockerNAT ' 라는 내부 vSwitch를 사용 하 여 Moby Linux Vm을 컨테이너 호스트에 연결 합니다. Windows에서 Moby Linux Vm을 사용 하는 개발자는 호스트가 HNS 서비스 (Windows 컨테이너에 사용 되는 기본 스위치)에서 생성 된 "nat" vSwitch 대신 DockerNAT vSwitch를 사용 하 고 있음을 인식 해야 합니다.



#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>가상 컨테이너 호스트에서 IP 할당에 DHCP를 사용 하 여 MACAddressSpoofing 사용

컨테이너 호스트가 가상화 되 고 IP 할당에 DHCP를 사용 하려는 경우 가상 컴퓨터의 네트워크 어댑터에서 MACAddressSpoofing을 사용 하도록 설정 해야 합니다. 그렇지 않으면 Hyper-V 호스트는 여러 MAC 주소를 사용하여 VM의 컨테이너에서 들어오는 네트워크 트래픽을 차단합니다. 다음 PowerShell 명령을 사용 하 여 MACAddressSpoofing을 사용 하도록 설정할 수 있습니다.
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
VMware를 하이퍼바이저로 실행 하는 경우이 작업을 수행 하려면 무차별 모드를 사용 하도록 설정 해야 합니다. 자세한 내용은 [여기](https://kb.vmware.com/s/article/1004099) 를 참조 하세요.


#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>단일 컨테이너 호스트에서 여러 투명 네트워크 만들기
둘 이상의 투명 네트워크를 만들려는 경우 외부 Hyper-v 가상 스위치가 바인딩되어야 하는 (가상) 네트워크 어댑터를 지정 해야 합니다. 네트워크에 대 한 인터페이스를 지정 하려면 다음 구문을 사용 합니다.
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME>

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>고정 IP 할당을 사용 하는 경우 *--subnet* 및 *--gateway* 를 지정 해야 합니다.
고정 IP 할당을 사용하는 경우 먼저 네트워크를 만들 때 *--subnet* 및 *--gateway* 매개 변수를 지정해야 합니다. 서브넷 및 게이트웨이 IP 주소는 컨테이너 호스트 즉, 실제 네트워크에 대한 네트워크 설정과 동일해야 합니다. 예를 들어 다음은 투명 네트워크를 만든 다음 고정 IP 할당을 사용 하 여 해당 네트워크에서 끝점을 실행 하는 방법입니다.

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>DHCP IP 할당은 L2Bridge networks에서 지원 되지 않습니다.
L2bridge 드라이버를 사용 하 여 만든 컨테이너 네트워크에는 고정 IP 할당만 지원 됩니다. 위에서 설명한 대로 *--subnet* 및 *--gateway* 매개 변수를 사용 하 여 고정 IP 할당에 대해 구성 된 네트워크를 만들어야 합니다.

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>외부 vSwitch를 활용 하는 네트워크에는 각각 고유한 네트워크 어댑터가 있어야 합니다.
연결에 외부 vSwitch를 사용 하는 여러 네트워크 (예: 투명, L2 브리지, L2 투명)가 동일한 컨테이너 호스트에 생성 되는 경우 각 네트워크에는 자체 네트워크 어댑터가 필요 합니다.

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>중지 됨에 대 한 IP 할당 및 실행 중인 컨테이너
고정 IP 할당은 컨테이너의 네트워크 어댑터에서 직접 수행되며, 컨테이너가 중지됨 상태인 경우에만 수행해야 합니다. 컨테이너가 실행 되는 동안에는 컨테이너 네트워크 어댑터의 "핫 추가" 또는 네트워크 스택에 대 한 변경 (Windows Server 2016)이 지원 되지 않습니다.

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>기존 vSwitch (Docker에 표시 되지 않음)는 투명 네트워크 생성을 차단할 수 있습니다.
투명 네트워크를 만들 때 오류가 발생 하는 경우 시스템에 외부 vSwitch가 있어 Docker에서 자동으로 검색 되지 않으므로 투명 한 네트워크가 컨테이너 호스트의 외부 네트워크 어댑터에 바인딩되지 않도록 할 수 있습니다.

투명 네트워크를 만들 때 Docker는 네트워크에 대한 외부 vSwitch를 만든 다음 스위치를 (외부) 네트워크 어댑터에 바인딩하려고 합니다. 어댑터는 VM 네트워크 어댑터 또는 실제 네트워크 어댑터일 수 있습니다. vSwitch가 컨테이너 호스트에 이미 생성되고 *Docker에 표시되는 경우 *Windows Docker 엔진은 새 스위치를 만드는 대신 해당 스위치를 사용합니다. 그러나 vSwitch가 대역 외에서 생성되고(HYper-V 관리자 또는 PowerShell을 사용하여 컨테이너 호스트에서 생성) 아직 Docker에 표시되지 않는 경우, Windows Docker 엔진은 새 vSwitch를 만들려고 시도한 다음 새 스위치를 컨테이너 호스트 외부 네트워크 어댑터에 연결할 수 없습니다(네트워크 어댑터가 이미 대역 외에서 생성된 스위치에 연결되어 있기 때문).

예를 들어 Docker 서비스가 실행 중일 때 호스트에 새 vSwitch를 먼저 만든 다음 투명 네트워크를 만들려고 하면 이 문제가 발생합니다. 이 경우 Docker에서는 사용자가 만든 스위치를 인식할 수 없으며 투명 네트워크에 대한 새 vSwitch를 만듭니다.

이 문제를 해결하는 데는 다음과 같은 세 가지 방법이 있습니다.

* 물론 대역 외에서 만든 vSwitch를 삭제할 수 있습니다. 그러면 Docker에서 문제 없이 새 vSwitch를 만들고 호스트 네트워크 어댑터에 연결할 수 있습니다. 이 방법을 선택하기 전에 대역 외 vSwitch를 다른 서비스(예: Hyper-V)에서 사용하고 있지 않은지 확인하세요.
* 또는 대역 외에서 생성된 외부 vSwitch를 사용하려는 경우 Docker 및 HNS 서비스를 다시 시작하여 *Docker에 스위치가 표시되도록 하세요.*
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* 또 다른 방법으로는 '-o com.docker.network.windowsshim.interface' 옵션을 사용하여 투명 네트워크의 외부 vSwitch를 컨테이너 호스트에서 아직 사용하지 않는 특정 네트워크 어댑터(대역 외에서 생성된 vSwitch에서 사용하는 것 외의 네트워크 어댑터)에 바인딩하는 것입니다. '-O ' 옵션에 대 한 자세한 내용은이 문서의 [단일 컨테이너 호스트에서 여러 투명 네트워크 만들기](advanced.md#creating-multiple-transparent-networks-on-a-single-container-host) 섹션을 참조 하세요.


## <a name="windows-server-2016-work-arounds"></a>Windows Server 2016 해결 방법

새 기능 및 개발을 계속 해 서 추가 해도 이러한 기능 중 일부는 이전 플랫폼으로 다시 이식할 수 없습니다. 대신, Windows 10 및 Windows Server에 대 한 최신 업데이트를 보려면 "학습에 대해 알아봅니다."를 수행 하는 것이 가장 좋습니다.  아래 섹션에는 Windows Server 2016 및 이전 버전의 Windows 10에 적용 되는 몇 가지 해결 방법 및 주의 사항이 나와 있습니다 (즉, 1704 작성자 업데이트 전).

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>WS2016 Container Host의 여러 NAT 네트워크

새 NAT 네트워크에 대한 분할은 더 큰 내부 NAT 네트워킹 접두사 아래에 만들어야 합니다. PowerShell에서 다음 명령을 실행하고 "InternalIPInterfaceAddressPrefix" 필드를 참조하여 접두사를 찾을 수 있습니다.

```
PS C:\> Get-NetNAT
```

예를 들어 호스트의 NAT 네트워크 내부 접두사는 172.16.0.0/16 일 수 있습니다. 이 경우 Docker는 *172.16.0.0/16 접두사의 하위 집합인* 경우에만 추가 NAT 네트워크를 만드는 데 사용할 수 있습니다. 예를 들어 IP 접두사 172.16.1.0/24 (게이트웨이, 172.16.1.1) 및 172.16.2.0/24 (게이트웨이, 172.16.2.1)를 사용 하 여 두 개의 NAT 네트워크를 만들 수 있습니다.

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
