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
translationtype: Human Translation
ms.sourcegitcommit: 23d4b665da627f35cf5fce49c3c9974d0ef287dd
ms.openlocfilehash: e56a5b984cc1c42e27628d00a5cd532788aef11c
ms.lasthandoff: 02/10/2017

---

# 컨테이너 네트워킹

Windows 컨테이너는 네트워킹에 관해서 가상 컴퓨터와 유사하게 작동합니다. 각 컨테이너에는 가상 네트워크 어댑터(vNIC)가 있고 이것은 가상 스위치(vSwitch)에 연결되며 이를 통해 인바운드 및 아웃바운드 트래픽이 전달됩니다. 동일한 호스트에서 컨테이너 간에 격리를 적용하려면 컨테이너에 대한 네트워크 어댑터가 설치되는 각 Windows Server 및 Hyper-V 컨테이너에 대해 네트워크 구획을 만듭니다. Windows Server 컨테이너는 호스트 vNIC를 사용하여 가상 스위치에 연결합니다. Hyper-V 컨테이너는 가상 VM NIC(유틸리티 VM에 노출되지 않음)를 사용하여 가상 스위치에 연결합니다.

Windows 컨테이너는 *nat*, *transparent*, *l2bridge* 및 *l2tunnel*의 네 가지 다른 네트워킹 드라이버 또는 모드를 지원합니다. 실제 네트워크 인프라와 단일 및 다중 호스트 네트워킹 요구 사항에 따라 요구에 가장 적합한 네트워크 모드를 선택해야 합니다.

Docker 엔진은 docker 서비스가 처음으로 실행될 때 기본적으로 NAT 네트워크를 만듭니다. 만들어지는 기본 내부 IP 접두사는 172.16.0.0/12입니다. 컨테이너 끝점은 자동으로 이 기본 네트워크에 연결되고 내부 접두사에서 IP 주소가 할당됩니다.

> 참고: 컨테이너 호스트 IP가 이와 동일한 접두사에 있는 경우 아래 설명된 대로 NAT 내부 IP 접두사를 변경해야 합니다.

동일한 컨테이너 호스트에서 다른 드라이버(예: transparent, l2bridge)를 사용하여 추가 네트워크를 만들 수 있습니다. 아래 표에서는 각 모드의 내부(컨테이너 간) 및 외부 연결에 대한 네트워크 연결을 제공하는 방법을 보여 줍니다.

- **NAT(Network Address Translation)** – 각 컨테이너가 내부 개인 IP 접두사(예: 172.16.0.0/12)에서 IP 주소를 수신합니다. 컨테이너 호스트와 컨테이너 끝점 간 포트 전달/매핑이 지원됩니다.

- **Transparent**– 각 컨테이너 끝점이 실제 네트워크에 직접 연결됩니다. 실제 네트워크의 IP는 외부 DHCP 서버를 사용하여 정적으로나 동적으로 할당할 수 있습니다.

- **[신규!] 오버레이** - Docker 엔진이 [Swarm 모드](./swarm-mode.md)로 실행될 경우 VXLAN 기술을 기반으로 한 오버레이 네트워크를 사용하여 여러 컨테이너 호스트의 컨테이너 끝점을 연결할 수 있습니다. Swarm 클러스터에 만들어지는 각 오버레이 네트워크는 개인 IP 접두사에 의해 정의된 자체 IP 서브넷으로 만들어집니다.

- **L2 브리지** - 각 컨테이너 끝점이 컨테이너 호스트와 동일한 IP 서브넷에 있습니다. IP 주소는 컨테이너 호스트와 동일한 접두사에서 정적으로 할당해야 합니다. 호스트의 모든 컨테이너 끝점은 계층 2 주소 변환으로 인해 MAC 주소가 동일합니다.

- **L2 터널** - _이 모드는 Microsoft 클라우드 스택에서만 사용되어야 합니다._

> Microsoft SDN 스택을 사용하여 컨테이너 끝점을 오버레이 가상 네트워크에 연결하는 방법을 알아보려면 [가상 네트워크에 컨테이너 연결](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network) 항목을 참조하세요.

## 단일 노드

