---
title: Kubernetes를 Windows 서비스로 실행
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Windows 서비스로 Kubernetes 구성 요소를 실행 하는 방법.
keywords: kubernetes, 1.13, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 6c68edda6e2017640b0a490c3c30f063c81698b3
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120598"
---
# <a name="kubernetes-components-as-windows-services"></a>Windows 서비스로 Kubernetes 구성 요소 

일부 사용자 프로세스 flanneld.exe, kubelet.exe, kube proxy.exe 또는 다른 사용자가 Windows 서비스로 실행을 같은 구성 하고자 할 수도 있습니다. 이렇게 하면 예기치 않은 프로세스 또는 노드 충돌 시 자동으로 다시 시작 프로세스와 같은 추가 내결함성을 혜택.


## <a name="prerequisites"></a>필수 조건
1. [Nssm.exe](https://nssm.cc/download) 에 다운로드 했다고 합니다 `c:\k` 디렉터리
2. 노드 클러스터에 가입 되어 있고 이전에 노드 [install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) 또는 [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) 스크립트를 실행

## <a name="registering-windows-services"></a>Windows 서비스 등록
사용 하 여 nssm.exe는 등록 하는 [샘플 스크립트](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) 를 실행할 수 있습니다 `kubelet`, `kube-proxy`, 및 `flanneld.exe` 백그라운드에서 Windows 서비스로 실행 하려면:

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
Windows 노드에 할당 된 IP 주소입니다. 사용할 수 있습니다 `ipconfig` 검색 합니다.

|  |  | 
|---------|---------|
|매개 변수     | `-ManagementIP`        |
|기본값    | n.A.        |


# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
네트워크 모드 `l2bridge` (flannel 호스트 gw) 또는 `overlay` (flannel vxlan) [네트워크 솔루션](./network-topologies.md)으로 선택 합니다.

> [!Important] 
> `overlay` 네트워킹 모드 (flannel vxlan) Kubernetes v1.14 바이너리 필요 이상.

|  |  | 
|---------|---------|
|매개 변수     | `-NetworkMode`        |
|기본값    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[클러스터 서브넷 범위](./getting-started-kubernetes-windows.md#cluster-subnet-def)입니다.

|  |  | 
|---------|---------|
|매개 변수     | `-ClusterCIDR`        |
|기본값    | `10.244.0.0/16`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Kubernetes DNS 서비스 IP](./getting-started-kubernetes-windows.md#kube-dns-def)합니다.

|  |  | 
|---------|---------|
|매개 변수     | `-KubeDnsServiceIP`        |
|기본값    | `10.96.0.10`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Kubelet 및 kube 프록시 로그는 해당 출력 파일로 리디렉션됩니다 있는 디렉터리입니다.

|  |  | 
|---------|---------|
|매개 변수     | `-LogDir`        |
|기본값    | `C:\k`        |

---


> [!TIP] 
> [문제 해결 섹션](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services) 을 참조 하십시오 문제가 있나요 해야 합니다

## <a name="manual-approach"></a>수동 접근 방식
해야는 [참조 되는 스크립트 위에](#registering-windows-services) 작동 하지 않으면이 섹션에서는 몇 가지 *샘플 명령을* 수동으로 단계별 이러한 서비스를 등록 하는 사용할 수 있는

> [!TIP] 
> [Kubelet 및 kube 프록시 이제 Windows 서비스로 실행할 수](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) 구성 하는 방법에 대 한 자세한 내용은 참조 `kubelet` 및 `kube-proxy` 을 통해 기본 Windows 서비스로 실행 `sc`.

### <a name="register-flanneldexe"></a>Flanneld.exe 등록
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Kubelet.exe 등록
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Kube proxy.exe 등록 (l2bridge / 호스트 gw)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Kube proxy.exe 등록 (오버레이 / vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```