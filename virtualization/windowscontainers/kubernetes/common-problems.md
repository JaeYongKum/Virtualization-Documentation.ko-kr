---
title: Kubernetes 문제 해결
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Kubernetes를 배포하고 Windows 노드를 가입할 때 발생하는 일반적인 문제에 대한 해결 방법입니다.
keywords: kubernetes, 1.14, linux, 컴파일
ms.openlocfilehash: 471731ec50c7c03816a956bd7aae859ad218be6d
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/03/2019
ms.locfileid: "10332364"
---
# Kubernetes 문제 해결 #
이 페이지에서는 Kubernetes 설정, 네트워킹 및 배포 관련 몇 가지 일반적인 문제를 안내합니다.

> [!tip]
> [Microsoft의 문서 저장소](https://github.com/MicrosoftDocs/Virtualization-Documentation/)에서 PR을 제기하여 FAQ 항목을 제안해 주세요.

이 페이지는 다음 범주로 나뉩니다.
1. [일반적인 질문](#general-questions)
2. [일반적인 네트워킹 오류](#common-networking-errors)
3. [일반적인 Windows 오류](#common-windows-errors)
4. [일반적인 Kubernetes 마스터 오류](#common-kubernetes-master-errors)

## 일반적인 질문 ##

### 시작을 확인 하는 방법 Windows의 ps1이 성공적으로 완료 되었습니까? ###
Kubelet, kube-프록시 및 (네트워킹 솔루션으로 Flannel을 선택한 경우) flanneld는 별도의 PoSh 창에 표시 되는 실행 중인 로그를 사용 하 여 노드에서 실행 되는 호스트 에이전트 프로세스를 확인 해야 합니다. 이 외에도, Kubernetes 클러스터에 Windows 노드가 "준비"로 나열 되어야 합니다.

### PoSh windows 대신 백그라운드에서 모두 실행 하도록 구성할 수 있나요? ###
Kubernetes 버전 1.11부터 kubelet & kube-프록시를 기본 [Windows 서비스로](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)실행할 수 있습니다. 또한 [nssm](https://nssm.cc/) 같은 대체 서비스 관리자를 사용 하 여 항상 이러한 프로세스 (flanneld, kubelet & kube-프록시)를 백그라운드에서 실행할 수 있습니다. 예를 들어 [Kubernetes의 Windows 서비스](./kube-windows-services.md) 를 참조 하세요.

### Kubernetes 프로세스를 Windows 서비스로 실행 하는 데 문제가 있습니다. ###
초기 문제 해결을 위해 [nssm](https://nssm.cc/) 에서 다음 플래그를 사용 하 여 stdout 및 stderr을 출력 파일로 리디렉션할 수 있습니다.
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
자세한 내용은 공식 [nssm 사용](https://nssm.cc/usage) 문서를 참조 하세요.

## 일반적인 네트워킹 오류 ##

### 부하 분산 장치가 클러스터 노드에 일관 되지 않게 배관 됨 ###
Windows에서 kube-proxy는 클러스터의 모든 Kubernetes 서비스에 대해 HNS 부하 분산 장치를 만듭니다. (기본값) kube-프록시 구성에서는 많은 (일반적으로 100 개 +) 부하 분산 장치를 포함 하는 클러스터의 노드가 사용할 수 있는 임시 TCP 포트 (예, 동적 포트 범위 (기본적으로 포트 49152 ~ 65535)를 포함 합니다. 이는 모든 (DSR이 아닌) 부하 분산 장치에 대해 각 노드에 예약 된 포트 수가 많아 발생 하는 것입니다. 이 문제는 다음과 같은 kube 프록시에 오류가 발생할 수 있습니다.
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

사용자는 [CollectLogs](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) 스크립트를 실행 하 고 파일을 `*portrange.txt` 컨설팅 하 여이 문제를 확인할 수 있습니다.

또한 `CollectLogs.ps1` 이는 HNS 할당 논리를 모방 하 여 임시 TCP 포트 범위에서 포트 풀 할당을 테스트 하 고 성공/실패를 `reservedports.txt`보고 합니다. 이 스크립트는 10 개의 64 범위 (HNS 동작을 에뮬레이트하기 위해)를 예약 하 고 예약 성공 & 오류 수를 계산한 다음 할당 된 포트 범위를 해제 합니다. 10 보다 작은 성공 값은 임시 풀이 사용 가능한 공간이 부족 함을 나타냅니다. Heuristical에서 `reservedports.txt`대략적으로 사용할 수 있는 64 블록 포트 예약이 몇 개 있는지도 요약 됩니다.

이 문제를 해결 하려면 다음과 같은 몇 가지 단계를 수행 하면 됩니다.
1.  영구 솔루션의 경우 kube 부하 분산을 [DSR 모드로](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710)설정 해야 합니다. DSR 모드는 완전히 구현 되 고 최신 [Windows Server 참가자 빌드 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (또는 그 이상) 에서만 사용할 수 있습니다.
2. 해결 방법으로 사용자는 등 `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>`의 명령을 사용 하 여 사용 가능한 임시 포트의 기본 Windows 구성을 늘릴 수도 있습니다. *경고:* 기본 동적 포트 범위를 재정의 하는 경우 비 임시 범위에서 사용 가능한 TCP 포트를 사용 하는 호스트의 다른 프로세스/서비스에 영향을 미칠 수 있으므로이 범위를 신중 하 게 선택 해야 합니다.
3. Q1 2020에서 누적 업데이트를 통해 출시 하도록 예약 된 지능형 포트 풀 공유를 사용 하 여 비 DSR 모드 로드 균형 조정기에 확장성이 향상 되었습니다.

### HostPort 게시가 작동 하지 않음 ###
현재이 필드는 Windows CNI 플러그 인으로 허용 `containers.ports.hostPort` 되지 않으므로 Kubernetes 필드를 사용 하 여 포트를 게시할 수 없습니다. 노드의 포트를 게시 하는 시간 동안 NodePort 게시를 사용 하세요.

### "HnsCall에 오류가 발생 했습니다: 잘못 된 디스켓이 드라이브에 있습니다." 등의 오류를 보고 있습니다. ###
이 오류는 이전 HNS 개체를 해제 하지 않고 HNS에서 변경 내용을 적용 하거나 새 Windows 업데이트를 설치할 때 HNS 개체를 사용자 지정 하 여 수정할 때 발생할 수 있습니다. 이는 업데이트 전에 이전에 만든 HNS 개체가 현재 설치 된 HNS 버전과 호환 되지 않음을 나타냅니다.

Windows Server 2019 (이 하)에서는 사용자가 HNS를 삭제 하 여 HNS 개체를 삭제할 수 있습니다. 데이터 파일 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

사용자는 호환 되지 않는 HNS 끝점 또는 네트워크를 직접 삭제할 수 있습니다.
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Windows Server 버전 1903의 사용자는 다음 레지스트리 위치로 이동 하 여 네트워크 이름 (예: `vxlan0` or `cbr0`)으로 시작 하는 모든 nic를 삭제할 수 있습니다.
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### Azure에서 내 Flannel host-gw 배포의 컨테이너가 인터넷에 액세스할 수 없음 ###
Azure의 호스트-gw 모드에서 Flannel를 배포 하는 경우 패킷이 Azure 실제 호스트 vSwitch를 통과 해야 합니다. 사용자는 노드에 할당 된 각 서브넷에 대해 "가상 기기" 유형의 [사용자 정의 경로](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined) 를 프로그래밍 해야 합니다. 이 작업은 Azure portal ( [여기](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)참조) 또는 `az` azure CLI를 통해 수행할 수 있습니다. 다음은 IP 10.0.0.4 및 각 pod 서브넷 10.244.0.0/24를 사용 하는 노드에 대 한 az 명령을 사용 하는 이름 "MyRoute"를 사용 하는 예제 1입니다.
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> 다른 클라우드 공급자의 Azure 또는 IaaS Vm에 Kubernetes를 배포 하는 경우에는 [오버레이 네트워킹](./network-topologies.md#flannel-in-vxlan-mode) 을 대신 사용할 수도 있습니다.

### 내 Windows pods에서 외부 리소스를 ping 할 수 없음 ###
Windows pods에는 지금 ICMP 프로토콜에 대 한 아웃 바운드 규칙이 프로그래밍 되어 있지 않습니다. 그러나 TCP/UDP는 지원 됩니다. 클러스터 외부의 리소스에 대 한 연결을 보여 줄 때 해당 `ping <IP>` `curl <IP>` 명령으로 대체 하세요.

여전히 문제가 발생 하는 경우에는 [cni.](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) e a i의 네트워크 구성에 몇 가지 추가 주의가 필요할 수 있습니다. 언제 든 지이 정적 파일을 편집할 수 있으며,이 구성은 새로 만든 Kubernetes 리소스에 적용 됩니다.

왜 그럴까요?
Kubernetes 네트워킹 요구 사항 중 하나 ( [Kubernetes model](https://kubernetes.io/docs/concepts/cluster-administration/networking/)참조)는 NAT 없이 내부적으로 발생 하는 클러스터 통신에만 해당 됩니다. 이 요구 사항을 충족 하기 위해 아웃 바운드 NAT가 필요 하지 않은 모든 통신에 대 한 [예외](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) 를 발생 시킬 수 있습니다. 그러나이는 쿼리를 수행 하려는 외부 IP를 예외 표시에서 제외 해야 한다는 의미 이기도 합니다. 그런 다음에는 Windows pods에서 발생 하는 트래픽이 외부 세계 로부터 응답을 받을 수 있도록 올바르게 SNAT'ed 됩니다. 이런 측면에서의 `cni.conf` 예외 표시는 다음과 같습니다.
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### 내 Windows 노드가 NodePort 서비스에 액세스할 수 없음 ###
노드 자체의 로컬 NodePort 액세스는 실패 합니다. 이는 잘 알려진 제한 사항입니다. 다른 노드 또는 외부 클라이언트에서 NodePort 액세스를 사용할 수 있습니다.

### 잠시 후 컨테이너의 vNICs 및 HNS 끝점이 삭제 됩니다. ###
이 문제는 `hostname-override` 매개 변수가 [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)에 전달 되지 않은 경우에 발생할 수 있습니다. 이를 해결 하기 위해 사용자는 다음과 같이 kube 프록시에 hostname을 전달 해야 합니다.
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### Flannel (vxlan) 모드에서 pods는 노드를 rejoining 후 연결 문제가 발생 합니다. ###
이전에 삭제 된 노드가 클러스터에 다시 가입 될 때마다 flannelD가 노드에 새 pod 서브넷을 할당 하려고 합니다. 사용자는 다음 경로에서 이전 pod 서브넷 구성 파일을 제거 해야 합니다.
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### Start. ps1을 실행 한 후에는 "네트워크 생성을 기다리는 중 Flanneld"이 중지 됩니다. ###
이 문제에 대 한 여러 보고서가 조사 중입니다. 대부분의 경우 flannel 네트워크의 관리 IP가 설정 되어 있는 경우에 대 한 타이밍 문제가 될 수 있습니다. 해결 방법은 다음과 같이 시작을 다시 실행 하는 것입니다.
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

현재 검토 중에이 문제를 해결 하는 [PR](https://github.com/coreos/flannel/pull/1042) 도 있습니다.


### 누락 된/run/flannel/subnet.env 때문에 Windows pods를 시작할 수 없습니다. ###
이는 Flannel 올바르게 실행 되지 않았음을 의미 합니다. Flanneld를 다시 시작 하거나 Kubernetes 마스터 `/run/flannel/subnet.env` `C:\run\flannel\subnet.env` 에서 Windows 작업자 노드에 수동으로 파일을 복사 하 고 할당 된 서브넷으로 `FLANNEL_SUBNET` 행을 수정할 수 있습니다. 예를 들어 노드 서브넷 10.244.4.1/24가 지정 된 경우:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Flanneld에서이 파일을 자동으로 생성 하는 것이 더 안전 합니다.


### VSphere 실행 되는 Kubernetes 클러스터에서 호스트 간의 pod-pod 연결이 중단 되었습니다. 
VSphere Flannel는 오버레이 네트워킹에 대 한 포트 4789 (기본 VXLAN 포트)을 보유 하 고 있기 때문에 패킷이 차단 될 수 있습니다. VSphere 오버레이 네트워킹에 사용 되는 경우에는 4789을 확보 하기 위해 다른 포트를 사용 하도록 구성 해야 합니다.  


### 내 끝점/i p i 누수가 발생 하는 경우 ###
현재 알려진 두 가지 문제가 있어 끝점에 누수가 발생할 수 있습니다. 
1.  첫 번째 [알려진 문제](https://github.com/kubernetes/kubernetes/issues/68511) 는 Kubernetes 버전 1.11의 문제입니다. Kubernetes 버전 1.11.0-1.11.2을 사용 하지 마세요.
2. 끝점을 누출 시킬 수 있는 두 번째 [알려진 문제](https://github.com/docker/libnetwork/issues/1950) 는 끝점 저장소의 동시성 문제입니다. 수정을 받으려면 Docker EE 18.09 이상을 사용 해야 합니다.

### "네트워크: 범위에 할당할 수 없습니다." 오류 때문에 내 pods를 실행할 수 없습니다. ###
이는 노드의 IP 주소 공간이 사용 됨을 나타냅니다. [누수 된 끝점](#my-endpointsips-are-leaking)을 정리 하려면 영향을 받는 노드의 리소스를 모두 마이그레이션한 다음 명령을 실행 & 하세요.
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### 내 Windows 노드가 서비스 IP를 사용하여 내 서비스에 액세스할 수 없습니다. ###
이는 Windows에서 현재 네트워킹 스택의 알려진 제한 사항입니다. 그러나 Windows *pods* **는** 서비스 IP에 액세스할 수 있습니다.

### Kubelet 시작 시 네트워크 어댑터를 찾을 수 없음 ###
Windows 네트워킹 스택에는 Kubernetes 네트워킹을 위한 가상 어댑터가 작동되어야 합니다. 다음 명령이 관리 셀에 결과를 리턴하지 않는 경우 가상 네트워크 만들기 &mdash; Kubelet 작동에 필요한 전제 조건 &mdash; 가 실패했습니다.

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

호스트의 네트워크 어댑터가 "Ethernet"이 아닌 경우에는 시작. ps1 스크립트의 [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) 매개 변수를 수정 하는 것이 좋습니다. 그렇지 않은 경우에는 `start-kubelet.ps1` 스크립트의 출력을 참조 하 여 가상 네트워크를 만드는 동안 오류가 있는지 확인 합니다. 

### 일정 시간이 지난 후 DNS 쿼리 해석 중 포드 중지 ###
Windows Server의 네트워킹 스택에 알려진 DNS 캐싱 문제가 있는 경우 (간혹 DNS 요청이 실패할 수 있음) 버전 1803이 발생 합니다. 이 문제를 해결 하려면 다음 레지스트리 키를 사용 하 여 최대 TTL 캐시 값을 0으로 설정할 수 있습니다.

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### 문제가 계속 발생 합니다. 제가 뭘 해야 하나요? ### 
네트워크 또는 호스트에 특정 유형의 노드 간 통신을 방지하는 추가 제한 사항이 있을 수 있습니다. 다음을 확인합니다.
  - 선택한 [네트워크 토폴로지](./network-topologies.md) 를 적절 하 게 구성 했습니다.
  - 포드에서 가져온 것처럼 보이는 트래픽이 허용되었는지 여부
  - 웹 서비스를 배포하는 경우 HTTP 트래픽이 허용되었는지 여부
  - 다른 프로토콜 (ie ICMP vs TCP/UDP)의 패킷이 삭제 되지 않음

>[!TIP]
> 다른 자체 지원 리소스에 대 한 자세한 내용은 Windows에 대 한 Kubernetes 문제 해결 안내서를 [참조 하세요.](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)

## 일반적인 Windows 오류 ##

### 내 Kubernetes 포드가 "ContainerCreating"에서 멈춤 ###
이 문제에는 여러 원인이 있을 수 있지만 가장 일반적인 원인 중 하나는 일시 중지 이미지가 잘못 구성된 경우입니다. 이 문제는 다음 문제의 고급 증상입니다.


### 배포 시 Docker 컨테이너가 재시작 유지 ###
일시 중지 이미지가 OS 버전과 호환되는지 확인합니다. 이 [지침](./deploying-resources.md) 에서는 OS와 컨테이너의 버전이 모두 1803 것으로 가정 합니다. 참가자 빌드와 같은 최신 버전의 Windows를 사용하는 경우 그에 따라 이미지를 조정해야 합니다. 이미지에 대해서는 Microsoft의 [Docker 저장소](https://hub.docker.com/u/microsoft/)를 참조하세요. 그럼에 도 불구하고 일시 중지 이미지 Dockerfile과 샘플 서비스 모두 이미지에 `:latest`라는 태그가 지정됩니다.


## 일반적인 Kubernetes 마스터 오류 ##
Kubernetes 마스터 디버깅은 세 가지 주요 범주로 나누어집니다(발생 가능성 순서).

  - Kubernetes 시스템 컨테이너에 문제가 있습니다.
  - `kubelet`이 실행되는 방식에 문제가 있습니다.
  - 시스템에 문제가 있습니다.

`kubectl get pods -n kube-system`을 실행하여 Kubernetes에 의해 생성되고 있는 포드를 확인할 수 있습니다. 이를 통해 어떠한 것이 충돌되었으며 제대로 시작하지 않는지 파악할 수 있습니다. 그런 다음 `docker ps -a`를 실행하여 이러한 포드를 백업하는 모든 원시 컨테이너를 볼 수 있습니다. 마지막으로 문제를 일으킨 것으로 의심되는 컨테이너에서 `docker logs [ID]`를 실행하여 프로세스의 원시 출력을 볼 수 있습니다.


### 에서 API 서버에 연결할 수 없습니다. `https://[address]:[port]` ###
대개 이 오류는 인증서에 문제가 있음을 의미합니다. 구성 파일을 올바르게 생성했는지, 그 안의 IP 주소가 호스트의 IP 주소와 일치하는지, 그리고 이를 API 서버에 의해 마운트된 디렉터리에 복사했는지 확인합니다.

[지침](./creating-a-linux-master.md)을 팔 로우 하는 경우에는 다음과 같은 좋은 위치를 찾을 수 있습니다.   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 그렇지 않으면 API 서버의 매니페스트 파일을 참조 하 여 탑재 지점을 확인 합니다.
