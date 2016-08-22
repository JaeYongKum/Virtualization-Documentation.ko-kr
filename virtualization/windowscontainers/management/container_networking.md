---
title: "Windows 컨테이너 네트워킹"
description: "Windows 컨테이너에 대한 네트워킹을 구성합니다."
keywords: "Docker, 컨테이너"
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: c412171773e9c66569eab2252b5adfc187eedafd
ms.openlocfilehash: eb7d2c25d929cb51abfad17c26a89105f6574a48

---

# 컨테이너 네트워킹

Windows 컨테이너는 네트워킹에 관해서 가상 컴퓨터와 유사하게 작동합니다. 각 컨테이너에는 가상 네트워크 어댑터가 있고 이것은 가상 스위치에 연결되며 이를 통해 인바운드 및 아웃바운드 트래픽이 전달됩니다. 동일한 호스트에서 컨테이너 간에 격리를 적용하려면 컨테이너에 대한 네트워크 어댑터가 설치되는 각 Windows Server 및 Hyper-V 컨테이너에 대해 네트워크 구획을 만듭니다. Windows Server 컨테이너는 호스트 vNIC를 사용하여 가상 스위치에 연결합니다. Hyper-V 컨테이너는 가상 VM NIC(유틸리티 VM에 노출되지 않음)를 사용하여 가상 스위치에 연결합니다.

Windows 컨테이너는 *nat*, *transparent*, *l2bridge* 및 *l2tunnel*의 네 가지 다른 네트워킹 드라이버 또는 모드를 지원합니다. 실제 네트워크 인프라와 단일 및 다중 호스트 네트워킹 요구 사항에 따라 요구에 가장 적합한 네트워크 모드를 선택해야 합니다. 

Docker 엔진은 docker 서비스가 처음으로 실행될 때 기본적으로 nat 네트워크를 만듭니다. 만들어지는 기본 내부 IP 접두사는 172.16.0.0/12입니다. 

> 참고: 컨테이너 호스트 IP가 이와 동일한 접두사에 있는 경우 아래 설명된 대로 NAT 내부 IP 접두사를 변경해야 합니다.

