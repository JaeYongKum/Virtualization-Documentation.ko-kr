---
title: 네트워크 토폴로지
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows 및 Linux에서 네트워크 토폴로지를 지원 합니다.
keywords: kubernetes, 1.12, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: bcbd7b530b58b663305ea5d8b84a75eaf971f997
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948062"
---
# <a name="network-solutions"></a>네트워크 솔루션 #

[Kubernetes 마스터 노드를 설정](./creating-a-linux-master.md) 했으면 네트워킹 솔루션 선택 준비가 되었습니다. 노드 간에 가상 [클러스터 서브넷](./getting-started-kubernetes-windows.md#cluster-subnet-def) 라우팅할 수 있도록 하는 방법은 여러 가지가 있습니다. Kubernetes에 대 한 다음 옵션 중 하나에서 선택 Windows 현재:

1. 설치 경로에 [Flannel](network-topologies.md#flannel-in-host-gateway-mode) 같은 타사 CNI 플러그를 사용 하 여 드립니다.
1. 스마트 [랙 위쪽 (ToR) 전환](network-topologies.md#configuring-a-tor-switch) 하 여 서브넷을 라우팅합니다 구성 합니다.

> [!tip]  
> 세 번째 네트워킹 솔루션 열기 vSwitch (OvS)를 활용 하는 Windows 및 열기 가상 네트워크 (OVN)에 있습니다. 이 문서에 대 한 범위를 벗어나면는이 문서화 하지만를 설정 하려면 [다음이 지침](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) 읽을 수 있습니다.

## <a name="flannel-in-host-gateway-mode"></a>Flannel 호스트 게이트웨이 모드

Flannel 네트워킹에 대해 사용 가능한 옵션 중 하나는 *호스트 게이트웨이 모드* (호스트-gw)는 모든 노드에서 포드 서브넷 간 정적 경로 구성 해야 합니다.
> [!NOTE]  
> *오버레이* 네트워킹 모드는 VXLAN 캡슐화를 사용 하 고 지금 바로 개발 중인 Flannel에 차이가 있습니다. 이 공간을 확인 하세요.

### <a name="prepare-kubernetes-master-for-flannel"></a>Flannel에 대 한 Kubernetes 마스터 준비

[Kubernetes 마스터](./creating-a-linux-master.md) 우리의 클러스터의 몇 가지 사소한 준비 것이 좋습니다. Flannel를 사용 하 여 때 iptables 체인 IPv4 트래픽을 브리지를 사용 하도록 설정 하는 것이 좋습니다. 이 수행할 수 있습니다 다음 명령을 사용 합니다.

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>다운로드 및 Flannel 구성 ###
가장 최근 Flannel 매니페스트를 다운로드 합니다.

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

두 가지 방법으로 호스트 gw 모두 Windows/Linux에서 네트워킹을 사용 하도록 설정 하기 위해 수행 해야 합니다.

에 `net-conf.json` 섹션을 확인 하면 kube flannel.yml:
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

### <a name="launch-flannel--validate"></a>Flannel 시작 및 유효성 검사 ###
Flannel 사용 하 여 시작 합니다.

```bash
kubectl apply -f kube-flannel.yml
```

그런 다음 Flannel 포드는 Linux 기반으로 하므로 우리의 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 패치를 적용 `kube-flannel-ds` DaemonSet만 Linux (하는 프로세스를 시작 Flannel "flanneld" 호스트 에이전트 Windows에서 나중에 가입 하는 경우)를 대상으로 합니다.

```
kubectl patch ds/kube-flannel-ds --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

몇 분 후 모든 포드가 Flannel 포드 네트워크를 배포한 경우 실행 중으로 표시 되어야 합니다.

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
> 자세한 내용은 다음은 기본 클러스터 서브넷에 대 한 미리 적용이 2 단계를 사용 하 여 전체 [예제 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) Flannel에 대 한 v0.9.1 `10.244.0.0/16`.

## <a name="configuring-a-tor-switch"></a>ToR 스위치를 구성합니다. ##
> [!NOTE]
> [Flannel 네트워킹 솔루션으로](#flannel-in-host-gateway-mode)선택한 경우이 섹션을 건너뛸 수 있습니다.
구성 ToR 스위치의 실제 노드 외부에서 발생합니다. 이 대 한 자세한 내용은 [공식 Kubernetes 문서](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)를 참조 하세요.


## <a name="next-steps"></a>다음 단계 ## 
이 섹션에서는 네트워킹 솔루션을 선택 하는 방법을 설명 했습니다. 이제 4 단계에 대 한 준비가:

> [!div class="nextstepaction"]
> [Windows 작업자 조인](./joining-windows-workers.md)