|  | 컨테이너-컨테이너 | 컨테이너-외부 |
| :---: | :---------------     |  :---                |
| nat | Hyper-V 가상 스위치를 통한 브리지된 연결 | 주소 변환이 적용된 WinNAT를 통해 라우트됨 |
| transparent | Hyper-V 가상 스위치를 통한 브리지된 연결 | 실제 네트워크에 직접 액세스 |
| overlay | VXLAN 캡슐화는 Hyper-V 가상 스위치의 VFP 전달 확장에서 발생하고, *호스트 내* 통신은 Hyper-V 가상 스위치를 통한 브리지된 연결로 발생함 | 주소 변환이 적용된 WinNAT를 통해 라우트됨
| l2bridge | Hyper-V 가상 스위치를 통한 브리지된 연결|  MAC 주소 변환을 사용하여 실제 네트워크에 액세스|  



## 다중 노드

|  | 컨테이너-컨테이너 | 컨테이너-외부 |
| :---: | :----       | :---------- |
| nat | 외부 컨테이너 호스트 IP 및 포트를 참조해야 함. 주소 변환이 적용된 WinNAT를 통해 라우트됨 | 외부 컨테이너 호스트를 참조해야 함. 주소 변환이 적용된 WinNAT를 통해 라우트됨 |
| transparent | 컨테이너 IP 끝점을 직접 참조해야 함 | 실제 네트워크에 직접 액세스 |
| overlay | VXLAN 캡슐화는 Hyper-V 가상 스위치의 VFP 전달 확장에서 발생하고, *호스트 간* 통신은 IP 끝점을 직접 참조함 | 주소 변환이 적용된 WinNAT를 통해 라우트됨| 
| l2bridge | 컨테이너 IP 끝점을 직접 참조해야 함| MAC 주소 변환을 사용하여 실제 네트워크에 액세스|


## 네트워크 만들기

### (기본값) NAT 네트워크

Windows docker 엔진은 IP 접두사 172.16.0.0/12를 사용하여 기본 NAT 네트워크(Docker 이름 ‘nat’)를 만듭니다. 사용자가 특정 IP 접두사를 사용하여 NAT 네트워크를 만들려는 경우 C:\ProgramData\Docker\config\daemon.json에 있는 Docker 구성 daemon.json 파일(없는 경우 만듦)에서 옵션을 변경하여 두 가지 작업 중 하나를 수행할 수 있습니다.
 1. 지정된 IP 접두사 및 일치를 사용하여 기본 NAT 네트워크를 만드는 _"fixed-cidr": "< IP Prefix > / Mask"_ 옵션 사용
 2. 기본 네트워크를 만들지 않는 _"bridge": "none"_ 옵션 사용. 사용자는 *docker network create -d <driver>* 명령을 사용하여 원하는 드라이버로 사용자 정의 네트워크를 만들 수 있습니다.

이러한 구성 옵션 중 하나를 수행하기 전에 Docker 서비스를 먼저 중지해야 하고 기존 NAT 네트워크를 모두 삭제해야 합니다.

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

daemon.json 파일에 "fixed-cidr" 옵션을 추가한 경우 Docker 엔진은 지정된 사용자 지정 IP 접두사 및 마스크를 사용하여 사용자 정의 NAT 네트워크를 만듭니다. 그렇지 않고 "bridge:none" 옵션을 추가한 경우 네트워크를 수동으로 만들어야 합니다.

```none
# Create a user-defined NAT network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

기본적으로 컨테이너 끝점은 기본 ‘nat’ 네트워크에 연결됩니다. daemon.json에 "bridge:none"을 지정하여 ‘nat’ 네트워크가 만들어지지 않았거나 다른 사용자 정의 네트워크에 액세스해야 하는 경우 사용자는 docker run 명령에 *--network* 매개 변수를 지정할 수 있습니다.

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### 포트 매핑

NAT 네트워크에 연결된 컨테이너 내부에서 실행되는 응용 프로그램에 액세스하려면 컨테이너 호스트와 컨테이너 끝점 사이에 포트 매핑을 만들어야 합니다. 이러한 매핑은 컨테이너 생성 시나 컨테이너가 중지된 상태일 때 지정해야 합니다.

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Creates a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

또는 -p 매개 변수를 사용하거나 Dockerfile에 EXPOSE 명령과 함께 -P 매개 변수를 사용하여 동적 포트 매핑을 지정할 수도 있습니다. 지정하지 않으면 임의로 선택한 사용 후 삭제 포트가 컨테이너 호스트에서 선택되며 'docker ps'를 실행할 때 검사할 수 있습니다.

```none
C:\> docker run -itd -p 80 windowsservercore cmd