컨테이너 끝점은 이 기본 네트워크에 연결되고 내부 접두사에서 IP 주소가 할당됩니다. 현재 Windows에서는 하나의 NAT 네트워크만 지원됩니다(보류 중인 [Pull Request](https://github.com/docker/docker/pull/25097)이 제한을 해결할 수는 있음). 

동일한 컨테이너 호스트에서 다른 드라이버(예: transparent, l2bridge)를 사용하여 추가 네트워크를 만들 수 있습니다. 아래 표에서는 각 모드의 내부(컨테이너 간) 및 외부 연결에 대한 네트워크 연결을 제공하는 방법을 보여 줍니다.

- **Network Address Translation** – 각 컨테이너가 내부 개인 IP 접두사(예: 172.16.0.0/12)에서 IP 주소를 수신합니다. 컨테이너 호스트와 컨테이너 끝점 간 포트 전달/매핑이 지원됩니다.

- **Transparent**– 각 컨테이너 끝점이 실제 네트워크에 직접 연결됩니다. 실제 네트워크의 IP는 외부 DHCP 서버를 사용하여 정적으로나 동적으로 할당할 수 있습니다.

- **L2 브리지** - 각 컨테이너 끝점이 컨테이너 호스트와 동일한 IP 서브넷에 있습니다. IP 주소는 컨테이너 호스트와 동일한 접두사에서 정적으로 할당해야 합니다. 호스트의 모든 컨테이너 끝점은 계층 2 주소 변환으로 인해 MAC 주소가 동일합니다.

- **L2 터널** - 이 모드는 Microsoft 클라우드 스택에서만 사용되어야 합니다.

> Microsoft SDN 스택을 사용하여 컨테이너 끝점을 오버레이 가상 네트워크에 연결하는 방법을 알아보려면 [가상 네트워크에 컨테이너 연결](location) 항목을 참조하세요.

## 단일 노드

|  | 컨테이너-컨테이너 | 컨테이너-외부 |
| :---: | :---------------     |  :---                |
| nat | Hyper-V 가상 스위치를 통한 브리지된 연결 | 주소 변환이 적용된 WinNAT를 통해 라우트됨 | 
| transparent | Hyper-V 가상 스위치를 통한 브리지된 연결 | 실제 네트워크에 직접 액세스 | 
| l2bridge | Hyper-V 가상 스위치를 통한 브리지된 연결|  MAC 주소 변환을 사용하여 실제 네트워크에 액세스|  



## 다중 노드

|  | 컨테이너-컨테이너 | 컨테이너-외부 |
| :---: | :----       | :---------- |
| nat | 외부 컨테이너 호스트 IP 및 포트를 참조해야 함. 주소 변환이 적용된 WinNAT를 통해 라우트됨 | 외부 컨테이너 호스트를 참조해야 함. 주소 변환이 적용된 WinNAT를 통해 라우트됨 | 
| transparent | 컨테이너 IP 끝점을 직접 참조해야 함 | 실제 네트워크에 직접 액세스 | 
| l2bridge | 컨테이너 IP 끝점을 직접 참조해야 함| MAC 주소 변환을 사용하여 실제 네트워크에 액세스| 


## 네트워크 만들기 

### (기본값) NAT 네트워크

Windows docker 엔진는 IP 접두사 172.16.0.0/12를 사용하여 기본 'nat' 네트워크를 만듭니다. 현재 Windows 컨테이너 호스트에서는 nat 네트워크를 하나만 허용합니다. 사용자가 특정 IP 접두사를 사용하여 nat 네트워크를 만들려는 경우 C:\ProgramData\Docker\config\daemon.json에 있는 docker 구성 daemon.json 파일(없는 경우 만듦)에서 옵션을 변경하여 두 가지 작업 중 하나를 수행할 수 있습니다.
 1. 지정된 IP 접두사 및 일치를 사용하여 기본 nat 네트워크를 만드는 _"fixed-cidr": "< IP Prefix > / Mask"_ 옵션 사용
 2. 기본 네트워크를 만들지 않는 _"bridge": "none"_ 옵션 사용. 사용자는 *docker network create -d <driver>* 명령을 사용하여 원하는 드라이버로 사용자 정의 네트워크를 만들 수 있습니다.

이러한 구성 옵션 중 하나를 수행하기 전에 Docker 서비스를 먼저 중지해야 하고 기존 nat 네트워크를 모두 삭제해야 합니다.

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

사용자가 daemon.json 파일에 "fixed-cidr" 옵션을 추가한 경우 docker 엔진은 이제 지정된 사용자 지정 IP 접두사 및 마스크를 사용하여 사용자 정의 nat 네트워크를 만듭니다. 그렇지 않고 사용자가 "bridge:none" 옵션을 추가한 경우 네트워크를 수동으로 만들어야 합니다.

```none
# Create a user-defined nat network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

기본적으로 컨테이너 끝점은 기본 nat 네트워크에 연결됩니다. daemon.json에 "bridge:none"을 지정하여 기본 nat 네트워크가 만들어지지 않았거나 다른 사용자 정의 네트워크에 액세스해야 하는 경우 사용자는 docker run 명령에 *--network* 매개 변수를 지정할 수 있습니다.

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### 포트 매핑

NAT 네트워크에 연결된 컨테이너 내부에서 실행되는 응용 프로그램에 액세스하려면 컨테이너 호스트와 컨테이너 끝점 사이에 포트 매핑을 만들어야 합니다. 이러한 매핑은 컨테이너 생성 시나 컨테이너가 중지된 상태일 때 지정해야 합니다.

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Create a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

또는 -p 매개 변수를 사용하거나 Dockerfile에 EXPOSE 명령과 함께 -P 매개 변수를 사용하여 동적 포트 매핑을 지정할 수도 있습니다. 임의로 선택한 사용 후 삭제 포트는 컨테이너 호스트에서 선택되며 Docker ps를 실행할 때 검사할 수 있습니다.

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
 1. 컨테이너 호스트당 하나의 NAT 네트워크(하나의 내부 IP 접두사)만 지원됩니다.
 2. 컨테이너 끝점은 컨테이너 호스트에서 내부 IP 및 포트를 사용해서만 연결할 수 있습니다.

다른 드라이버를 사용하여 추가 네트워크를 만들 수 있습니다. 

> Docker 네트워크 드라이버는 모두 소문자입니다.

### 투명 네트워킹

투명 네트워킹 모드를 사용하려면 드라이버 이름 '투명'을 사용하여 컨테이너 네트워크를 만듭니다. 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```

컨테이너 호스트가 가상화되는 경우 IP 할당에 DHCP를 사용하려면 가상 컴퓨터 네트워크 어댑터에서 MACAddressSpoofing을 사용하도록 설정해야 합니다. 그렇지 않으면 Hyper-V 호스트는 여러 MAC 주소를 사용하여 VM의 컨테이너에서 들어오는 네트워크 트래픽을 차단합니다.

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> 둘 이상의 투명 네트워크를 만들려는 경우 외부 Hyper-V 가상 스위치(자동으로 생성)가 바인딩되어야 하는 (가상) 네트워크 어댑터를 지정해야 합니다.

Hyper-V 가상 스위치를 통해 연결된 네트워크를 특정 네트워크 인터페이스에 바인딩하려면 *-o com.docker.network.windowsshim.interface=<Interface>* 옵션을 사용합니다.

```none
# Create a transparent network which is attached to the "Ethernet 2" network interface
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

*com.docker.network.windowsshim.interface*의 값은 다음에서 어댑터의 *이름*입니다. 
```none
Get-NetAdapter
```

투명 네트워크에 연결된 컨테이너 끝점의 IP 주소는 외부 DHCP 서버에서 정적으로나 동적으로 할당할 수 있습니다.

고정 IP 할당을 사용하는 경우 먼저 네트워크를 만들 때 *--subnet* 및 *--gateway* 매개 변수를 지정해야 합니다. 서브넷 및 게이트웨이 IP 주소는 컨테이너 호스트 즉, 실제 네트워크에 대한 네트워크 설정과 동일해야 합니다.

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
docker run 명령에 *--ip* 옵션을 사용하여 IP 주소를 지정합니다.

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> 이 IP 주소는 실제 네트워크에 있는 다른 네트워크 장치에 할당되어 있지 않아야 합니다.

컨테이너 끝점은 실제 네트워크에 직접 액세스할 수 있으므로 포트 매핑을 지정할 필요가 없습니다.

### L2 브리지 

L2 브리지 네트워킹 모드를 사용하려면 드라이버 이름 'l2bridge'를 사용하여 컨테이너 네트워크를 만듭니다. 마찬가지로 실제 네트워크에 해당되는 서브넷 및 게이트웨이를 지정해야 합니다.

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

l2bridge 네트워크에서는 고정 IP 할당만 지원됩니다. 

> SDN 패브릭에서 l2bridge 네트워크를 사용할 경우에는 동적 IP 할당만 지원됩니다. 자세한 내용은 [가상 네트워크에 컨테이너 연결](location) 항목을 참조하세요.

## 기타 작업 및 구성

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
C:\> docker network rm "<network name>"
```

그러면 컨테이너 네트워크에서 사용한 Hyper-V 가상 스위치를 정리하고 만들어진 모든 Network Address Translation(WinNAT - NetNat 인스턴스)도 정리합니다.

### 네트워크 검사 

특정 네트워크에 연결된 컨테이너와 이러한 컨테이너 끝점과 연결된 IP를 확인하려면 다음을 실행할 수 있습니다.

```none
C:\> docker network inspect <network name>
```

### 여러 컨테이너 네트워크

여러 컨테이너 네트워크를 단일 컨테이너 호스트에서 만들 수 있으며, 다음 사항에 주의해야 합니다.
* 컨테이너 호스트당 하나의 NAT 네트워크만 만들 수 있습니다.
* 연결에 외부 vSwitch를 사용하는 여러 네트워크(예: 투명, L2 브리지, L2 투명)는 각각 고유한 네트워크 어댑터를 사용해야 합니다.

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


## 주의 사항 및 문제

### 방화벽

컨테이너 호스트에서 ICMP(Ping) 및 DHCP를 사용하도록 설정하려면 특정 방화벽 규칙을 만들어야 합니다. ICMP 및 DHCP는 Windows Server 컨테이너에서 동일한 호스트의 두 컨테이너 사이를 ping하고 DHCP를 통해 동적으로 할당된 IP 주소를 받는 데 필요합니다. TP5에서는 이러한 규칙이 Install-ContainerHost.ps1 스크립트를 통해 만들어지고, Post-TP5에서는 이러한 규칙이 자동으로 만들어집니다. NAT 포트 전달 규칙에 해당하는 모든 방화벽 규칙은 자동으로 만들어지고 컨테이너가 중지되면 정리됩니다.

### 지원되지 않는 기능

다음과 같은 네트워킹 기능은 Docker CLI를 통해 현재 지원되지 않습니다.
 * 컨테이너 연결(예: --link)
 * 컨테이너에 대한 이름/서비스 기반 IP 확인. _서비스 업데이트에서 곧 지원될 예정입니다._
 * 기본 오버레이 네트워크 드라이버

지금은 다음과 같은 네트워크 옵션이 Windows Docker에서 지원되지 않습니다.
 * --add-host
 * --dns
 * --dns-opt
 * --dns-search
 * -h, --hostname
 * --net-alias
 * --aux-address
 * --internal
 * --ip-range

 > Windows Server 2016 Technical Preview 5 및 최신 WIP(Windows Insider Preview) "플라이트" 빌드에는 새 빌드로 업그레이드할 때 중복된(예: "누수") 컨테이너 네트워크 및 vSwitch가 생성되는 알려진 버그가 있습니다. 이 문제를 해결하려면 다음 스크립트를 실행하세요.
```none
$KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
$keys = get-childitem $KeyPath
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
remove-netnat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # Note: failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```
> 호스트를 다시 부팅 후 나머지 단계를 실행합니다.
```none
Get-NetNat | Remove-NetNat -Confirm $false
Set-Service docker -StartupType automatic
Start-Service docker 
```



<!--HONumber=Aug16_HO3-->


