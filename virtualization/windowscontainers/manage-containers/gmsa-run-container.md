---
title: GMSA를 사용 하 여 컨테이너 실행
description: GMSA (그룹 관리 서비스 계정)를 사용 하 여 Windows 컨테이너를 실행 하는 방법입니다.
keywords: docker, 컨테이너, active directory, gmsa, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 52625517748356251aa41115caebd7801ec3cdaf
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909763"
---
# <a name="run-a-container-with-a-gmsa"></a>GMSA를 사용 하 여 컨테이너 실행

GMSA (그룹 관리 서비스 계정)를 사용 하 여 컨테이너를 실행 하려면 [docker run](https://docs.docker.com/engine/reference/run)의 `--security-opt` 매개 변수에 자격 증명 사양 파일을 제공 합니다.

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Windows Server 2016 버전 1709 및 1803에서는 컨테이너의 호스트 이름이 gMSA short 이름과 일치 해야 합니다.

이전 예제에서 gMSA SAM 계정 이름은 "webapp01" 이므로 컨테이너 호스트 이름이 "webapp01"입니다.

Windows Server 2019 이상에서 호스트 이름 필드는 필요 하지 않지만 다른 항목을 명시적으로 제공 하는 경우에도 컨테이너는 호스트 이름 대신 gMSA 이름으로 자신을 식별 합니다.

GMSA 올바르게 작동 하는지 확인 하려면 컨테이너에서 다음 cmdlet을 실행 합니다.

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

트러스트 된 DC 연결 상태 및 신뢰 확인 상태가 `NERR_Success`되지 않은 경우 문제 [해결 지침](gmsa-troubleshooting.md#check-the-container) 에 따라 문제를 디버그 합니다.

다음 명령을 실행 하 고 클라이언트 이름을 확인 하 여 컨테이너 내에서 gMSA id를 확인할 수 있습니다.

```powershell
PS C:\> klist get krbtgt

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

PowerShell 또는 다른 콘솔 앱을 gMSA 계정으로 열려면 normal ContainerAdministrator (또는 ContainerUser for NanoServer) 계정 대신 네트워크 서비스 계정에서 실행 되도록 컨테이너에 요청할 수 있습니다.

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

네트워크 서비스를 실행 하는 경우 도메인 컨트롤러에서 SYSVOL에 연결을 시도 하 여 네트워크 인증을 gMSA으로 테스트할 수 있습니다.

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>다음 단계

컨테이너를 실행 하는 것 외에도 gMSAs를 사용 하 여 다음을 수행할 수 있습니다.

- [앱 구성](gmsa-configure-app.md)
- [컨테이너 오케스트레이션](gmsa-orchestrate-containers.md)

설치 하는 동안 문제가 발생 하는 경우 [문제 해결 가이드](gmsa-troubleshooting.md) 에서 가능한 해결 방법을 확인 하세요.
