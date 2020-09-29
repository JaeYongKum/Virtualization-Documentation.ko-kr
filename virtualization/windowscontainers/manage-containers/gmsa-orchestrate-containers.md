---
title: gMSA를 사용하여 컨테이너 오케스트레이션
description: gMSA(그룹 관리 서비스 계정)로 Windows 컨테이너를 오케스트레이션하는 방법을 설명합니다.
keywords: docker, 컨테이너, active directory, gmsa, 오케스트레이션, kubernetes, 그룹 관리 서비스 계정, 그룹관리서비스 계정
author: rpsqrd
ms.author: jgerend
ms.date: 09/10/2019
ms.topic: how-to
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 64a5e5c010922bcc5e61f41c047ac965c4f31415
ms.sourcegitcommit: 160405a16d127892b6e2897efa95680f29f0496a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/22/2020
ms.locfileid: "90990896"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>gMSA를 사용하여 컨테이너 오케스트레이션

프로덕션 환경에서는 종종 컨테이너 오케스트레이터를 사용하여 앱과 서비스를 배포하고 관리합니다. 각 오케스트레이터는 고유한 관리 패러다임을 갖고 있으며 Windows 컨테이너 플랫폼에 제공할 자격 증명 사양을 수락하는 역할을 담당합니다.

gMSA(그룹 관리 서비스 계정)를 사용하여 컨테이너를 오케스트레이션할 때 다음 사항을 확인해야 합니다.

> [!div class="checklist"]
> * gMSA로 컨테이너를 실행하도록 예약할 수 있는 모든 컨테이너 호스트는 도메인에 가입되어야 합니다.
> * 컨테이너 호스트는 컨테이너에서 사용하는 모든 gMSA의 암호를 검색할 수 있는 액세스 권한을 갖고 있어야 합니다.
> * 오케스트레이터의 처리 방식에 따라 자격 증명 사양 파일이 생성되어 오케스트레이터에 업로드되거나 모든 컨테이너 호스트에 복사됩니다.
> * 컨테이너 네트워크는 컨테이너가 Active Directory 도메인 컨트롤러와 통신하여 gMSA 티켓을 검색할 수 있도록 허용합니다.

## <a name="how-to-use-gmsa-with-service-fabric"></a>Service Fabric에서 gMSA를 사용하는 방법

애플리케이션 매니페스트에서 자격 증명 사양 위치를 지정할 때 Service Fabric은 gMSA로 Windows 컨테이너를 실행하는 것을 지원합니다. 자격 증명 사양 파일을 만들고 각 호스트 Docker 데이터 디렉터리의 하위 디렉터리 **CredentialSpecs**에 배치해야 합니다. 그래야만 Service Fabric이 파일을 찾을 수 있습니다. [CredentialSpec PowerShell 모듈](https://aka.ms/credspec)에 포함된 **Get-CredentialSpec** cmdlet을 실행하여 자격 증명 사양이 올바른 위치에 있는지 확인할 수 있습니다.

[빠른 시작: Service Fabric에 Windows 컨테이너 배포](/azure/service-fabric/service-fabric-quickstart-containers) 및 [Service Fabric에서 실행되는 Windows 컨테이너에 대한 gMSA 설정](/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)에서 애플리케이션을 구성하는 방법에 대해 자세히 알아보세요.

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Docker Swarm에서 gMSA를 사용하는 방법

Docker Swarm으로 관리되는 컨테이너에서 gMSA를 사용하려면 `--credential-spec` 매개 변수를 사용하여 [docker service create](https://docs.docker.com/engine/reference/commandline/service_create/) 명령을 실행합니다.

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Docker 서비스에서 자격 증명 사양을 사용하는 방법에 대한 자세한 내용은 [Docker Swarm 예제](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)를 참조하세요.

## <a name="how-to-use-gmsa-with-kubernetes"></a>Kubernetes에서 gMSA를 사용하는 방법

Kubernetes에서 gMSA를 사용하여 Windows 컨테이너 일정을 예약하는 기능은 Kubernetes 1.14에서 알파 기능으로 제공됩니다. 이 기능에 대한 최신 정보와 Kubernetes 배포판에서 이 기능을 테스트하는 방법에 대한 자세한 내용은 [Windows Pod 및 컨테이너에 대한 gMSA 구성](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa)을 참조하세요.

## <a name="next-steps"></a>다음 단계

컨테이너 오케스트레이션 외에도, gMSAs를 사용하여 다음을 수행할 수 있습니다.

- [앱 구성](gmsa-configure-app.md)
- [컨테이너 실행](gmsa-run-container.md)

설정하는 동안 문제가 발생하면 [문제 해결 지침](gmsa-troubleshooting.md)에서 가능한 해결 방법을 확인하세요.