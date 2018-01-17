---
title: "Kubernetes 문제 해결"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "Kubernetes를 배포하고 Windows 노드를 가입할 때 발생하는 일반적인 문제에 대한 해결 방법입니다."
keywords: "kubernetes, 1.9, linux, 컴파일"
ms.openlocfilehash: 73b44ffd12fba58ac4ef38352c012061a6817945
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/05/2017
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes 문제 해결 #
이 페이지에서는 Kubernetes 설정, 네트워킹 및 배포 관련 몇 가지 일반적인 문제를 안내합니다.

> [!tip]
> [Microsoft의 문서 저장소](https://github.com/MicrosoftDocs/Virtualization-Documentation/)에서 PR을 제기하여 FAQ 항목을 제안해 주세요.


## <a name="common-deployment-errors"></a>일반적인 배포 오류 ##
Kubernetes 마스터 디버깅은 세 가지 주요 범주로 나누어집니다(발생 가능성 순서).

  - Kubernetes 시스템 컨테이너에 문제가 있습니다.
  - `kubelet`이 실행되는 방식에 문제가 있습니다.
  - 시스템에 문제가 있습니다.


`kubectl get pods -n kube-system`을 실행하여 Kubernetes에 의해 생성되고 있는 포드를 확인할 수 있습니다. 이를 통해 어떠한 것이 충돌되었으며 제대로 시작하지 않는지 파악할 수 있습니다. 그런 다음 `docker ps -a`를 실행하여 이러한 포드를 백업하는 모든 원시 컨테이너를 볼 수 있습니다. 마지막으로 문제를 일으킨 것으로 의심되는 컨테이너에서 `docker logs [ID]`를 실행하여 프로세스의 원시 출력을 볼 수 있습니다.


### <a name="permission-denied-errors"></a>_"권한이 거부되었습니다"_ 오류 ###
스크립트에 실행 권한이 있는지 확인합니다.

```bash
chmod +x [script name]
```

또한 특정 스크립트는 `kubelet`과 같은 관리자 권한으로 실행해야 하며 `sudo` 접두사가 붙어야 합니다.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>`https://[address]:[port]`에서 API 서버에 연결할 수 없습니다. ###
대개 이 오류는 인증서에 문제가 있음을 의미합니다. 구성 파일을 올바르게 생성했는지, 그 안의 IP 주소가 호스트의 IP 주소와 일치하는지, 그리고 이를 API 서버에 의해 마운트된 디렉터리에 복사했는지 확인합니다.

[우리의 지침](./creating-a-linux-master)을 따른 경우 `~/kube/kubelet/`에 있습니다. 그렇지 않은 경우 마운트 지점을 확인하기 위해 API 서버의 매니페스트 파일을 참조하세요.


## <a name="common-networking-errors"></a>일반적인 네트워킹 오류 ##
네트워크 또는 호스트에 특정 유형의 노드 간 통신을 방지하는 추가 제한 사항이 있을 수 있습니다. 다음을 확인합니다.

  - 포드에서 가져온 것처럼 보이는 트래픽이 허용되었는지 여부
  - 웹 서비스를 배포하는 경우 HTTP 트래픽이 허용되었는지 여부
  - ICMP 패킷이 손실되지 않았는지 여부


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>일반적인 Windows 오류 ##


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>내 Windows 포드가 Linux 마스터에 액세스할 수 없으며 그 반대도 마찬가지입니다. ###
Hyper-V 가상 컴퓨터를 사용하는 경우 네트워크 어댑터에서 MAC 스푸핑이 활성화되어 있는지 확인합니다.


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>내 Windows 노드가 서비스 IP를 사용하여 내 서비스에 액세스할 수 없습니다. ###
이는 Windows에서 현재 네트워킹 스택의 알려진 제한 사항입니다.
