---
title: Windows 서비스로 Kubernetes 실행
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: how-to
ms.prod: containers
description: Windows 서비스로 Kubernetes 구성 요소를 실행 하는 방법
keywords: kubernetes, 1.14, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 470538ad796773252c08c7295c5086d0002a55ce
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192490"
---
# <a name="kubernetes-components-as-windows-services"></a>Windows 서비스로 구성 요소 Kubernetes

일부 사용자는 flanneld.exe, kubelet.exe, kube-proxy.exe 등의 프로세스를 Windows 서비스로 실행 하도록 구성할 수 있습니다. 이로 인해 예기치 않은 프로세스나 노드 충돌 시 자동으로 다시 시작 하는 프로세스와 같은 추가적인 내결함성 이점이 있습니다.


## <a name="prerequisites"></a>필수 구성 요소
1. [nssm.exe](https://nssm.cc/download) 를 디렉터리에 다운로드 했습니다. `c:\k`
2. 클러스터에 노드를 조인 하 고 이전 노드에 [install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) 또는 [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) 스크립트를 실행 했습니다.

## <a name="registering-windows-services"></a>Windows 서비스 등록
, 및를 등록 하는 nssm.exe를 사용 하는 [샘플 스크립트](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) 를 실행 `kubelet` 하 여 `kube-proxy` `flanneld.exe` 백그라운드에서 Windows 서비스로 실행할 수 있습니다.

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementip"></a>[ManagementIP](#tab/ManagementIP)
Windows 노드에 할당 된 IP 주소입니다. 를 사용 하 여이를 찾을 수 있습니다 `ipconfig` .

|  |  |
|---------|---------|
|매개 변수     | `-ManagementIP`        |
|기본값    | N.a.        |


# <a name="networkmode"></a>[NetworkMode](#tab/NetworkMode)
네트워크 `l2bridge` 솔루션으로 선택 된 네트워크 모드 (flannel 호스트-gw) 또는 `overlay` (flannel vxlan [network solution](./network-topologies.md))입니다.

> [!Important]
> `overlay`네트워킹 모드 (flannel vxlan)에는 Kubernetes v 1.14 이진 이상이 필요 합니다.

|  |  |
|---------|---------|
|매개 변수     | `-NetworkMode`        |
|기본값    | `l2bridge`        |


# <a name="clustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
[클러스터 서브넷 범위](./getting-started-kubernetes-windows.md#cluster-subnet-def)입니다.

|  |  |
|---------|---------|
|매개 변수     | `-ClusterCIDR`        |
|기본값    | `10.244.0.0/16`        |


# <a name="kubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS 서비스 IP](./getting-started-kubernetes-windows.md#kube-dns-def)입니다.

|  |  |
|---------|---------|
|매개 변수     | `-KubeDnsServiceIP`        |
|기본값    | `10.96.0.10`        |


# <a name="logdir"></a>[LogDir](#tab/LogDir)
Kubelet 및 kube 로그가 해당 출력 파일로 리디렉션되는 디렉터리입니다.

|  |  |
|---------|---------|
|매개 변수     | `-LogDir`        |
|기본값    | `C:\k`        |

---


> [!TIP]
> 문제가 발생 하는 경우 [문제 해결 섹션](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services) 을 참조 하세요.

## <a name="manual-approach"></a>수동 접근 방식
[위의 참조 된 스크립트가](#registering-windows-services) 제대로 작동 하지 않을 경우이 섹션에서는 이러한 서비스를 수동으로 단계별로 등록 하는 데 사용할 수 있는 몇 가지 *샘플 명령을* 제공 합니다.

> [!TIP]
> 를 통해를 구성 하 고 네이티브 Windows 서비스로 실행 하는 방법에 대 한 자세한 내용은 [Kubelet 및 kube-이제 Windows 서비스로 실행 될 수 있습니다](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) `kubelet` `kube-proxy` `sc` .

### <a name="register-flanneldexe"></a>flanneld.exe 등록
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>kubelet.exe 등록
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>kube-proxy.exe 등록 (l2bridge/gw)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>kube-proxy.exe 등록 (오버레이/vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```