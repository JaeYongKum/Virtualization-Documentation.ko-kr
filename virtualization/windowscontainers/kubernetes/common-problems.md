---
title: Kubernetes 문제 해결
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Kubernetes를 배포 하 고 Windows 노드를 조인할 때 발생 하는 일반적인 문제에 대 한 솔루션입니다.
keywords: kubernetes, 1.14, linux, 컴파일
ms.openlocfilehash: dfb9be5bb5a5dd3507ee7266346634579df503c0
ms.sourcegitcommit: 7f3d98da46c73e428565268683691f383c72221f
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/05/2020
ms.locfileid: "84461611"
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes 문제 해결 #
이 페이지에서는 Kubernetes 설정, 네트워킹 및 배포와 관련 된 몇 가지 일반적인 문제를 안내 합니다.

> [!tip]
> PR을 [설명서 리포지토리로](https://github.com/MicrosoftDocs/Virtualization-Documentation/)올려서 FAQ 항목을 제안 합니다.

이 페이지는 다음과 같은 범주로 나뉩니다.
1. [일반적인 질문](#general-questions)
2. [일반적인 네트워킹 오류](#common-networking-errors)
3. [일반적인 Windows 오류](#common-windows-errors)
4. [일반적인 Kubernetes 마스터 오류](#common-kubernetes-master-errors)

## <a name="general-questions"></a>일반적인 질문 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Windows에서 시작. p s 1이 성공적으로 완료 어떻게 할까요?? ###
Kubelet, kube 및 (Flannel를 네트워킹 솔루션으로 선택한 경우)에서 실행 중인 로그가 별도의 PoSh 창에 표시 되는 것을 확인 하 고 노드에서 실행 되는 호스트 에이전트 프로세스를 확인 해야 합니다. 이 외에도 Windows 노드가 Kubernetes 클러스터에 "Ready"로 표시 되어야 합니다.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>PoSh windows 대신 백그라운드에서이 모든 작업을 실행 하도록를 구성할 수 있나요? ###
Kubernetes 버전 1.11부터 kubelet & kube를 네이티브 [Windows 서비스로](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)실행할 수 있습니다. 항상 [nssm.exe](https://nssm.cc/) 와 같은 대체 서비스 관리자를 사용 하 여 이러한 프로세스 (flanneld, kubelet & kube)를 백그라운드에서 항상 실행할 수 있습니다. 예제 단계는 [Kubernetes의 Windows 서비스](./kube-windows-services.md) 를 참조 하세요.

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Windows 서비스로 Kubernetes 프로세스를 실행 하는 데 문제가 있음 ###
초기 문제 해결을 위해 [nssm.exe](https://nssm.cc/) 에서 다음 플래그를 사용 하 여 stdout 및 stderr을 출력 파일로 리디렉션할 수 있습니다.
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
자세한 내용은 공식 [nssm.exe 사용](https://nssm.cc/usage) 문서를 참조 하세요.

## <a name="common-networking-errors"></a>일반적인 네트워킹 오류 ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>부하 분산 장치가 클러스터 노드에 일관 되 게 배관 됨 ###
Windows에서 kube는 클러스터의 모든 Kubernetes 서비스에 대 한 HNS 부하 분산 장치를 만듭니다. (기본값) kube 구성에서 많은 (일반적으로 100 개 이상)의 부하 분산 장치를 포함 하는 클러스터의 노드는 사용 가능한 임시 TCP 포트 ( 동적 포트 범위-기본적으로 49152 ~ 65535 포트를 포함 합니다. 이는 DSR이 아닌 모든 부하 분산 장치에 대해 각 노드에 예약 된 포트 수가 많기 때문입니다. 이 문제는 다음과 같은 kube에서 오류가 발생할 수 있습니다.
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

사용자는 [CollectLogs](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) 스크립트를 실행 하 고 파일을 컨설팅 하 여이 문제를 식별할 수 있습니다. `*portrange.txt`

는 `CollectLogs.ps1` 또한는 HNS 할당 논리를 모방 하 여 임시 TCP 포트 범위에서 포트 풀 할당 가용성을 테스트 하 고에서 성공/실패를 보고 `reservedports.txt` 합니다. 이 스크립트는 10 개의 64 TCP 임시 포트 (HNS 동작 에뮬레이트)를 예약 하 고, 예약 성공 & 오류 수를 계산 하 고, 할당 된 포트 범위를 해제 합니다. 성공 숫자가 10 보다 작은 경우에는 임시 풀에 사용 가능한 공간이 부족 한 것을 나타냅니다. Heuristical에 대 한 64-블록 포트 예약의 수를 요약 하 여에서 생성할 수도 `reservedports.txt` 있습니다.

이 문제를 해결 하려면 몇 가지 단계를 수행 하면 됩니다.
1.  영구 솔루션의 경우 kube 부하 분산을 [DSR 모드로](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710)설정 해야 합니다. DSR 모드는 완전히 구현 되며 새로운 [Windows Server Insider build 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) 이상 에서만 사용할 수 있습니다.
2. 해결 방법으로 사용자는와 같은 명령을 사용 하 여 사용 가능한 임시 포트의 기본 Windows 구성을 늘릴 수도 있습니다 `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>` . *경고:* 기본 동적 포트 범위를 재정의 하면 임시 범위에서 사용 가능한 TCP 포트를 사용 하는 호스트의 다른 프로세스/서비스에 영향을 미칠 수 있으므로이 범위는 신중 하 게 선택 해야 합니다.
3. Q 2020의 누적 업데이트를 통해 출시 되도록 예약 된 인텔리전트 포트 풀 공유를 사용 하는 비-DSR 모드 부하 분산 장치에 대 한 확장성이 향상 되었습니다.

### <a name="hostport-publishing-is-not-working"></a>HostPort 게시가 작동 하지 않습니다. ###
HostPort 기능을 사용 하려면 CNI 플러그 인이 [v 0.8.6](https://github.com/containernetworking/plugins/releases/tag/v0.8.6) release 이상 인지 확인 하 고 cni 구성 파일에는 `portMappings` 다음과 같은 기능이 설정 되어 있는지 확인 하세요.
```
"capabilities": {
    "portMappings":  true
}
```

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>"HnsCall에 실패 했습니다. Win32에서 잘못 된 디스켓이 드라이브에 있습니다."와 같은 오류가 표시 됩니다. ###
이 오류는 사용자 지정 수정 작업 Windows 업데이트을 수행 하는 경우에 발생할 수 있습니다. 업데이트 전에 이전에 만든 HNS 개체가 현재 설치 된 HNS 버전과 호환 되지 않음을 나타냅니다.

Windows Server 2019 (및 아래)에서 사용자는 HNS 데이터 파일을 삭제 하 여 HNS 개체를 삭제할 수 있습니다. 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

사용자는 호환 되지 않는 모든 HNS 끝점 또는 네트워크를 직접 삭제할 수 있어야 합니다.
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Windows Server, 버전 1903의 사용자는 다음 레지스트리 위치로 이동 하 여 네트워크 이름 (예: `vxlan0` 또는)으로 시작 하는 모든 nic를 삭제할 수 있습니다. `cbr0`
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>Azure에서 내 Flannel 호스트-gw 배포의 컨테이너가 인터넷에 연결할 수 없음 ###
Azure에서 Flannel를 gw 모드로 배포할 때 패킷은 Azure 실제 호스트 vSwitch를 통해 이동 해야 합니다. 사용자는 노드에 할당 된 각 서브넷에 대해 "가상 어플라이언스" 유형의 [사용자 정의 경로](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined) 를 프로그래밍 해야 합니다. 이 작업은 Azure Portal ( [여기](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)예제 참조) 또는 Azure CLI를 통해 수행할 수 있습니다 `az` . 다음은 IP 10.0.0.4 및 해당 pod 서브넷 10.244.0.0/24가 있는 노드에 az commands를 사용 하는 이름이 "MyRoute" 인 UDR의 예입니다.
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> 다른 클라우드 공급자의 Azure 또는 IaaS Vm에 Kubernetes를 직접 배포 하는 경우 [오버레이 네트워킹](./network-topologies.md#flannel-in-vxlan-mode) 을 대신 사용할 수도 있습니다.

### <a name="my-windows-pods-cannot-ping-external-resources"></a>Windows pod 외부 리소스를 ping 할 수 없음 ###
Windows pod에는 현재 ICMP 프로토콜에 대해 프로그래밍 된 아웃 바운드 규칙이 없습니다. 그러나 TCP/UDP가 지원 됩니다. 클러스터 외부의 리소스에 대 한 연결을 시연 하려고 할 때 `ping <IP>` 해당 명령으로 대체 하세요 `curl <IP>` .

여전히 문제가 발생 하는 경우에는 일반적으로 [cni](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) 의 네트워크 구성으로 인해 몇 가지 주의가 필요 합니다. 언제 든 지이 정적 파일을 편집할 수 있으며, 구성은 새로 만든 Kubernetes 리소스에 적용 됩니다.

이유
Kubernetes 네트워킹 요구 사항 ( [Kubernetes 모델](https://kubernetes.io/docs/concepts/cluster-administration/networking/)참조) 중 하나는 내부적으로 NAT 없이 클러스터 통신을 수행 하는 것입니다. 이러한 요구 사항을 충족 하기 위해 아웃 바운드 NAT가 발생 하지 않도록 하는 모든 통신에 대 한 [예외](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) 를 발생 시킬 수 있습니다. 그러나이는 쿼리를 시도 하는 외부 IP를 제외 해야 한다는 의미 이기도 합니다. 그 다음에만 Windows pod에서 시작 된 트래픽이 외부 세계의 응답을 수신 하기 위해 올바르게 SNAT'ed 됩니다. 이와 관련 하 여의 예외는 다음과 같습니다 `cni.conf` .
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Windows 노드가 NodePort 서비스에 액세스할 수 없습니다. ###
Windows Server 2019에 대 한 디자인 제한으로 인해 노드 자체의 로컬 NodePort 액세스는 실패 합니다. NodePort 액세스는 다른 노드나 외부 클라이언트에서 작동 합니다.

### <a name="my-windows-node-stops-routing-thourgh-nodeports-after-i-scaled-down-my-pods"></a>내 Windows 노드가 pod을 확장 한 후 thourgh NodePorts 라우팅을 중지 합니다. ###
디자인 제한으로 인해 NodePort 전달이 작동 하려면 Windows 노드에서 하나 이상의 pod를 실행 해야 합니다.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>잠시 후 컨테이너의 vNICs 및 HNS 끝점을 삭제 하 고 있습니다. ###
`hostname-override`매개 변수가 [kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)에 전달 되지 않은 경우이 문제가 발생할 수 있습니다. 이 문제를 해결 하려면 다음과 같이 사용자가 호스트 이름을 kube에 전달 해야 합니다.
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>Flannel (vxlan) 모드에서 pod는 노드 다시 가입 하기 후 연결 문제가 있습니다. ###
이전에 삭제 한 노드가 클러스터에 다시 가입 될 때마다 flannelD는 노드에 새 pod 서브넷을 할당 합니다. 사용자는 다음 경로에서 이전 pod 서브넷 구성 파일을 제거 해야 합니다.
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Flanneld를 시작한 후에는 "네트워크를 만들 때까지 기다리는 중입니다." ###
조사 중인이 문제에 대 한 다양 한 보고서가 있습니다. flannel 네트워크의 관리 IP가 설정 된 경우에는 타이밍 문제가 있을 가능성이 높습니다. 해결 방법은 단순히 start. p s 1을 다시 시작 하거나 다음과 같이 수동으로 다시 시작 하는 것입니다.
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

현재 검토에서이 문제를 해결 하는 [PR](https://github.com/coreos/flannel/pull/1042) 도 있습니다.


### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>/Run/flannel/subnet.env가 없어서 Windows pod을 시작할 수 없습니다. ###
이는 Flannel이 올바르게 실행 되지 않았음을 나타냅니다. Flanneld를 다시 시작 하거나 `/run/flannel/subnet.env` Kubernetes 마스터의에서 수동으로 Windows 작업자 노드에 파일을 복사 하 `C:\run\flannel\subnet.env` 고 `FLANNEL_SUBNET` 할당 된 서브넷으로 행을 수정할 수 있습니다. 예를 들어 노드 서브넷 10.244.4.1/24가 할당 된 경우:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Flanneld에서이 파일을 자동으로 생성 하도록 하는 것이 더 안전 합니다.


### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>VSphere 실행 중인 Kubernetes 클러스터에서 호스트 간의 pod 간 연결이 끊어졌습니다. 
VSphere Flannel는 오버레이 네트워킹에 대해 포트 4789 (기본 VXLAN 포트)를 예약 하므로 패킷이 차단 될 수 있습니다. VSphere 오버레이 네트워킹에 사용 되는 경우 4789을 확보 하기 위해 다른 포트를 사용 하도록 구성 해야 합니다.  


### <a name="my-endpointsips-are-leaking"></a>내 끝점/i p s에서 누수가 발생 합니다. ###
현재 알려진 문제가 있어 끝점이 누출 될 수 있습니다. 
1.  첫 번째 [알려진 문제](https://github.com/kubernetes/kubernetes/issues/68511) 는 Kubernetes 버전 1.11의 문제입니다. Kubernetes 버전 1.11.0-1.11.2를 사용 하지 마세요.
2. 끝점을 누출 시킬 수 있는 두 번째 [알려진 문제](https://github.com/docker/libnetwork/issues/1950) 는 끝점 저장소의 동시성 문제입니다. 수정을 받으려면 Docker EE 18.09 이상을 사용 해야 합니다.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>"네트워크: 범위를 할당 하지 못했습니다." 오류 때문에 pod를 시작할 수 없습니다. ###
이는 노드의 IP 주소 공간이 사용 됨을 나타냅니다. [누수 된 끝점](#my-endpointsips-are-leaking)을 정리 하려면 영향을 받는 노드의 모든 리소스를 마이그레이션하고 다음 명령을 실행 & 합니다.
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Windows 노드에서 서비스 IP를 사용 하 여 내 서비스에 액세스할 수 없습니다. ###
이는 Windows의 현재 네트워킹 스택에 대 한 알려진 제한 사항입니다. 그러나 Windows *pod* **는** 서비스 IP에 액세스할 수 있습니다.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Kubelet를 시작할 때 네트워크 어댑터를 찾을 수 없습니다. ###
Windows 네트워킹 스택은 Kubernetes 네트워킹이 작동 하려면 가상 어댑터가 필요 합니다. 다음 명령에서 결과를 반환 하지 않는 경우 (관리자 셸에서) &mdash; Kubelet to work에 필요한 필수 구성 요소를 만듭니다 &mdash; .

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

호스트의 네트워크 어댑터가 "이더넷"이 아닌 경우에는 [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) 스크립트의 매개 변수를 수정 하는 것이 유용한 경우가 많습니다. 그렇지 않으면 스크립트의 출력을 참조 `start-kubelet.ps1` 하 여 가상 네트워크를 만드는 동안 오류가 발생 하는지 확인 합니다. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>일정 시간 후에 DNS 쿼리 확인이 성공적으로 중지 pod ###
Windows Server의 네트워킹 stack, 버전 1803의 알려진 DNS 캐싱 문제로 인해 DNS 요청이 실패할 수 있습니다. 이 문제를 해결 하려면 다음 레지스트리 키를 사용 하 여 최대 TTL 캐시 값을 0으로 설정 하면 됩니다.

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>여전히 문제가 발생 하 고 있습니다. 어떻게 해야 하나요? ### 
네트워크 또는 호스트에는 노드 간 특정 유형의 통신을 방지 하는 추가 제한 사항이 있을 수 있습니다. 다음 사항을 확인합니다.
  - 선택한 [네트워크 토폴로지](./network-topologies.md) 를 적절 하 게 구성 했습니다.
  - pod에서 오는 것 처럼 보이는 트래픽이 허용 됩니다.
  - 웹 서비스를 배포 하는 경우 HTTP 트래픽이 허용 됩니다.
  - 다른 프로토콜 (ie ICMP 및 TCP/UDP)의 패킷이 삭제 되지 않습니다.

>[!TIP]
> 추가 자가 진단 리소스에 대 한 자세한 내용은 [여기에 제공](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)되는 Windows에 대 한 Kubernetes 문제 해결 가이드도 있습니다.

## <a name="common-windows-errors"></a>일반적인 Windows 오류 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>내 Kubernetes pod는 "ContainerCreating"에 걸려 있습니다. ###
이 문제는 여러 가지 원인이 있을 수 있지만 가장 일반적인 방법 중 하나는 일시 중지 이미지가 잘못 구성 된 것입니다. 다음 문제에 대 한 높은 수준의 증상입니다.


### <a name="when-deploying-docker-containers-keep-restarting"></a>배포할 때 Docker 컨테이너는 계속 다시 시작 됩니다. ###
일시 중지 이미지가 OS 버전과 호환 되는지 확인 합니다. 이 [지침](./deploying-resources.md) 에서는 OS와 컨테이너 모두 버전 1803 이라고 가정 합니다. 이후 버전의 Windows (예: Insider build)가 있는 경우 이미지를 적절 하 게 조정 해야 합니다. 이미지에 대 한 Microsoft의 [Docker 리포지토리](https://hub.docker.com/u/microsoft/) 를 참조 하세요. 상관 없이 일시 중지 이미지 Dockerfile과 샘플 서비스는 모두 이미지를로 태그를 지정 합니다 `:latest` .


## <a name="common-kubernetes-master-errors"></a>일반적인 Kubernetes 마스터 오류 ##
Kubernetes 마스터 디버깅은 다음과 같은 세 가지 주요 범주로 나뉩니다.

  - Kubernetes 시스템 컨테이너에 문제가 있습니다.
  - 가 실행 되는 방식에 문제가 `kubelet` 있습니다.
  - 시스템에 문제가 있습니다.

`kubectl get pods -n kube-system`을 실행 하 여 Kubernetes에 의해 생성 되는 pod를 확인 합니다 .이를 통해 특정 충돌 또는 올바르게 시작 되지 않는 특정 항목을 파악할 수 있습니다. 그런 다음를 실행 `docker ps -a` 하 여 이러한 pod를 다시 실행 하는 모든 원시 컨테이너를 확인 합니다. 마지막으로, `docker logs [ID]` 문제의 원인이 되는 것으로 의심 되는 컨테이너에서를 실행 하 여 프로세스의 원시 출력을 표시 합니다.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>에서 API 서버에 연결할 수 없습니다.`https://[address]:[port]` ###
그렇지 않으면이 오류는 인증서 문제를 나타냅니다. 구성 파일을 올바르게 생성 했는지 확인 하 고, 해당 IP 주소가 호스트의 IP 주소와 일치 하 고, API 서버에서 탑재 된 디렉터리에 해당 파일을 복사 했는지 확인 합니다.

[지침](./creating-a-linux-master.md)을 따르면 다음과 같은 것이 좋습니다.   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 그렇지 않으면 API 서버의 매니페스트 파일을 참조 하 여 탑재 위치를 확인 합니다.
