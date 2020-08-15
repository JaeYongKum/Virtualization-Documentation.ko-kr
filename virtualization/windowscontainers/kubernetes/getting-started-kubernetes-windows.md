---
title: Windows의 Kubernetes
author: daschott
ms.author: daschott
ms.date: 08/13/2020
ms.topic: overview
description: Windows에서 Kubernetes를 시작 합니다.
keywords: kubernetes, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e5e468d3c0092752e13f0dbbb4d9c87bd0328e9
ms.sourcegitcommit: aa139e6e77a27b8afef903fee5c7ef338e1c79d4
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/15/2020
ms.locfileid: "88251546"
---
# <a name="kubernetes-on-windows"></a>Windows의 Kubernetes

> [!TIP]
> 현재 Windows에서 지원 되는 Kubernetes 기능에 대 한 자세한 내용이 있나요? 자세한 내용은 [공식적으로 지원 되는 기능](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) 및 [Windows의 Kubernetes](https://github.com/orgs/kubernetes/projects/8) 를 참조 하세요.

이 페이지는 Windows에서 Kubernetes 시작에 대 한 개요 역할을 합니다.


### <a name="try-out-kubernetes-on-windows"></a>Windows에서 Kubernetes 사용해 보기

선택한 환경에서 Windows에 Kubernetes를 수동으로 설치 하는 방법에 대 한 지침은 [windows에서 Kubernetes 배포](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/) 를 참조 하세요.


### <a name="scheduling-windows-containers"></a>Windows 컨테이너 예약

Kubernetes에서 Windows 컨테이너를 예약 하는 방법에 대 한 모범 사례 및 권장 사항은 [Kubernetes의 windows 컨테이너 예약](https://kubernetes.io/docs/setup/production-environment/windows/user-guide-windows-containers/) 을 참조 하세요.


### <a name="deploying-kubernetes-on-windows-in-azure"></a>Azure에서 Windows에 Kubernetes 배포

[Azure Kubernetes 서비스의 Windows 컨테이너](/azure/aks/windows-container-cli) 가이드를 통해이를 쉽게 수행할 수 있습니다. 모든 Kubernetes 구성 요소를 직접 배포 하 고 관리 하려는 경우 오픈 소스 도구를 사용 하는 단계별 [연습](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md) 을 참조 하세요 `AKS-Engine` .

### <a name="troubleshooting"></a>문제 해결
알려진 문제에 대 한 제안 사항 및 해결 방법 목록은 [Kubernetes 문제 해결](./common-problems.md) 을 참조 하세요.
>[!TIP]
> 추가 자가 진단 리소스에 대 한 자세한 내용은 [여기에서 제공](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)되는 Windows에 대 한 Kubernetes 네트워킹 문제 해결 가이드도 있습니다.