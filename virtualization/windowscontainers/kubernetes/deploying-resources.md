---
title: Linux 노드 조인
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes resoureces를 Kubernetes 클러스터에 배포 합니다.
keywords: kubernetes, 1.14, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909963"
---
# <a name="deploying-kubernetes-resources"></a>Kubernetes 리소스 배포 #
마스터 1 개 이상으로 구성 된 Kubernetes 클러스터가 있다고 가정 하면 Kubernetes 리소스를 배포할 준비가 된 것입니다.
> [!TIP] 
> Windows에서 현재 지원 되는 Kubernetes 리소스는 무엇 인가요? 자세한 내용은 [공식적으로 지원 되는 기능](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) 및 [Windows 로드맵 Kubernetes](https://github.com/orgs/kubernetes/projects/8) 을 참조 하세요.


## <a name="running-a-sample-service"></a>샘플 서비스 실행 ##
클러스터에 성공적으로 가입되고 네트워크가 적절히 구성되었는지 확인하기 위해 매우 간단한 [PowerShell 기반 웹 서비스](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)를 배포할 것입니다.

이렇게 하기 전에 모든 노드가 정상 인지 확인 하는 것이 좋습니다.
```bash
kubectl get nodes
```

모든 항목이 양호 하면 다음 서비스를 다운로드 하 여 실행할 수 있습니다.
> [!Important] 
> `kubectl apply`하기 전에 샘플 파일의 `microsoft/windowsservercore` 이미지를 [노드에서 실행할 수 있는 컨테이너 이미지로](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)두 번 확인/수정 해야 합니다.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

그러면 배포 및 서비스가 만들어집니다. 마지막 조사식 명령은 pod을 무기한 쿼리하여 해당 상태를 추적 합니다. 관찰이 완료 되 면 `Ctrl+C`를 눌러 `watch` 명령을 종료 하면 됩니다.

모든 작업이 제대로 진행되면 다음을 수행할 수 있습니다.

  - Windows 노드의 `docker ps` 명령에서 pod 당 2 개의 컨테이너를 참조 하세요.
  - Linux 마스터의 `kubectl get pods` 명령 아래에서 포드 2개를 확인합니다.
  - Linux 마스터에서 포트 80의 *pod* ip에 `curl` 웹 서버 응답을 가져옵니다. 네트워크를 통해 통신 하는 적절 한 노드를 보여 줍니다.
  - `docker exec`를 통해 *포드 간* Ping(Windows 노드가 여러 개 있는 경우 호스트 간 포함). 이는 적절한 포드 간 통신을 나타냅니다.
  - Linux 마스터와 개별 pod 가상 *서비스 IP* (`kubectl get services`아래에 표시 됨)를 `curl` 합니다. 이는 적절 한 서비스를 pod 통신을 보여 줍니다.
  - Kubernetes [기본 DNS 접미사](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)를 사용 하 여 *서비스 이름을* `curl` 하 여 적절 한 서비스 검색을 보여 줍니다.
  - Linux 마스터 또는 클러스터 외부의 컴퓨터에서 *Nodeport* 를 `curl` 합니다. 인바운드 연결을 보여 줍니다.
  - pod 내부에서 외부 Ip를 `curl` 합니다. 아웃 바운드 연결을 보여 줍니다.

> [!Note]  
> Windows *컨테이너 호스트* 는 예약 된 서비스에서 서비스 IP에 액세스할 수 **없습니다** . 이는 Windows Server에 대 한 이후 버전에서 향상 될 [알려진 플랫폼 제한](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) 입니다. 그러나 Windows *pod* **는** 서비스 IP에 액세스할 수 있습니다.

## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 Windows 노드에서 Kubernetes 리소스를 예약 하는 방법을 살펴보았습니다. 이 가이드를 마칩니다. 문제가 발생 하는 경우 문제 해결 섹션을 검토 하세요.

> [!div class="nextstepaction"]
> [문제 해결](./common-problems.md)

그렇지 않으면 Windows 서비스로 Kubernetes 구성 요소를 실행 하는 데 관심이 있을 수도 있습니다.
> [!div class="nextstepaction"]
> [Windows 서비스](./kube-windows-services.md)
