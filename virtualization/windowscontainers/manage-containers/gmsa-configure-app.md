---
title: 그룹 관리 서비스 계정을 사용하도록 앱 구성
description: Windows 컨테이너에 gMSA(그룹 관리 서비스 계정)를 사용하도록 앱을 구성하는 방법을 설명합니다.
keywords: docker, 컨테이너, active directory, gmsa, 앱, 애플리케이션, 그룹 관리 서비스 계정, 그룹관리서비스 계정, 구성
author: rpsqrd
ms.date: 09/10/2019
ms.topic: how-to
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: fa14f2fa953373f8dfea74f822b025be0910fd54
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985267"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>gMSA를 사용하도록 앱 구성

일반적인 구성에서는 컨테이너 컴퓨터 계정이 네트워크 리소스에 인증을 시도할 때마다 사용되는 gMSA(그룹 관리 서비스 계정) 하나만 컨테이너에 제공됩니다. 즉, gMSA ID를 사용해야 하는 경우 앱을 **로컬 시스템** 또는 **네트워크 서비스**로 실행해야 합니다.

## <a name="run-an-iis-app-pool-as-network-service"></a>IIS 앱 풀을 네트워크 서비스로 실행

IIS 웹 사이트를 컨테이너에 호스팅하는 경우 앱 풀 ID를 **네트워크 서비스**로 설정하기만 하면 gMSA를 활용할 수 있습니다. 이렇게 설정하려면 Dockerfile에서 다음 명령을 추가합니다.

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

이전에 IIS 앱 풀에 정적 사용자 자격 증명을 사용했다면 해당 자격 증명을 gMSA로 대체하는 방안을 생각해 보세요. 개발, 테스트 및 프로덕션 환경 간에 gMSA를 변경할 수 있으며, IIS는 컨테이너 이미지를 변경할 필요 없이 현재 ID를 자동으로 선택합니다.

## <a name="run-a-windows-service-as-network-service"></a>Windows 서비스를 네트워크 서비스로 실행

컨테이너화 된 앱이 Windows 서비스로 실행되는 경우 Dockerfile에서 서비스가 **Network Service**로 실행되도록 설정할 수 있습니다.

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>임의의 콘솔 앱을 네트워크 서비스로 실행

IIS 또는 Service Manager에 호스팅되지 않는 일반 콘솔 앱의 경우 앱이 자동으로 gMSA 컨텍스트를 상속하도록 컨테이너를 **네트워크 서비스**로 실행하는 것이 가장 쉬운 경우가 많습니다. 이 기능은 Windows Server 버전 1709부터 사용할 수 있습니다.

기본적으로 네트워크 서비스로 실행되도록 Dockerfile에 다음 줄을 추가합니다.

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

일회성 작업으로 `docker exec`를 사용하여 컨테이너에 네트워크 서비스로 연결할 수도 있습니다. 컨테이너가 네트워크 서비스로 정상 실행되지 않는 경우 실행 중인 컨테이너에서 연결 문제를 해결할 때 특히 유용한 방법입니다.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>다음 단계

앱 구성 외에도, gMSAs를 사용하여 다음을 수행할 수 있습니다.

- [컨테이너 실행](gmsa-run-container.md)
- [컨테이너 오케스트레이션](gmsa-orchestrate-containers.md)

설정하는 동안 문제가 발생하면 [문제 해결 지침](gmsa-troubleshooting.md)에서 가능한 해결 방법을 확인하세요.
