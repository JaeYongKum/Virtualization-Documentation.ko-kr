---
title: 그룹 관리 서비스 계정을 사용 하도록 앱 구성
description: Windows 컨테이너에 대 한 gMSAs (그룹 관리 서비스 계정)를 사용 하도록 앱을 구성 하는 방법
keywords: docker, 컨테이너, active directory, gmsa, 앱, 응용 프로그램, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정, 구성
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 234909d7f0cb0f30ee7fbf4796dd0381bfbff89f
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079750"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>앱을 구성 하 여 gMSA 사용

일반적인 구성에서 컨테이너는 컨테이너 컴퓨터 계정이 네트워크 리소스에 대 한 인증을 시도할 때마다 사용 되는 gMSA (그룹 관리 서비스 계정) 한 개만 부여 됩니다. 즉, gMSA id를 사용 해야 하는 경우 앱이 **로컬 시스템** 또는 **네트워크 서비스로** 실행 되어야 합니다.

## <a name="run-an-iis-app-pool-as-network-service"></a>IIS 앱 풀을 네트워크 서비스로 실행

컨테이너에서 IIS 웹 사이트를 호스트 하는 경우에는 gMSA를 이용 하는 데 필요한 모든 작업은 앱 풀 id를 **네트워크 서비스**에 설정 합니다. 다음 명령을 추가 하 여 Dockerfile에서이 작업을 수행할 수 있습니다.

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

이전에 IIS 앱 풀에 대 한 정적 사용자 자격 증명을 사용한 경우 해당 자격 증명을 대체 하는 gMSA을 고려해 보세요. 개발자, 테스트 및 프로덕션 환경 간에 gMSA을 변경할 수 있으며, IIS는 컨테이너 이미지를 변경할 필요 없이 현재 id를 자동으로 선택 합니다.

## <a name="run-a-windows-service-as-network-service"></a>네트워크 서비스로 Windows 서비스 실행

Containerized 앱이 Windows 서비스로 실행 되는 경우 Dockerfile에서 **네트워크 서비스로** 실행 되도록 서비스를 설정할 수 있습니다.

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>네트워크 서비스로 임의 콘솔 앱 실행

IIS 또는 서비스 관리자에서 호스팅되지 않은 일반 콘솔 앱의 경우에는 앱이 자동으로 gMSA 컨텍스트를 상속 하도록 **네트워크 서비스로** 컨테이너를 실행 하는 것이 가장 좋습니다. 이 기능은 Windows Server 버전 1709에서 사용할 수 있습니다.

기본적으로 네트워크 서비스로 실행 되도록 다음 줄을 Dockerfile에 추가 합니다.

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

하나 이상의에서 네트워크 서비스로 컨테이너에 연결할 수도 있습니다 `docker exec`. 이는 컨테이너가 일반적으로 네트워크 서비스로 실행 되지 않는 경우 실행 중인 컨테이너의 연결 문제를 해결 하는 경우에 특히 유용 합니다.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>다음 단계

앱을 구성 하는 것 외에도 다음과 같은 작업을 수행할 수 있습니다.

- [컨테이너 실행](gmsa-run-container.md)
- [컨테이너 조정](gmsa-orchestrate-containers.md)

설치 하는 동안 문제가 발생 하는 경우 [문제 해결 가이드](gmsa-troubleshooting.md) 에서 가능한 해결 방법을 확인 하세요.