# Network services running on port TCP:80 in this container can be accessed externally on port TCP:14824
C:\> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
bbf72109b1fc        windowsservercore   "cmd"               6 seconds ago       Up 2 seconds        *0.0.0.0:14824->80/tcp*   drunk_stonebraker

# Container image specified EXPOSE 80 in Dockerfile - publish this port mapping
C:\> docker network
```
> WS2016 TP5 및 Windows 참가자 빌드 14300 이상부터 모든 NAT 포트 매핑에 대해 방화벽 규칙이 자동으로 만들어집니다. 이 방화벽 규칙은 컨테이너 호스트 전체에 적용되며 특정 컨테이너 끝점이나 네트워크 어댑터로 지역화되지 않습니다.

Windows NAT(WinNAT) 구현에는 [WinNAT capabilities and limitations](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/)(WinNAT 기능 및 제한 사항) 블로그에서 설명하는 몇 가지 기능 차이가 있습니다.
 1. 컨테이너 호스트당 NAT 내부 IP 접두사 하나만 지원되므로 ‘여러’ NAT 네트워크는 접두사를 분할하여 정의해야 합니다(이 문서의 “여러 NAT 네트워크” 참조).
 2. 컨테이너 끝점은 컨테이너 내부 IP 및 포트를 사용하여 컨테이너 호스트에서만 연결할 수 있습니다('docker 네트워크 검사 <CONTAINER ID>'를 사용하여 이 정보 검색).

다른 드라이버를 사용하여 추가 네트워크를 만들 수 있습니다.

> Docker 네트워크 드라이버는 모두 소문자입니다.

### 투명 네트워킹

투명 네트워킹 모드를 사용하려면 드라이버 이름 '투명'을 사용하여 컨테이너 네트워크를 만듭니다.

```none
C:\> docker network create -d transparent MyTransparentNetwork
```
> 참고: 투명 네트워크를 만드는 중 오류가 발생하는 경우 시스템에 외부 vSwitch가 있을 수 있습니다. 이는 Docker에서 자동으로 검색되지 않으므로 투명 네트워크가 컨테이너 호스트의 외부 네트워크 어댑터에 바인딩되지 못하도록 합니다. 자세한 내용은 ‘주의 사항 및 문제’ 아래의 ‘투명 네트워크 생성을 막는 기존 vSwitch’ 섹션을 참조하세요.

컨테이너 호스트가 가상화되는 경우 IP 할당에 DHCP를 사용하려면 가상 컴퓨터 네트워크 어댑터에서 MACAddressSpoofing을 사용하도록 설정해야 합니다. 그렇지 않으면 Hyper-V 호스트는 여러 MAC 주소를 사용하여 VM의 컨테이너에서 들어오는 네트워크 트래픽을 차단합니다.

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> 둘 이상의 투명 네트워크를 만들려는 경우 외부 Hyper-V 가상 스위치(자동으로 생성)가 바인딩되어야 하는 (가상) 네트워크 어댑터를 지정해야 합니다.

투명 네트워크에 연결된 컨테이너 끝점의 IP 주소는 외부 DHCP 서버에서 정적으로나 동적으로 할당할 수 있습니다.

고정 IP 할당을 사용하는 경우 먼저 네트워크를 만들 때 *--subnet* 및 *--gateway* 매개 변수를 지정해야 합니다. 서브넷 및 게이트웨이 IP 주소는 컨테이너 호스트 즉, 실제 네트워크에 대한 네트워크 설정과 동일해야 합니다.

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
`docker run` 명령에 *--ip* 옵션을 사용하여 IP 주소를 지정합니다.

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> 이 IP 주소는 실제 네트워크에 있는 다른 네트워크 장치에 할당되어 있지 않아야 합니다.

컨테이너 끝점은 실제 네트워크에 직접 액세스할 수 있으므로 포트 매핑을 지정할 필요가 없습니다.

### 오버레이 네트워크

*오버레이 네트워킹 모드를 사용하려면 Swarm 모드에서 관리자 노드로 실행 중인 Docker 호스트를 사용해야 합니다.* Swarm 모드와 Swarm 관리자 초기화 방법에 대한 자세한 내용은 [Swarm 모드 시작](./swarm-mode.md) 항목을 참조하세요.

오버레이 네트워크를 만들려면 **Swarm 관리자 모드**에서 다음 명령을 실행합니다.

```none
# Create an overlay network from a swarm manager node, called "myOverlayNet"
C:\> docker network create --driver=overlay myOverlayNet
```

### L2 브리지

L2 브리지 네트워킹 모드를 사용하려면 드라이버 이름 'l2bridge'를 사용하여 컨테이너 네트워크를 만듭니다. 마찬가지로 실제 네트워크에 해당되는 서브넷 및 게이트웨이를 지정해야 합니다.

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

l2bridge 네트워크에서는 고정 IP 할당만 지원됩니다.

> SDN 패브릭에서 l2bridge 네트워크를 사용할 경우에는 동적 IP 할당만 지원됩니다. 자세한 내용은 [가상 네트워크에 컨테이너 연결](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network) 항목을 참조하세요.

## 기타 작업 및 구성

> Microsoft는 Windows의 Docker 개선 작업을 지속적으로 하고 있습니다. 모든 최신 기능에 액세스할 수 있는지 확인하려면 최신 버전의 Docker 엔진을 사용하고 있는지 확인하세요. `docker -v`를 사용하여 Docker 버전을 확인할 수 있습니다. Docker 구성에 대한 지침은 [Windows의 Docker 엔진](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon) 항목을 참조하십시오.

### 사용 가능한 네트워크 나열

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### 네트워크 제거

`docker network rm`을 사용하여 컨테이너 네트워크를 삭제합니다.

```none
C:\> docker network rm <network name>
```

그러면 컨테이너 네트워크에서 사용한 Hyper-V 가상 스위치를 정리하고 만들어진 모든 Network Address Translation(WinNAT - NetNat 인스턴스)도 정리합니다.

### 네트워크 검사

특정 네트워크에 연결된 컨테이너와 이러한 컨테이너 끝점과 연결된 IP를 확인하려면 다음을 실행할 수 있습니다.

```none
C:\> docker network inspect <network name>
```

### HNS 서비스에 네트워크의 이름 지정

**일반적으로 `docker network create`를 사용하여 컨테이너 네트워크를 만들면, 제공하는 네트워크 이름이 Docker 서비스에 사용되지만 HNS 서비스에는 사용되지 않습니다.**

네트워크를 만들 경우 `docker network create` 명령에 `-o com.docker.network.windowsshim.networkname=<network name>` 옵션을 사용하여 HNS 서비스에서 이름을 제공하도록 지정할 수 있습니다. 예를 들어, 다음 명령을 사용하여 HNS 서비스에 지정된 이름으로 투명 네트워크를 만들 수 있습니다.

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

#### 예제: 기본 HNS 명명 방식

이 명명 옵션의 작동 방식을 설명하기 위해 아래 화면 캡처에서는 이 명명 옵션을 사용하지 *않을* 경우 HNS 서비스에서 어떻게 네트워크의 이름을 지정하는지를 보여 줍니다. 이 예제에서는 `docker network ls` 명령을 사용할 때와 같이 네트워크의 이름인 "MyTransparentNetwork"가 Docker에 표시됩니다. 그러나 HNS 서비스에는 `Get-ContainerNetwork` Windows PowerShell 명령을 사용할 때와 같이 네트워크의 이름이 표시되지 않고, 대신 네트워크에 대한 긴 영숫자 이름이 HNS에 의해 자동으로 생성됩니다.

><figure>
  <img src="media/SpecifyName_Capture.PNG">
  <figcaption>예제: 네트워크의 이름이 HNS 서비스에 지정되지 <i>않았습니다</i>. </figcaption>
</figure>

#### 예제: HNS 서비스에 네트워크의 이름 지정

반면 `-o com.docker.network.windowsshim.networkname=<network name>`을 *사용하면* 생성된 이름 대신 지정한 이름이 HNS 서비스에 사용됩니다. 이 작동 방식이 아래 화면 캡처에 나와 있습니다.

><figure>
  <img src="media/SpecifyName_Capture_2.PNG">
  <figcaption>예제: `-o com.docker.network.windowsshim.networkname=<network name>` 옵션을 사용하여 HNS 서비스에 네트워크의 이름이 지정됩니다.</figcaption>
</figure>


### 특정 네트워크 인터페이스에 네트워크 바인딩

Hyper-V 가상 스위치를 통해 연결된 네트워크를 특정 네트워크 인터페이스에 바인딩하려면 `docker network create` 명령에 `-o com.docker.network.windowsshim.interface=<Interface>` 옵션을 사용합니다. 예를 들어, 다음 명령을 사용하여 "Ethernet 2" 네트워크 인터페이스에 연결된 투명 네트워크를 만들 수 있습니다.

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 참고: *com.docker.network.windowsshim.interface*의 값은 다음 명령으로 찾을 수 있는 네트워크 어댑터의 *이름*입니다.

>```none
PS C:\> Get-NetAdapter
```

### Set the VLAN ID for a Network

To set a VLAN ID for a network, use the option, `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` to the `docker network create` command. For instance, you might use the following command to create a transparent network with a VLAN ID of 11:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
네트워크의 VLAN ID를 설정하면 해당 네트워크에 연결될 모든 컨테이너 끝점에 대해 VLAN 격리가 설정됩니다.

**참고:** vSwitch가 올바른 VLAN에서 액세스 모드로 vNIC(컨테이너 끝점)를 사용하여 태그가 지정된 트래픽을 모두 처리할 수 있도록 호스트 네트워크 어댑터(실제)가 트렁크 모드에 있는지 확인합니다.


### 네트워크의 DNS 접미사 및/또는 DNS 서버 지정

`-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` 옵션을 사용하여 네트워크의 DNS 접미사를 지정하고, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` 옵션을 사용하여 네트워크의 DNS 서버를 지정합니다. 예를 들어, 다음 명령을 사용하여 네트워크의 DNS 접미사를 "example.com"으로 설정하고 네트워크의 DNS 서버를 4.4.4.4 및 8.8.8.8로 설정합니다.

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

