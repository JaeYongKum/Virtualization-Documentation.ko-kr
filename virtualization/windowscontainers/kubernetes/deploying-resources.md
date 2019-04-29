---
title: Linux 노드를 가입
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes resoureces 혼합 OS Kubernetes 클러스터에 배포 됩니다.
keywords: kubernetes, 1.13, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 7d2f1dd789a96a3ee4898ef196f872e574d6321f
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574904"
---
# <a name="deploying-kubernetes-resources"></a>Kubernetes 리소스 배포 #
최소한 1 마스터 및 1 작업자로 구성 된 Kubernetes 클러스터 했 고, 준비가 Kubernetes 리소스를 배포 합니다.
> [!TIP] 
> 이름을 바꾼 Windows에서 현재 지원 되는 Kubernetes 리소스? 자세한 내용은 [공식적으로 지원 되는 기능](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) 및 [로드맵 Windows에서 Kubernetes](https://trello.com/b/rjTqrwjl/windows-k8s-roadmap) 를 참조 하십시오.


## <a name="running-a-sample-service"></a>샘플 서비스 실행 ##
클러스터에 성공적으로 가입되고 네트워크가 적절히 구성되었는지 확인하기 위해 매우 간단한 [PowerShell 기반 웹 서비스](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)를 배포할 것입니다.

이 작업을 수행 하기 전에 것은 항상 우리의 모든 노드 올바르게 작동 하는지 확인 하는 것이 좋습니다.
```bash
kubectl get nodes
```

모든 멋지게 다운로드 하 고 다음 서비스를 실행할 수 있습니다.
> [!Important] 
> 하기 전에 `kubectl apply`을 확인을 두-check/수정 확인 합니다 `microsoft/windowsservercore` [를 노드 하 여 실행할 수 있는 컨테이너 이미지](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)에 샘플 파일의 이미지!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

이렇게 하면 배포 및 서비스가 만들어집니다. 마지막 감시 명령; 이들의 상태를 추적 하는 무기한으로 포드를 쿼리 합니다. 누르기만 하면 `Ctrl+C` 종료 하 고 `watch` 관찰을 완료 했으면 명령 합니다.

모든 작업이 제대로 진행되면 다음을 수행할 수 있습니다.

  - 아래에서 포드 당 2 컨테이너 확인 `docker ps` 명령에 Windows 노드
  - Linux 마스터의 `kubectl get pods` 명령 아래에서 포드 2개를 확인합니다.
  - `curl` 포트 80에 있는 *포드* IP에서 Linux 마스터로부터 웹 서버 응답을 가져옵니다. 이는 네트워크에 걸친 적절한 노드에서 포드 통신을 보여줍니다.
  - `docker exec`를 통해 *포드 간* Ping(Windows 노드가 여러 개 있는 경우 호스트 간 포함). 이는 적절한 포드 간 통신을 나타냅니다.
  - `curl` 가상 *서비스 IP* (아래에 표시 `kubectl get services`) Linux 마스터 및 개별 포드에서; 적절 한 서비스 포드 통신을 보여줍니다.
  - `curl` Kubernetes [기본 DNS 접미사](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)를 적절 한 서비스 검색을 보여 주는 *서비스 이름* 입니다.
  - `curl` Linux 마스터 또는 클러스터; 외부 컴퓨터에서 *NodePort* 이 인바운드 연결을 보여 줍니다.
  - `curl` 포드; 내에서 외부 Ip 아웃 바운드 연결을 보여줍니다.

> [!Note]  
> Windows *컨테이너 호스트* 는 **하지** 서비스 IP에 예약 된 서비스에서 액세스할 수 있습니다. 이는 [알려진 플랫폼 제한](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) 하며, Windows server 이후 버전에서 개선 될입니다. 하지만 Windows *포드* **는** 서비스 IP에 액세스할 수 없습니다.

### <a name="port-mapping"></a>포트 매핑 ### 
또한 노드의 포트를 매핑하여 각 노드를 통해 포드에 호스팅되는 서비스에 액세스할 수도 있습니다. 이 기능을 보여주기 위해 노드의 포트 4444를 포드의 포트 80에 매핑하는 [다른 샘플 YAML](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml)도 볼 수 있습니다. 배포하려면 전과 동일한 단계를 따르세요.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

이제 포트 4444의 *노드* IP에 대해 `curl`을 수행할 수 있습니다. 이는 일 대 일 매핑을 적용해야 하므로 노드당 단일 포드로의 확장이 제한된다는 점에 유의하세요.


## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 Windows 노드에서 Kubernetes 리소스를 예약 하는 방법을 설명한 합니다. 이 가이드를 완료 했습니다. 문제가 있는 경우, 문제 해결 섹션을 참조 하십시오.

> [!div class="nextstepaction"]
> [문제 해결](./common-problems.md)

그렇지 않으면 수도 있습니다 Kubernetes 구성 요소가 Windows 서비스로 실행 되는 데 관심이 있습니다.
> [!div class="nextstepaction"]
> [Windows 서비스](./kube-windows-services.md)