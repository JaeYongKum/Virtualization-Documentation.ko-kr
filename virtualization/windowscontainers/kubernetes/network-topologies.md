---
title: 네트워크 토폴로지
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows 및 Linux에서 네트워크 토폴로지를 지원 합니다.
keywords: kubernetes, 1.14, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6a2b7021efa0d90b69a88e1b498cddeadb3af80e
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622918"
---
# <a name="network-solutions"></a>네트워크 솔루션 #

[Kubernetes 마스터 노드를 설정](./creating-a-linux-master.md) 했으면 네트워킹 솔루션 선택 준비가 되었습니다. 노드 간에 가상 [클러스터 서브넷](./getting-started-kubernetes-windows.md#cluster-subnet-def) 라우팅할 수 있도록 하는 방법은 여러 가지가 있습니다. Kubernetes에 대 한 다음 옵션 중 하나에서 선택 Windows 현재:

1. 오버레이 네트워크를 설정 하려면 [Flannel](#flannel-in-vxlan-mode) 같은 CNI 플러그 인을 사용 합니다.
2. [Flannel](#flannel-in-host-gateway-mode) 같은 프로그램 경로 CNI 플러그 인을 사용 하 여 드립니다.
3. 스마트 [랙 위쪽 (ToR) 전환](#configuring-a-tor-switch) 하 여 서브넷을 라우팅합니다 구성 합니다.

> [!tip]  
> 네 번째 네트워킹 솔루션 열기 vSwitch (OvS)를 활용 하는 Windows 및 열기 가상 네트워크 (OVN)에 있습니다. 이 문서에 대 한 범위를 벗어납니다이 문서화 하지만를 설정 하려면 [다음이 지침](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) 읽을 수 있습니다.

## <a name="flannel-in-vxlan-mode"></a>Flannel vxlan 모드

Flannel vxlan 모드에서 노드 간에 패킷을 라우팅합니다 터널링 VXLAN을 사용 하는 구성 가능한 가상 오버레이 네트워크 설정에 사용할 수 있습니다.

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel에 대 한 Kubernetes 마스터 준비
[Kubernetes 마스터](./creating-a-linux-master.md) 는 클러스터의 몇 가지 사소한 알아보는 것이 좋습니다. Flannel를 사용 하는 경우 iptables 체인 IPv4 트래픽을 브리지를 사용 하는 것이 좋습니다. 이 방법은 다음 명령을 사용 하 여:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>다운로드 & Flannel 구성 ###
가장 최근 Flannel 매니페스트를 다운로드 합니다.

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Vxlan 네트워킹 백 엔드를 사용 하도록 수정 해야 하는 두 섹션 가지가 있습니다.

1. 에 `net-conf.json` 섹션에 `kube-flannel.yml`를 다시 확인:
 * 클러스터 서브넷 (예: "10.244.0.0/16")을 설정 하는 필요에 따라 합니다.
 * 백 엔드에 VNI 4096 설정
 * 백 엔드에 포트 4789 설정
2. 에 `cni-conf.json` 섹션에 `kube-flannel.yml`, 네트워크 이름이 변경 `"vxlan0"`합니다.

위의 단계를 적용 한 후에 `net-conf.json` 다음과 같이 표시 됩니다.
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
> Windows에서 Flannel와 상호 운용 하는 VNI 4096 및 Linux에서 Flannel에 대 한 포트 4789 설정 되어야 합니다. 다른 VNIs에 대 한 지원을 출시 예정입니다. [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 이러한 필드에 대 한 설명을 참조 하세요.

사용자 `cni-conf.json` 다음과 같이 표시 됩니다.
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
> 위의 옵션에 대 한 자세한 내용은 Linux 용 공식 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)및 [브리지](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) 플러그인 문서를 참조 하세요.

### <a name="launch-flannel--validate"></a>Flannel & 유효성 검사를 실행 합니다. ###
Flannel를 사용 하 여 시작 합니다.

```bash
kubectl apply -f kube-flannel.yml
```

다음으로 Flannel 포드가 Linux 기반 이므로, Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 패치를 적용 `kube-flannel-ds` DaemonSet만 Linux (하는 프로세스를 시작 Flannel "flanneld" 호스트 에이전트 Windows에서 나중에 가입 하는 경우)를 대상으로 합니다.

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 모든 노드 x86 64 기반가 아닌 경우 `-amd64` 위에 프로세서 아키텍처와 합니다.

몇 분 후 Flannel 포드 네트워크를 배포한 경우 실행을 모든 포드 표시 되어야 합니다.

```bash
kubectl get pods --all-namespaces
```

![텍스트](media/kube-master.png)

Flannel DaemonSet는 NodeSelector 있어야 `beta.kubernetes.io/os=linux` 적용 합니다.

```bash
kubectl get ds -n kube-system
```

![텍스트](media/kube-daemonset.png)

> [!tip]  
> -Ds-나머지 flannel에 대 한 * DaemonSets, 이러한 수 무시 하거나 삭제할 경우 해당 프로세서 아키텍처와 일치 하는 노드가 없으면 예약할 수 없습니다.

> [!tip]  
> 자세한 내용은 다음은 기본 클러스터 서브넷에 대 한 미리 적용 다음이 단계를 사용 하 여 v0.11.0 Flannel에 대 한 전체 [예제 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) `10.244.0.0/16`.

성공 하면 되 면 [다음 단계](#next-steps)를 계속 합니다.

## <a name="flannel-in-host-gateway-mode"></a>Flannel 호스트 게이트웨이 모드

[Flannel vxlan](#flannel-in-vxlan-mode)함께 Flannel 네트워킹에 대 한 또 다른 옵션은 *호스트 게이트웨이 모드* (호스트-gw)는 다른 노드에서 포드 서브넷 다음 홉으로 대상 노드에 호스트 주소를 사용 하 여 각 노드에서 정적 경로 프로그래밍 정확성 합니다.

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel에 대 한 Kubernetes 마스터 준비

[Kubernetes 마스터](./creating-a-linux-master.md) 는 클러스터의 몇 가지 사소한 알아보는 것이 좋습니다. Flannel를 사용 하는 경우 iptables 체인 IPv4 트래픽을 브리지를 사용 하는 것이 좋습니다. 이 방법은 다음 명령을 사용 하 여:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>다운로드 & Flannel 구성 ###
가장 최근 Flannel 매니페스트를 다운로드 합니다.

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

파일은 하나 호스트 gw 모두 Windows/Linux에서 네트워킹을 사용 하도록 설정 하려면 변경 해야 합니다.

에 `net-conf.json` 섹션에 kube flannel.yml의 여러 번 확인 합니다.
1. 사용 중인 네트워크 백 엔드 유형의 설정은 `host-gw` 대신 `vxlan`.
2. 클러스터 서브넷 (예: "10.244.0.0/16")을 설정 하는 필요에 따라 합니다.

2 단계를 적용 한 후에 `net-conf.json` 다음과 같이 표시 됩니다.
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Flannel & 유효성 검사를 실행 합니다. ###
Flannel를 사용 하 여 시작 합니다.

```bash
kubectl apply -f kube-flannel.yml
```

그런 다음 Flannel 포드는 Linux 기반으로 하므로 우리의 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 패치를 적용 `kube-flannel-ds` DaemonSet만 Linux (하는 프로세스를 시작 Flannel "flanneld" 호스트 에이전트 Windows에서 나중에 가입 하는 경우)를 대상으로 합니다.

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 모든 노드 x86 64 기반가 아닌 경우 `-amd64` 위에 원하는 프로세서 아키텍처.

몇 분 후 Flannel 포드 네트워크를 배포한 경우 실행을 모든 포드 표시 되어야 합니다.

```bash
kubectl get pods --all-namespaces
```

![텍스트](media/kube-master.png)

또한 Flannel DaemonSet 적용 NodeSelector 있어야 합니다.

```bash
kubectl get ds -n kube-system
```

![텍스트](media/kube-daemonset.png)

> [!tip]  
> -Ds-나머지 flannel에 대 한 * DaemonSets, 이러한 수 무시 하거나 삭제할 경우 해당 프로세서 아키텍처와 일치 하는 노드가 없으면 예약할 수 없습니다.

> [!tip]  
> 자세한 내용은 다음은 기본 클러스터 서브넷에 대 한 미리 적용이 2 단계를 사용 하 여 전체 [예제 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) Flannel에 대 한 v0.11.0 `10.244.0.0/16`.

성공 하면 되 면 [다음 단계](#next-steps)를 계속 합니다.

## <a name="configuring-a-tor-switch"></a>ToR 스위치 구성 ##
> [!NOTE]
> [Flannel 네트워킹 솔루션으로](#flannel-in-host-gateway-mode)선택한 경우이 섹션을 건너뛸 수 있습니다.
구성 ToR 스위치의 실제 노드 외부에서 발생합니다. 이 대 한 자세한 내용은 [공식 Kubernetes 문서](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)를 참조 하세요.


## <a name="next-steps"></a>다음 단계 ## 
이 섹션에서는 선택 네트워킹 솔루션을 구성 하는 방법을 설명한 합니다. 이제 준비가 4 단계:

> [!div class="nextstepaction"]
> [Windows 작업자에 가입](./joining-windows-workers.md)