### 여러 컨테이너 네트워크
여러 컨테이너 네트워크를 단일 컨테이너 호스트에서 만들 수 있으며, 다음 사항에 주의해야 합니다.

* 연결에 외부 vSwitch를 사용하는 여러 네트워크(예: 투명, L2 브리지, L2 투명)는 각각 고유한 네트워크 어댑터를 사용해야 합니다.
* 현재 단일 컨테이너 호스트에서 여러 NAT 네트워크를 만들기 위한 솔루션은 기존 NAT 네트워크의 내부 접두사를 분할합니다. 이에 대한 자세한 지침은 아래의 '여러 NAT 네트워크' 섹션을 참조하세요.

### 여러 NAT 네트워크
호스트의 NAT 네트워크 내부 접두사를 분할하여 단일 컨테이너 호스트에 여러 NAT 네트워크를 정의할 수 있습니다.

새 NAT 네트워크에 대한 분할은 더 큰 내부 NAT 네트워킹 접두사 아래에 만들어야 합니다. PowerShell에서 다음 명령을 실행하고 "InternalIPInterfaceAddressPrefix" 필드를 참조하여 접두사를 찾을 수 있습니다.

```none
PS C:\> Get-NetNAT
```

예를 들어 호스트의 NAT 네트워크 내부 접두사는 172.16.0.0/12일 수 있습니다. 이 경우 Docker를 사용하여 *172.16.0.0/12 접두사에 해당하는 경우* 추가 NAT 네트워크를 만들 수 있습니다. 예를 들어 IP 접두사 172.16.0.0/16(게이트웨이, 172.16.0.1) 및 172.17.0.0/16(게이트웨이, 172.17.0.1)을 사용하여 NAT 네트워크를 두 개 만들 수 있습니다.

