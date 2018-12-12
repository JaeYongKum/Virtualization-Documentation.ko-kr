---
title: Kubernetes 문제 해결
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Kubernetes를 배포하고 Windows 노드를 가입할 때 발생하는 일반적인 문제에 대한 해결 방법입니다.
keywords: kubernetes, 1.12, linux, 컴파일
ms.openlocfilehash: a5e9369b000aa83aa7ec6ec9bb147f0fd844c820
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178916"
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes 문제 해결 #
이 페이지에서는 Kubernetes 설정, 네트워킹 및 배포 관련 몇 가지 일반적인 문제를 안내합니다.

> [!tip]
> [Microsoft의 문서 저장소](https://github.com/MicrosoftDocs/Virtualization-Documentation/)에서 PR을 제기하여 FAQ 항목을 제안해 주세요.

이 페이지는 다음 범주로 나뉩니다.
1. [일반적인 질문](#general-questions)
2. [일반적인 네트워킹 오류](#common-networking-errors)
3. [일반적인 Windows 오류](#common-windows-errors)
4. [일반적인 Kubernetes 마스터 오류](#common-kubernetes-master-errors)

## <a name="general-questions"></a>일반적인 질문 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>성공적으로 완료 하는 Windows에서 start.ps1 어떻게 알 수 있나요? ###
Kubelet, 표시 kube 프록시 및 (Flannel 네트워킹 솔루션으로 선택한) 하는 경우 로그에 표시 되 고 실행 된 노드를에서 실행 flanneld 호스트 에이전트 프로세스 멋지게 windows를 구분 합니다. 이 외에도 Windows 노드 Kubernetes 클러스터에 있는 "준비"으로 표시 해야 합니다.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>이 모든 멋지게 windows 대신 백그라운드에서 실행 하도록 구성할 수 있습니까? ###
Kubernetes 1.11 버전 부터는 kubelet 및 kube 프록시 실행할 수 있습니다 네이티브 [Windows 서비스](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). 또한 항상 항상 있습니다 (flanneld, kubelet 및 kube 프록시) 이러한 프로세스 백그라운드에서 실행 [nssm.exe](https://nssm.cc/) 와 같은 다른 서비스 관리자를 사용할 수 있습니다.


## <a name="common-networking-errors"></a>일반적인 네트워킹 오류 ##

### <a name="my-windows-pods-do-not-have-network-connectivity"></a>내 Windows 포드가 네트워크에 연결 되지 않았을 ###
모든 가상 컴퓨터를 사용 하는 경우 모든 VM 네트워크 어댑터에서 MAC 스푸핑이 활성화 되어 있는지 확인 합니다. 자세한 내용은 [스푸핑 방지](./getting-started-kubernetes-windows.md#disable-anti-spoofing-protection) 참조 하세요.


### <a name="my-windows-pods-cannot-ping-external-resources"></a>내 Windows 포드가 외부 리소스를 ping 할 수 없습니다 ###
Windows 포드가 오늘날 ICMP 프로토콜에 대 한 프로그래밍 아웃 바운드 규칙 필요는 없습니다. 그러나 TCP/UDP 지원 됩니다. 클러스터 외부 리소스에 대 한 연결을 보여 주기 위해 시도할 때 대체 하세요 `ping <IP>` 해당 `curl <IP>` 명령 합니다.

여전히 문제가 접하는, [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) 에서 네트워크 구성 일부 관심을 만한 될 가능성이 높습니다. 항상 정적 파일을 편집할 수, 구성을 새로 만든된 Kubernetes 리소스에 적용 됩니다.

왜 그럴까요?
Kubernetes 네트워킹 요구 사항 중 하나는 내부 NAT 하지 않고도 클러스터 통신을 위한입니다 ( [Kubernetes 모델](https://kubernetes.io/docs/concepts/cluster-administration/networking/)참조). 이 요구를 인정 하는 모든 통신에 대 한 [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) 여기서 되려면 아웃 바운드 NAT 우리가 하지 않을 있습니다. 그러나 즉는 ExceptionList에서 쿼리 하려는 외부 IP를 제외 해야 합니다. 경우에 Windows 포드에서 소통이 됩니다 SNAT'ed 올바르게 외부에서 응답을 받습니다. 이 관계에서 프로그램 ExceptionList에서 `cni.conf` 다음과 같이 표시 됩니다.
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Start.ps1를 시작한 후 Flanneld 눌려져 "to be created 네트워크에 대 한 대기" ###
이 문제의; 조사할는 다양 한 보고서가 대개이 flannel 네트워크의 관리 IP 설정 된 경우에 대 한 타이밍 문제입니다. Start.ps1 다시 시작 또는 같이 수동으로 다시 시작 하는 해결 방법은입니다.
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

현재 검토이 문제를 해결 하는 [PR](https://github.com/coreos/flannel/pull/1042) 이기도 합니다.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>내 Windows 포드가 /run/flannel/subnet.env 누락으로 인해 시작할 수 없습니다. ###
이 Flannel 제대로 시작 되지 않은 것을 나타냅니다. Flanneld.exe를 다시 시작 하려고 하거나 수 또는 복사할 수는 있지만 파일을 통해 수동으로에서 `/run/flannel/subnet.env` 에 Kubernetes 마스터 `C:\run\flannel\subnet.env` Windows 작업자 노드 및 수정 합니다 `FLANNEL_SUBNET` 여러 행. 예를 들어, 노드 서브넷 10.244.4.1/24 필요한 경우:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>내 끝점/Ip 누수가 발생 ###
끝점에서 누수를 일으킬 수 있는 2 현재 알려진된 문제 존재 합니다. 
1.  [알려진 문제는](https://github.com/kubernetes/kubernetes/issues/68511) 첫 번째 버전 1.11 Kubernetes 문제입니다. Kubernetes 1.11.2 1.11.0-버전을 사용 하지 마십시오.
2. 두 번째 [알려진 문제](https://github.com/docker/libnetwork/issues/1950) 누수를 끝점을 초래할 수 있는 끝점의 저장소에서 동시성 문제입니다. Docker EE 18.09 해결 방법은 받으려면 사용 해야 이상.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>내 포드로 인해 시작할 수 없습니다 "네트워크: 범위에 대 한 할당 하지 못했습니다." 오류 ###
이 IP 주소 공간 노드 사용 되 고 있음을 나타냅니다. [끝점 누수](#my-endpointsips-are-leaking)를 정리 하려면 영향을 받는 노드에서 모든 리소스 마이그레이션 및 다음 명령을 실행:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>내 Windows 노드가 서비스 IP를 사용하여 내 서비스에 액세스할 수 없습니다. ###
이는 Windows에서 현재 네트워킹 스택의 알려진 제한 사항입니다. 하지만 Windows *포드* **는** 서비스 IP에 액세스할 수 없습니다.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Kubelet 시작 시 네트워크 어댑터를 찾을 수 없음 ###
Windows 네트워킹 스택에는 Kubernetes 네트워킹을 위한 가상 어댑터가 작동되어야 합니다. 다음 명령이 관리 셀에 결과를 리턴하지 않는 경우 가상 네트워크 만들기 &mdash; Kubelet 작동에 필요한 전제 조건 &mdash; 가 실패했습니다.

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

`start-kubelet.ps1` 스크립트의 출력을 검토하여 가상 네트워크 작성 중에 오류가 발생했는지 확인하세요.

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>일정 시간이 지난 후 DNS 쿼리 해석 중 포드 중지 ###
Windows Server의 네트워킹 스택의 문제 캐싱 알려진된 DNS가 버전 1803 및 그 아래 실패에 대 한 DNS 요청을 경우가 발생할 수 있습니다. 이 문제를 해결 하려면 다음 레지스트리 키를 사용 하 여 0으로 최대 TTL 캐시 값을 설정할 수 있습니다.

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>문제가 계속 표시 합니다. 제가 뭘 해야 하나요? ### 
네트워크 또는 호스트에 특정 유형의 노드 간 통신을 방지하는 추가 제한 사항이 있을 수 있습니다. 다음을 확인합니다.
  - 선택한 [네트워크 토폴로지가](./network-topologies.md) 올바르게 구성
  - 포드에서 가져온 것처럼 보이는 트래픽이 허용되었는지 여부
  - 웹 서비스를 배포하는 경우 HTTP 트래픽이 허용되었는지 여부
  - 다른 프로토콜 (ie ICMP 및 TCP/UDP)에서 패킷은 삭제 되지 않습니다.


## <a name="common-windows-errors"></a>일반적인 Windows 오류 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>내 Kubernetes 포드가 "ContainerCreating"에서 멈춤 ###
이 문제에는 여러 원인이 있을 수 있지만 가장 일반적인 원인 중 하나는 일시 중지 이미지가 잘못 구성된 경우입니다. 이 문제는 다음 문제의 고급 증상입니다.


### <a name="when-deploying-docker-containers-keep-restarting"></a>배포 시 Docker 컨테이너가 재시작 유지 ###
일시 중지 이미지가 OS 버전과 호환되는지 확인합니다. [지침](./deploying-resources.md) 컨테이너와 OS 버전 1803 지 가정 합니다. 참가자 빌드와 같은 최신 버전의 Windows를 사용하는 경우 그에 따라 이미지를 조정해야 합니다. 이미지에 대해서는 Microsoft의 [Docker 저장소](https://hub.docker.com/u/microsoft/)를 참조하세요. 그럼에 도 불구하고 일시 중지 이미지 Dockerfile과 샘플 서비스 모두 이미지에 `:latest`라는 태그가 지정됩니다.


## <a name="common-kubernetes-master-errors"></a>일반적인 Kubernetes 마스터 오류 ##
Kubernetes 마스터 디버깅은 세 가지 주요 범주로 나누어집니다(발생 가능성 순서).

  - Kubernetes 시스템 컨테이너에 문제가 있습니다.
  - `kubelet`이 실행되는 방식에 문제가 있습니다.
  - 시스템에 문제가 있습니다.

`kubectl get pods -n kube-system`을 실행하여 Kubernetes에 의해 생성되고 있는 포드를 확인할 수 있습니다. 이를 통해 어떠한 것이 충돌되었으며 제대로 시작하지 않는지 파악할 수 있습니다. 그런 다음 `docker ps -a`를 실행하여 이러한 포드를 백업하는 모든 원시 컨테이너를 볼 수 있습니다. 마지막으로 문제를 일으킨 것으로 의심되는 컨테이너에서 `docker logs [ID]`를 실행하여 프로세스의 원시 출력을 볼 수 있습니다.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>`https://[address]:[port]`에서 API 서버에 연결할 수 없습니다. ###
대개 이 오류는 인증서에 문제가 있음을 의미합니다. 구성 파일을 올바르게 생성했는지, 그 안의 IP 주소가 호스트의 IP 주소와 일치하는지, 그리고 이를 API 서버에 의해 마운트된 디렉터리에 복사했는지 확인합니다.

[우리의 지침](./creating-a-linux-master.md)을 준수 하는 경우 이것이 찾으려고 좋은 넣습니다.   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 그렇지 않은 경우 마운트 지점을 확인 하려면 API 서버의 매니페스트 파일을 참조 하세요.