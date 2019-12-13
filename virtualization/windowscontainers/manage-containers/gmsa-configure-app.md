---
title: 그룹 관리 서비스 계정을 사용 하도록 앱 구성
description: Windows 컨테이너에 대해 gMSAs (그룹 관리 서비스 계정)를 사용 하도록 앱을 구성 하는 방법
keywords: docker, 컨테이너, active directory, gmsa, 앱, 응용 프로그램, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정, 구성
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909793"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>GMSA를 사용 하도록 앱 구성

일반적인 구성에서 컨테이너는 컨테이너 컴퓨터 계정이 네트워크 리소스에 대 한 인증을 시도할 때마다 사용 되는 하나의 gMSA (그룹 관리 서비스 계정)만 지정 합니다. 즉, gMSA id를 사용 해야 하는 경우 앱을 **로컬 시스템** 또는 **네트워크 서비스로** 실행 해야 합니다.

## <a name="run-an-iis-app-pool-as-network-service"></a>네트워크 서비스로 IIS 응용 프로그램 풀 실행

컨테이너에서 IIS 웹 사이트를 호스팅하는 경우 gMSA를 활용 하려면 앱 풀 id를 **Network Service**로 설정 해야 합니다. 다음 명령을 추가 하 여 Dockerfile에서이 작업을 수행할 수 있습니다.

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

IIS 응용 프로그램 풀에 대 한 정적 사용자 자격 증명을 이전에 사용한 경우 해당 자격 증명에 대 한 대체로 gMSA을 고려 합니다. 개발, 테스트 및 프로덕션 환경 간에 gMSA를 변경할 수 있으며, IIS는 컨테이너 이미지를 변경 하지 않고 현재 id를 자동으로 선택 합니다.

## <a name="run-a-windows-service-as-network-service"></a>네트워크 서비스로 Windows 서비스 실행

컨테이너 화 된 앱이 Windows 서비스로 실행 되는 경우 Dockerfile에서 **네트워크 서비스로** 실행 되도록 서비스를 설정할 수 있습니다.

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>네트워크 서비스로 임의의 콘솔 앱 실행

IIS 또는 Service Manager에서 호스팅되지 않는 일반 콘솔 앱의 경우, 앱이 gMSA 컨텍스트를 자동으로 상속 하도록 컨테이너를 **네트워크 서비스로** 실행 하는 것이 가장 쉬운 경우가 많습니다. 이 기능은 Windows Server 버전 1709에서 사용할 수 있습니다.

Dockerfile에 다음 줄을 추가 하 여 기본적으로 네트워크 서비스로 실행 되도록 합니다.

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

`docker exec`를 사용 하 여 한 번에 네트워크 서비스로 컨테이너에 연결할 수도 있습니다. 컨테이너를 일반적으로 네트워크 서비스로 실행 하지 않는 경우 실행 중인 컨테이너에서 연결 문제를 해결 하는 경우 특히 유용 합니다.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>다음 단계

앱을 구성 하는 것 외에도 gMSAs를 사용 하 여 다음을 수행할 수 있습니다.

- [컨테이너 실행](gmsa-run-container.md)
- [컨테이너 오케스트레이션](gmsa-orchestrate-containers.md)

설치 하는 동안 문제가 발생 하는 경우 [문제 해결 가이드](gmsa-troubleshooting.md) 에서 가능한 해결 방법을 확인 하세요.