```none
C:\> docker network create -d nat --subnet=172.16.0.0/16 --gateway=172.16.0.1 CustomNat1
C:\> docker network create -d nat --subnet=172.17.0.0/16 --gateway=172.17.0.1 CustomNat2
```

새로 만든 네트워크는 다음을 사용하여 나열할 수 있습니다.
```none
C:\> docker network ls
```


### 네트워크 선택

Windows 컨테이너를 만들 때 컨테이너 네트워크 어댑터가 연결될 네트워크를 지정할 수 있습니다. 네트워크를 지정하지 않을 경우 기본 NAT 네트워크가 사용됩니다.

컨테이너를 기본이 아닌 NAT 네트워크에 연결하려는 경우 Docker run 명령에서 --network 옵션을 사용합니다.

```none
C:\> docker run -it --network=MyTransparentNet windowsservercore cmd
```

### 고정 IP 주소입니다.

```none
C:\> docker run -it --network=MyTransparentNet --ip=10.80.123.32 windowsservercore cmd
```

고정 IP 할당은 컨테이너의 네트워크 어댑터에서 직접 수행되며, 컨테이너가 중지됨 상태인 경우에만 수행해야 합니다. 컨테이너가 실행되는 동안에는 컨테이너 네트워크 어댑터의 핫 애드나 네트워크 스택 변경이 지원되지 않습니다.

