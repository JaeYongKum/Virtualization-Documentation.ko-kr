---
title: GMSA를 사용 하 여 컨테이너 조정
description: GMSA (그룹 관리 서비스 계정)를 사용 하 여 Windows 컨테이너를 조정 하는 방법
keywords: docker, 컨테이너, active directory, gmsa, 오케스트레이션, kubernetes, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b4dac775dc7a4ee6375f0d803e921527e66aae5b
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079742"
---
## <a name="orchestrate-containers-with-a-gmsa"></a>GMSA를 사용 하 여 컨테이너 조정

프로덕션 환경에서는 컨테이너 조정자를 사용 하 여 앱과 서비스를 배포 하 고 관리 하는 경우가 많습니다. 각 orchestrator에는 고유한 관리 패러다임과 Windows 컨테이너 플랫폼에 제공 하는 자격 증명 사양을 수락 하는 책임이 있습니다.

그룹 관리 서비스 계정 (gMSAs)을 사용 하 여 컨테이너를 오케스트레이션 하는 경우 다음을 확인 합니다.

> [!div class="checklist"]
> * Gdr을 도메인에 가입 하는 것 처럼 컨테이너를 실행 하도록 예약할 수 있는 모든 컨테이너 호스트
> * 컨테이너 호스트가 모든 gMSAs에 대 한 암호를 검색 하는 데 사용할 수 있는 액세스 권한이 있습니다.
> * 자격 증명 사양 파일은 orchestrator에 대 한 생성 및 업로드, 그리고이를 처리 하는 방법에 따라 모든 컨테이너 호스트에 복사 됩니다.
> * 컨테이너 네트워크를 통해 컨테이너는 Active Directory 도메인 컨트롤러와 통신 하 여 gMSA 티켓을 검색할 수 있습니다.

### <a name="how-to-use-gmsa-with-service-fabric"></a>서비스 패브릭에서 gMSA를 사용 하는 방법

서비스 패브릭에는 응용 프로그램 매니페스트에서 자격 증명 사양 위치를 지정 하는 경우 gMSA으로 Windows 컨테이너를 실행할 수 있도록 지원 합니다. 서비스 패브릭에서 찾을 수 있도록 자격 증명 사양 파일을 만들고 각 호스트에 Docker 데이터 디렉터리의 **Credentialspecs** 하위 디렉터리에 배치 해야 합니다. 자격 증명 사양이 올바른 위치에 있는지 확인 하기 위해 [Credentialspec PowerShell 모듈](https://aka.ms/credspec)의 일부인 **Get credentialspec** cmdlet을 실행할 수 있습니다.

응용 프로그램을 구성 하는 방법에 대 한 자세한 내용은 [빠른 시작: Service fabric에 windows 컨테이너 배포](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) 및 [서비스 패브릭에서 실행 되는 windows 컨테이너에 대 한 gMSA 설정](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) 을 참조 하세요.

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Docker Swarm와 함께 gMSA를 사용 하는 방법

Docker Swarm에서 관리 하는 컨테이너에 gMSA를 사용 하려면 `--credential-spec` 매개 변수를 사용 하 여 [Docker 서비스 만들기](https://docs.docker.com/engine/reference/commandline/service_create/) 명령을 실행 합니다.

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Docker 서비스에서 자격 증명 사양을 사용 하는 방법에 대 한 자세한 내용은 [Docker Swarm 예제](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) 를 참조 하세요.

### <a name="how-to-use-gmsa-with-kubernetes"></a>Kubernetes에서 gMSA를 사용 하는 방법

Kubernetes에서의 gMSAs에서 Windows 컨테이너 예약에 대 한 지원은 Kubernetes 1.14의 alpha 기능으로 제공 됩니다. 이 기능에 대 한 최신 정보 및 Kubernetes 배포에서 테스트 하는 방법에 대 한 자세한 내용은 [Windows pods 및 컨테이너](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) 에 대 한 gMSA 구성을 참조 하세요.

## <a name="next-steps"></a>다음 단계

오케스트레이션 컨테이너 외에도 다음과 같은 경우 gMSAs를 사용할 수 있습니다.

- [앱 구성](gmsa-configure-app.md)
- [컨테이너 실행](gmsa-run-container.md)

설치 하는 동안 문제가 발생 하는 경우 [문제 해결 가이드](gmsa-troubleshooting.md) 에서 가능한 해결 방법을 확인 하세요.
