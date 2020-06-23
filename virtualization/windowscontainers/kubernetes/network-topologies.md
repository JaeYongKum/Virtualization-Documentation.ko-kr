---
title: 네트워크 토폴로지
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
ms.prod: containers
description: Windows 및 Linux에서 지원 되는 네트워크 토폴로지
keywords: kubernetes, 1.14, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c322edb6a5ead34d7988f83d8cb8fba7c99cec0d
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192540"
---
# <a name="network-solutions"></a>Network Solutions #

[Kubernetes 마스터 노드를 설정](./creating-a-linux-master.md) 하면 네트워킹 솔루션을 선택할 준비가 된 것입니다. 여러 가지 방법으로 가상 [클러스터 서브넷](./getting-started-kubernetes-windows.md#cluster-subnet-def) 을 노드 간에 라우팅할 수 있습니다. 현재 Windows에서 Kubernetes에 대 한 다음 옵션 중 하나를 선택 합니다.

1. [Flannel](#flannel-in-vxlan-mode) 와 같은 cni 플러그 인을 사용 하 여 오버레이 네트워크를 설정 합니다.
2. [Flannel](#flannel-in-host-gateway-mode) 와 같은 cni 플러그 인을 사용 하 여 경로를 프로그래밍 합니다 (l2bridge 네트워킹 모드 사용).
3. 서브넷을 라우팅하도록 스마트 [랙 (랙) 스위치](#configuring-a-tor-switch) 를 구성 합니다.

> [!tip]
> Windows에는 OvS (Open vSwitch) 및 Open Virtual Network (OVN)를 활용 하는 네 번째 네트워킹 솔루션이 있습니다. 이 문서는이 문서의 범위를 벗어나는 것 이지만 [이러한 지침](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) 을 참조 하 여 설정할 수 있습니다.

## <a name="flannel-in-vxlan-mode"></a>Vxlan 모드의 Flannel

Flannel vxlan 모드는 VXLAN 터널링을 사용 하 여 노드 간에 패킷을 라우팅하는 구성 가능한 가상 오버레이 네트워크를 설정 하는 데 사용할 수 있습니다.

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel에 대 한 Kubernetes 마스터 준비
클러스터의 [Kubernetes 마스터](./creating-a-linux-master.md) 에 몇 가지 사소한 준비가 권장 됩니다. Flannel를 사용 하는 경우 iptables 체인에 대해 브리지 된 IPv4 트래픽을 사용 하도록 설정 하는 것이 좋습니다. 다음 명령을 사용 하 여이 작업을 수행할 수 있습니다.

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Flannel를 다운로드 & 구성 ###
최신 Flannel 매니페스트를 다운로드 합니다.

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Vxlan 네트워킹 백 엔드를 사용 하도록 설정 하려면 다음 두 가지 섹션을 수정 해야 합니다.

1. `net-conf.json`의 섹션에서 `kube-flannel.yml` 다음을 두 번 확인 합니다.
 * 클러스터 서브넷 (예: "10.244.0.0/16")은 원하는 대로 설정 됩니다.
 * 백 엔드에서 VNI 4096가 설정 됨
 * 백 엔드에서 포트 4789가 설정 됨
2. `cni-conf.json`의 섹션에서 `kube-flannel.yml` 네트워크 이름을로 변경 `"vxlan0"` 합니다.

위의 단계를 적용 한 후에는 다음과 같이 표시 `net-conf.json` 됩니다.
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]
> Windows에서 Flannel와 상호 운용할 수 있도록 Linux에서 Flannel의 경우 VNI을 4096로 설정 하 고 포트 4789을 설정 해야 합니다. 다른 VNIs에 대 한 지원은 곧 제공 될 예정입니다. 이러한 필드에 대 한 설명은 [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 를 참조 하세요.

다음과 같이 표시 `cni-conf.json` 됩니다.
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]
> 위의 옵션에 대 한 자세한 내용은 공식 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)및 Linux 용 [브리지](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) 플러그 인 문서를 참조 하세요.

### <a name="launch-flannel--validate"></a>Flannel & 유효성 검사 시작 ###
다음을 사용 하 여 Flannel 시작:

```bash
kubectl apply -f kube-flannel.yml
```

다음으로, Flannel pod는 Linux 기반 이므로 linux [Nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 패치를 `kube-flannel-ds` 대상 linux에만 적용 합니다. 나중에 조인할 때 Windows에서 Flannel "flanneld" 호스트 에이전트 프로세스를 시작 합니다.

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]
> X86-64 기반이 아닌 노드가 있는 경우 `-amd64` 위의 프로세서 아키텍처로 대체 합니다.

몇 분 후에 Flannel pod 네트워크가 배포 된 경우 pod가 실행 중으로 표시 되어야 합니다.

```bash
kubectl get pods --all-namespaces
```

![text](media/kube-master.png)

Flannel DaemonSet에는 NodeSelector도 적용 해야 합니다 `beta.kubernetes.io/os=linux` .

```bash
kubectl get ds -n kube-system
```

![text](media/kube-daemonset.png)

> [!tip]
> 나머지 flannel-* DaemonSets의 경우 해당 프로세서 아키텍처와 일치 하는 노드가 없는 경우에는 예약 되지 않으므로 무시 하거나 삭제할 수 있습니다.

> [!tip]
> 스? 다음은 기본 클러스터 서브넷에 대해 이러한 단계가 미리 적용 된 Flannel v 0.11.0에 대 한 전체 [예제입니다.](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) `10.244.0.0/16`

성공적으로 완료 되 면 [다음 단계](#next-steps)를 계속 합니다.

## <a name="flannel-in-host-gateway-mode"></a>호스트-게이트웨이 모드에서 Flannel

[Flannel vxlan](#flannel-in-vxlan-mode)와 함께 Flannel 네트워킹에 대 한 또 다른 옵션은 *호스트 게이트웨이 모드* (gw) 이며,이는 대상 노드의 호스트 주소를 다음 홉으로 사용 하 여 각 노드에서 다른 노드의 pod 서브넷으로의 고정 경로를 프로그래밍 하는 데 사용 됩니다.

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel에 대 한 Kubernetes 마스터 준비

클러스터의 [Kubernetes 마스터](./creating-a-linux-master.md) 에 몇 가지 사소한 준비가 권장 됩니다. Flannel를 사용 하는 경우 iptables 체인에 대해 브리지 된 IPv4 트래픽을 사용 하도록 설정 하는 것이 좋습니다. 다음 명령을 사용 하 여이 작업을 수행할 수 있습니다.

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Flannel를 다운로드 & 구성 ###
최신 Flannel 매니페스트를 다운로드 합니다.

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Windows/Linux에서 호스트 gw 네트워킹을 사용 하도록 설정 하기 위해 변경 해야 하는 파일이 하나 있습니다.

`net-conf.json`Kube-flannel의 섹션에서 다음을 두 번 확인 합니다.
1. 사용 중인 네트워크 백 엔드의 유형은 대신로 설정 됩니다 `host-gw` `vxlan` .
2. 클러스터 서브넷 (예: "10.244.0.0/16")은 원하는 대로 설정 됩니다.

2 단계를 적용 한 후에는 다음과 같이 표시 `net-conf.json` 됩니다.
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Flannel & 유효성 검사 시작 ###
다음을 사용 하 여 Flannel 시작:

```bash
kubectl apply -f kube-flannel.yml
```

다음으로, Flannel pod는 Linux 기반 이므로 linux [Nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 패치를 `kube-flannel-ds` 대상 linux에만 적용 합니다. 나중에 조인할 때 Windows에서 Flannel "flanneld" 호스트 에이전트 프로세스를 시작 합니다.

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]
> X86-64 기반이 아닌 노드가 있는 경우 위의를 `-amd64` 원하는 프로세서 아키텍처로 바꿉니다.

몇 분 후에 Flannel pod 네트워크가 배포 된 경우 pod가 실행 중으로 표시 되어야 합니다.

```bash
kubectl get pods --all-namespaces
```

![text](media/kube-master.png)

Flannel DaemonSet에는 NodeSelector도 적용 해야 합니다.

```bash
kubectl get ds -n kube-system
```

![text](media/kube-daemonset.png)

> [!tip]
> 나머지 flannel-* DaemonSets의 경우 해당 프로세서 아키텍처와 일치 하는 노드가 없는 경우에는 예약 되지 않으므로 무시 하거나 삭제할 수 있습니다.

> [!tip]
> 스? 다음은 기본 클러스터 서브넷에 대해 이러한 2 단계가 미리 적용 된 Flannel v 0.11.0에 대 한 전체 [예제](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) `10.244.0.0/16` 입니다.

성공적으로 완료 되 면 [다음 단계](#next-steps)를 계속 합니다.

## <a name="configuring-a-tor-switch"></a>구성 스위치 구성 ##
> [!NOTE]
> [Flannel를 네트워킹 솔루션으로](#flannel-in-host-gateway-mode)선택한 경우이 섹션을 건너뛸 수 있습니다.
실제 노드 외부에서가 중 스위치의 구성이 발생 합니다. 자세한 내용은 [공식 Kubernetes 문서](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)를 참조 하세요.


## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 네트워킹 솔루션을 선택 하 고 구성 하는 방법을 설명 했습니다. 이제 4 단계를 수행할 준비가 되었습니다.

> [!div class="nextstepaction"]
> [Windows 작업자 참여](./joining-windows-workers.md)