## Docker Compose 및 Service Discovery

> 여러 서비스의 확장 응용 프로그램을 정의하는 데 Docker Compose 및 Service Discovery를 사용하는 방법에 대한 실제 예는 [Virtualization Blog](https://blogs.technet.microsoft.com/virtualization/)(가상화 블로그)에서 [이 게시물](https://blogs.technet.microsoft.com/virtualization/2016/10/18/use-docker-compose-and-service-discovery-on-windows-to-scale-out-your-multi-service-container-application/)을 참조하세요.

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/)는 해당 네트워크를 사용할 컨테이너/서비스와 함께 컨테이너 네트워크를 정의하고 구성하는 데 사용할 수 있습니다. Compose '네트워크' 키는 컨테이너를 연결할 네트워크 정의에서 최상위 키로 사용됩니다. 예를 들어 아래 구문에서는 지정한 Compose 파일에 정의된 모든 컨테이너/서비스에 대해 ‘기본’ 네트워크가 되도록 Docker에서 만든 기존 NAT 네트워크를 정의합니다.

```none
networks:
 default:
  external:
   name: "nat"
```

마찬가지로, 사용자 지정 NAT 네트워크를 정의하는 데 다음 구문을 사용할 수 있습니다.

> 참고: 아래 예에 정의된 '사용자 지정 NAT 네트워크'는 컨테이너 호스트의 기존 NAT 내부 접두사의 파티션으로 정의됩니다. 자세한 컨텍스트는 위의 '여러 NAT 네트워크' 섹션을 참조하세요.

```none
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.17.0.0/16
```

