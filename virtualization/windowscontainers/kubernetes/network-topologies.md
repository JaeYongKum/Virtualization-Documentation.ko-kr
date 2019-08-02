---
title: 네트워크 토폴로지
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows 및 Linux에서 지원 되는 네트워크 토폴로지.
keywords: kubernetes, 1.14, windows, 시작 하기
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 42cb47ba4f3e22163869d094bd0c9cff415a43b0
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/02/2019
ms.locfileid: "9884984"
---
# <a name="network-solutions"></a>네트워크 솔루션 #

[Kubernetes 마스터 노드 설정을](./creating-a-linux-master.md) 완료 하면 네트워킹 솔루션을 선택할 수 있습니다. 노드 간에 가상 [클러스터 서브넷](./getting-started-kubernetes-windows.md#cluster-subnet-def) 을 라우팅할 수 있는 여러 가지 방법이 있습니다. Windows 오늘 Kubernetes에 대해 다음 옵션 중 하나를 선택 합니다.

1. [Flannel](#flannel-in-vxlan-mode) 와 같은 cni 플러그 인을 사용 하 여 오버레이 네트워크를 설정 합니다.
2. [Flannel](#flannel-in-host-gateway-mode) 와 같은 cni 플러그 인을 사용 하 여 경로를 프로그래밍 합니다 (l2bridge 네트워킹 모드 사용).
3. 서브넷을 라우팅하도록 스마트 [상판 (랙 윗면) 스위치](#configuring-a-tor-switch) 를 구성 합니다.

> [!tip]  
> 개방형 vSwitch (OvS) 및 개방형 가상 네트워크 (OVN)를 활용 하는 Windows에는 네 번째 네트워킹 솔루션이 있습니다. 이 문서에 대 한 문서가 범위를 벗어났습니다. 하지만이 [지침](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) 을 읽고이를 설정할 수 있습니다.

## <a name="flannel-in-vxlan-mode"></a>Vxlan 모드 Flannel

Vxlan 모드의 Flannel는 VXLAN 터널링을 사용 하 여 노드 간에 패킷을 라우팅하는 구성 가능한 가상 오버레이 네트워크를 설정 하는 데 사용 될 수 있습니다.

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel Kubernetes 마스터 준비
클러스터의 [Kubernetes 마스터](./creating-a-linux-master.md) 에 몇 가지 사소한 준비를 하는 것이 좋습니다. Flannel를 사용 하는 경우에는 iptables 체인에 대해 브리지 된 IPv4 트래픽을 사용 하는 것이 좋습니다. 다음 명령을 사용 하 여이 작업을 수행할 수 있습니다.

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Flannel 구성 다운로드 & ###
최신 Flannel 매니페스트 다운로드:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Vxlan 네트워킹 백 엔드를 사용 하도록 설정 하려면 다음 두 가지 섹션을 수정 해야 합니다.

1. 의 `net-conf.json` 섹션에서 다음 `kube-flannel.yml`을 두 번 확인 합니다.
 * 클러스터 서브넷 (예: "10.244.0.0/16")이 원하는 대로 설정 됩니다.
 * VNI 4096이 백 엔드에 설정 됨
 * 포트 4789이 백 엔드에 설정 됨
2. 의 `cni-conf.json` `kube-flannel.yml`섹션에서 네트워크 이름을으로 `"vxlan0"`변경 합니다.

위의 단계를 적용 한 후 다음과 `net-conf.json` 같이 확인 해야 합니다.
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
> Windows의 Flannel와 상호 운용 하려면 Linux에서 Flannel에 대 한 VNI을 4096 및 포트 4789로 설정 해야 합니다. 다른 VNIs에 대 한 지원이 곧 제공 될 예정입니다. 이러한 필드에 대 한 설명은 [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 을 참조 하세요.

다음과 `cni-conf.json` 같이 확인 해야 합니다.
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
> 위의 옵션에 대 한 자세한 내용은 Linux 용 공식 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin), [브리지](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) 플러그 인 문서를 참조 하세요.

### <a name="launch-flannel--validate"></a>Flannel & 실행 확인 ###
다음을 사용 하 여 Flannel 실행:

```bash
kubectl apply -f kube-flannel.yml
```

그 다음, Flannel pods는 Linux 기반 이므로 Linux [Nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 패치를 대상 linux에만 `kube-flannel-ds` 적용 하 여 DaemonSet 다음에 참가할 때 Windows에서 Flannel "flanneld" 호스트 에이전트 프로세스를 실행 합니다.

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> X86-64 기반 노드가 아닌 경우 위의 프로세서 아키텍처로 `-amd64` 대체 합니다.

몇 분 후에는 Flannel pod 네트워크를 배포한 경우 모든 pods 실행 중으로 표시 됩니다.

```bash
kubectl get pods --all-namespaces
```

![텍스트](media/kube-master.png)

Flannel DaemonSet에는 NodeSelector `beta.kubernetes.io/os=linux` 도 적용 해야 합니다.

```bash
kubectl get ds -n kube-system
```

![텍스트](media/kube-daemonset.png)

> [!tip]  
> 나머지 flannel-ds-* DaemonSets의 경우이는 해당 프로세서 아키텍처와 일치 하는 노드가 없는 경우에는 예약 되지 않으므로 무시 되거나 삭제 될 수 있습니다.

> [!tip]  
> 혼동할? 이 단계는 기본 클러스터 서브넷 `10.244.0.0/16`에 대해 미리 적용 된 flannel v 0.11.0에 대 한 전체 [예제 kube-flannel.](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml)

성공적으로 완료 되 면 [다음 단계](#next-steps)를 계속 합니다.

## <a name="flannel-in-host-gateway-mode"></a>Flannel 호스트 게이트웨이 모드

[Flannel vxlan](#flannel-in-vxlan-mode)와 함께 Flannel 네트워킹의 다른 옵션은 호스트 *게이트웨이 모드* (호스트-gw)로, 각 노드의 고정 경로를 다음 홉으로 사용 하는 다른 노드의 pod 서브넷에 대 한 프로그래밍을 수반 합니다.

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel Kubernetes 마스터 준비

클러스터의 [Kubernetes 마스터](./creating-a-linux-master.md) 에 몇 가지 사소한 준비를 하는 것이 좋습니다. Flannel를 사용 하는 경우에는 iptables 체인에 대해 브리지 된 IPv4 트래픽을 사용 하는 것이 좋습니다. 다음 명령을 사용 하 여이 작업을 수행할 수 있습니다.

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Flannel 구성 다운로드 & ###
최신 Flannel 매니페스트 다운로드:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Windows/Linux에서 호스트 gw 네트워킹을 사용 하도록 설정 하기 위해 변경 해야 하는 파일이 하나 있습니다.

Kube-flannel의 `net-conf.json` 섹션에서 다음을 두 번 확인 합니다.
1. 사용 중인 네트워크 백 엔드 유형이 대신로 `host-gw` 설정 되어 `vxlan`있습니다.
2. 클러스터 서브넷 (예: "10.244.0.0/16")이 원하는 대로 설정 됩니다.

2 단계를 적용 한 후 다음과 `net-conf.json` 같이 확인 해야 합니다.
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Flannel & 실행 확인 ###
다음을 사용 하 여 Flannel 실행:

```bash
kubectl apply -f kube-flannel.yml
```

그 다음, Flannel pods는 Linux 기반 이므로 Linux [Nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 패치를 대상 linux에만 `kube-flannel-ds` 적용 하 여 DaemonSet 다음에 참가할 때 Windows에서 Flannel "flanneld" 호스트 에이전트 프로세스를 실행 합니다.

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> X86-64 기반 노드가 아닌 경우 원하는 프로세서 아키텍처로 `-amd64` 대체 합니다.

몇 분 후에는 Flannel pod 네트워크를 배포한 경우 모든 pods 실행 중으로 표시 됩니다.

```bash
kubectl get pods --all-namespaces
```

![텍스트](media/kube-master.png)

Flannel DaemonSet에는 NodeSelector도 적용 해야 합니다.

```bash
kubectl get ds -n kube-system
```

![텍스트](media/kube-daemonset.png)

> [!tip]  
> 나머지 flannel-ds-* DaemonSets의 경우이는 해당 프로세서 아키텍처와 일치 하는 노드가 없는 경우에는 예약 되지 않으므로 무시 되거나 삭제 될 수 있습니다.

> [!tip]  
> 혼동할? 다음은 기본 클러스터 서브넷 `10.244.0.0/16`에 대해 미리 적용 된 다음 2 단계가 있는 flannel v 0.11.0에 대 한 전체 [예제 kube-flannel.](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml)

성공적으로 완료 되 면 [다음 단계](#next-steps)를 계속 합니다.

## <a name="configuring-a-tor-switch"></a>인 데이 스위치 구성 ##
> [!NOTE]
> [네트워킹 솔루션으로 Flannel를](#flannel-in-host-gateway-mode)선택한 경우이 섹션을 건너뛸 수 있습니다.
사용자 전환의 구성은 실제 노드 외부에서 이루어집니다. 이에 대 한 자세한 내용은 [공식 Kubernetes 문서](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)를 참조 하세요.


## <a name="next-steps"></a>다음 단계 ## 
이 섹션에서는 네트워킹 솔루션을 선택 하 고 구성 하는 방법에 대해 설명 합니다. 이제 4 단계를 수행할 준비가 되었습니다.

> [!div class="nextstepaction"]
> [Windows 작업자 참가](./joining-windows-workers.md)