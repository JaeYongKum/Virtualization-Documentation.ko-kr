---
title: GMSA를 사용 하 여 컨테이너 오케스트레이션
description: GMSA (그룹 관리 서비스 계정)를 사용 하 여 Windows 컨테이너를 오케스트레이션 하는 방법입니다.
keywords: docker, 컨테이너, active directory, gmsa, 오케스트레이션, kubernetes, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910263"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>GMSA를 사용 하 여 컨테이너 오케스트레이션

프로덕션 환경에서는 종종 컨테이너 orchestrator를 사용 하 여 앱과 서비스를 배포 하 고 관리 합니다. 각 orchestrator에는 고유한 관리 패러다임이 있으며 Windows 컨테이너 플랫폼에 제공 하기 위해 자격 증명 사양을 수락 해야 합니다.

GMSAs (그룹 관리 서비스 계정)를 사용 하 여 컨테이너를 오케스트레이션 하는 경우 다음을 확인 합니다.

> [!div class="checklist"]
> * GMSAs로 컨테이너를 실행 하도록 예약할 수 있는 모든 컨테이너 호스트는 도메인에 가입 되어 있습니다.
> * 컨테이너 호스트는 컨테이너에서 사용 되는 모든 gMSAs에 대 한 암호를 검색 하기 위한 액세스 권한이 있습니다.
> * Orchestrator에서이를 처리 하는 방법에 따라 자격 증명 사양 파일이 생성 되어 오 케 스트레이 터에 업로드 되거나 모든 컨테이너 호스트에 복사 됩니다.
> * 컨테이너 네트워크는 컨테이너가 Active Directory 도메인 컨트롤러와 통신 하 여 gMSA 티켓을 검색할 수 있도록 허용 합니다.

## <a name="how-to-use-gmsa-with-service-fabric"></a>Service Fabric와 함께 gMSA를 사용 하는 방법

응용 프로그램 매니페스트에서 자격 증명 사양 위치를 지정 하는 경우 gMSA를 사용 하 여 Windows 컨테이너를 실행 하는 Service Fabric 지원 됩니다. 자격 증명 사양 파일을 만들고 각 호스트에서 Docker 데이터 디렉터리의 **credentialspecs** 하위 디렉터리에 배치 하 여 Service Fabric에서 찾을 수 있도록 해야 합니다. [Credentialspec PowerShell 모듈](https://aka.ms/credspec)의 일부인 **Get credentialspec** cmdlet을 실행 하 여 자격 증명 사양이 올바른 위치에 있는지 확인할 수 있습니다.

응용 프로그램을 구성 하는 방법에 대 한 자세한 내용은 [빠른 시작: windows 컨테이너를 Service Fabric에 배포](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) 및 [Service Fabric에서 실행 되는 windows 컨테이너에 대 한 gMSA 설정](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) 을 참조 하세요.

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Docker Swarm에서 gMSA를 사용 하는 방법

Docker Swarm에서 관리 하는 컨테이너에 gMSA를 사용 하려면 `--credential-spec` 매개 변수를 사용 하 여 [docker service create](https://docs.docker.com/engine/reference/commandline/service_create/) 명령을 실행 합니다.

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Docker 서비스에서 자격 증명 사양을 사용 하는 방법에 대 한 자세한 내용은 [Docker Swarm 예제](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) 를 참조 하세요.

## <a name="how-to-use-gmsa-with-kubernetes"></a>Kubernetes와 함께 gMSA를 사용 하는 방법

Kubernetes에서 gMSAs를 사용 하 여 Windows 컨테이너 일정 예약 지원은 Kubernetes 1.14에서 알파 기능으로 사용할 수 있습니다. 이 기능에 대 한 최신 정보와 Kubernetes 배포에서 테스트 하는 방법에 대 한 자세한 내용은 [Windows pod 및 컨테이너에 대 한 GMSA 구성](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) 을 참조 하세요.

## <a name="next-steps"></a>다음 단계

오케스트레이션 컨테이너 외에도 gMSAs를 사용 하 여 다음을 수행할 수 있습니다.

- [앱 구성](gmsa-configure-app.md)
- [컨테이너 실행](gmsa-run-container.md)

설치 하는 동안 문제가 발생 하는 경우 [문제 해결 가이드](gmsa-troubleshooting.md) 에서 가능한 해결 방법을 확인 하세요.