Docker Compose를 사용하여 컨테이너 네트워크 정의/구성에 대한 자세한 내용은 [Compose File reference](https://docs.docker.com/compose/compose-file/)(Compose 파일 참조)를 참조하세요.

### 서비스 검색
Service Discovery는 Docker에 기본 제공되며, 컨테이너 및 서비스의 IP(DNS) 매핑에 대한 서비스 등록 및 이름을 처리합니다. 서비스 검색을 통해 모든 컨테이너 끝점에서 이름(컨테이너 이름 또는 서비스 이름)으로 서로를 검색할 수 있습니다. 이는 특히 단일 서비스를 정의하는 데 여러 컨테이너 끝점이 사용되는 확장 시나리오에서 유용합니다. 이러한 경우 서비스 검색을 통해 백그라운드에서 실행되는 컨테이너 수에 관계없이, 서비스를 단일 엔터티로 간주하도록 할 수 있습니다. 다중 컨테이너 서비스의 경우, 들어오는 네트워크 트래픽이 라운드 로빈 방식을 사용하여 관리됩니다. 라운드 로빈 방식은 지정된 서비스를 구현하는 모든 컨테이너 인스턴스에서 트래픽이 균일하게 분산되도록 DNS 부하 분산을 사용합니다.

## 오버레이 네트워킹 및 Docker Swarm 모드(다중 노드 컨테이너 네트워킹)
네이티브 오버레이 네트워크 드라이버와 Docker Swarm 모드가 결합하여 Windows에서 다중 노드(클러스터링) 시나리오를 지원합니다. 오버레이와 Swarm 모드에 대한 자세한 내용은 Windows 10의 Windows 참가자에게 오버레이/Swarm 릴리스와 함께 제공된 [블로그 게시물](https://blogs.technet.microsoft.com/virtualization/2017/02/09/overlay-network-driver-with-support-for-docker-swarm-mode-now-available-to-windows-insiders-on-windows-10/)이나 [Swarm 모드 시작](./swarm-mode.md) 항목을 참조하세요.

## 주의 사항 및 문제

### 투명 네트워크 생성을 막는 기존 vSwitch

투명 네트워크를 만들 때 Docker는 네트워크에 대한 외부 vSwitch를 만든 다음 스위치를 (외부) 네트워크 어댑터에 바인딩하려고 합니다. 어댑터는 VM 네트워크 어댑터 또는 실제 네트워크 어댑터일 수 있습니다. vSwitch가 컨테이너 호스트에 이미 생성되고 *Docker에 표시되는 경우 *Windows Docker 엔진은 새 스위치를 만드는 대신 해당 스위치를 사용합니다. 그러나 vSwitch가 대역 외에서 생성되고(HYper-V 관리자 또는 PowerShell을 사용하여 컨테이너 호스트에서 생성) 아직 Docker에 표시되지 않는 경우, Windows Docker 엔진은 새 vSwitch를 만들려고 시도한 다음 새 스위치를 컨테이너 호스트 외부 네트워크 어댑터에 연결할 수 없습니다(네트워크 어댑터가 이미 대역 외에서 생성된 스위치에 연결되어 있기 때문).

예를 들어 Docker 서비스가 실행 중일 때 호스트에 새 vSwitch를 먼저 만든 다음 투명 네트워크를 만들려고 하면 이 문제가 발생합니다. 이 경우 Docker에서는 사용자가 만든 스위치를 인식할 수 없으며 투명 네트워크에 대한 새 vSwitch를 만듭니다.

이 문제를 해결하는 데는 다음과 같은 세 가지 방법이 있습니다.

* 물론 대역 외에서 만든 vSwitch를 삭제할 수 있습니다. 그러면 Docker에서 문제 없이 새 vSwitch를 만들고 호스트 네트워크 어댑터에 연결할 수 있습니다. 이 방법을 선택하기 전에 대역 외 vSwitch를 다른 서비스(예: Hyper-V)에서 사용하고 있지 않은지 확인하세요.
* 또는 대역 외에서 생성된 외부 vSwitch를 사용하려는 경우 Docker 및 HNS 서비스를 다시 시작하여 *Docker에 스위치가 표시되도록 하세요.*
```none
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* 또 다른 방법으로는 '-o com.docker.network.windowsshim.interface' 옵션을 사용하여 투명 네트워크의 외부 vSwitch를 컨테이너 호스트에서 아직 사용하지 않는 특정 네트워크 어댑터(대역 외에서 생성된 vSwitch에서 사용하는 것 외의 네트워크 어댑터)에 바인딩하는 것입니다. '-o' 옵션은 이 문서의 [투명 네트워크](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network) 섹션에서 자세히 설명됩니다.

### 지원되지 않는 기능

현재 다음과 같은 네트워킹 기능은 Docker CLI를 통해 지원되지 않습니다.
 * 컨테이너 연결(예: --link)

지금 다음과 같은 네트워크 옵션은 Windows Docker에서 지원되지 않습니다.
 * --add-host
 * --dns-opt
 * --dns-search
 * --aux-address
 * --internal
 * --ip-range

