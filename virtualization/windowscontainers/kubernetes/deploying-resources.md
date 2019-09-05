---
title: Linux 노드 가입
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 혼합 OS Kubernetes 클러스터에 Kubernetes resoureces를 배포 합니다.
keywords: kubernetes, 1.14, windows, 시작 하기
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: d252f356a3de98f224e1550536810dfc75345303
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/04/2019
ms.locfileid: "10069947"
---
# <a name="deploying-kubernetes-resources"></a>Kubernetes 리소스 배포 #
1 개 이상의 마스터로 구성 된 Kubernetes 클러스터가 있고 작업자 1 개 이상을 사용 하는 경우 Kubernetes 리소스를 배포할 준비가 된 것으로 가정 합니다.
> [!TIP] 
> Windows에서 현재 지원 되는 Kubernetes 리소스에 대해 궁금 하세요. 자세한 내용은 [공식적으로 지원 되는 기능](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) 및 [Windows 로드맵에 대 한 Kubernetes](https://github.com/orgs/kubernetes/projects/8) 을 참조 하세요.


## <a name="running-a-sample-service"></a>샘플 서비스 실행 ##
클러스터에 성공적으로 가입되고 네트워크가 적절히 구성되었는지 확인하기 위해 매우 간단한 [PowerShell 기반 웹 서비스](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)를 배포할 것입니다.

이 작업을 수행 하기 전에 모든 노드가 정상 인지 확인 하는 것이 항상 좋습니다.
```bash
kubectl get nodes
```

모든 항목이 보이면 다음 서비스를 다운로드 하 여 실행할 수 있습니다.
> [!Important] 
> 이전 `kubectl apply`에는 샘플 파일의 `microsoft/windowsservercore` 이미지를 [노드에서 실행할 수 있는 컨테이너 이미지로](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)두 번 확인/수정 해야 합니다.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

이렇게 하면 배포 및 서비스가 만들어집니다. 마지막 조사식 명령은 pods 무한정 쿼리 하 여 상태를 추적 합니다. 클릭 `Ctrl+C` 하기만 하면 `watch` 명령이 종료 됩니다.

모든 작업이 제대로 진행되면 다음을 수행할 수 있습니다.

  - Windows 노드의 명령 아래에서 `docker ps` pod 당 2 컨테이너를 참조 하세요.
  - Linux 마스터의 `kubectl get pods` 명령 아래에서 포드 2개를 확인합니다.
  - `curl` 포트 80에 있는 *포드* IP에서 Linux 마스터로부터 웹 서버 응답을 가져옵니다. 이는 네트워크에 걸친 적절한 노드에서 포드 통신을 보여줍니다.
  - `docker exec`를 통해 *포드 간* Ping(Windows 노드가 여러 개 있는 경우 호스트 간 포함). 이는 적절한 포드 간 통신을 나타냅니다.
  - `curl` Linux 마스터 및 개별 pods의 가상 `kubectl get services` *서비스 IP* (아래에 표시 됨) 이는 현재 pod 통신에 대 한 적절 한 서비스를 보여 줍니다.
  - `curl` Kubernetes [기본 DNS 접미사](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)를 포함 하는 *서비스 이름* 으로, 적절 한 서비스 검색을 보여 줍니다.
  - `curl` Linux 마스터의 *Nodeport* 또는 클러스터 외부의 컴퓨터 이는 인바운드 연결을 보여 줍니다.
  - `curl` pod 내부 로부터의 외부 Ip 아웃 바운드 연결을 보여 줍니다.

> [!Note]  
> Windows *컨테이너 호스트가* 서비스 **** 에 예약 된 서비스 IP에 액세스할 수 없게 됩니다. 이는 [알려진 플랫폼 제한 사항](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) 으로, 이후 버전의 Windows Server에 향상 될 것입니다. 그러나 Windows *pods* **는** 서비스 IP에 액세스할 수 있습니다.

## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 Windows 노드에서 Kubernetes 리소스를 예약 하는 방법을 설명 했습니다. 가이드를 마칩니다. 문제가 발생 하는 경우 문제 해결 섹션을 참조 하세요.

> [!div class="nextstepaction"]
> [문제 해결](./common-problems.md)

그렇지 않으면 Windows 서비스로 Kubernetes 구성 요소를 실행 하는 데에도 관심이 있을 수 있습니다.
> [!div class="nextstepaction"]
> [Windows 서비스](./kube-windows-services.md)
