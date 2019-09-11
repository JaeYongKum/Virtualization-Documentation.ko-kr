---
title: GMSA를 사용 하 여 컨테이너 실행
description: GMSA (그룹 관리 서비스 계정)를 사용 하 여 Windows 컨테이너를 실행 하는 방법
keywords: docker, 컨테이너, active directory, gmsa, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b9c0406b5fe9527d88365dabf0cfd10114c34c74
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079734"
---
# <a name="run-a-container-with-a-gmsa"></a>GMSA를 사용 하 여 컨테이너 실행

GMSA (그룹 관리 서비스 계정)를 사용 하 여 컨테이너를 실행 하려면 `--security-opt` [docker 실행](https://docs.docker.com/engine/reference/run)의 매개 변수에 자격 증명 사양 파일을 제공 합니다.

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Windows Server 2016 버전 1709 및 1803에서 컨테이너의 호스트 이름은 gMSA 짧은 이름과 일치 해야 합니다.

앞의 예제에서 gMSA SAM 계정 이름은 "webapp01" 이므로 컨테이너 호스트 이름에는 "webapp01"도 지정 되어 있습니다.

Windows Server 2019 이상에서는 hostname 필드가 필요 하지 않지만, 다른 것을 명시적으로 제공 하더라도 컨테이너는 호스트 이름 대신 gMSA 이름으로 식별 됩니다.

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

신뢰할 수 있는 DC 연결 상태 및 신뢰 확인 상태가 되지 `NERR_Success`않으면 [문제 해결 지침](gmsa-troubleshooting.md#check-the-container) 을 따라 문제를 디버그 하세요.

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

PowerShell 또는 다른 콘솔 앱을 gMSA 계정으로 열려면 정상 ContainerAdministrator (또는 ContainerUser for NanoServer) 계정 대신 네트워크 서비스 계정으로 실행 하도록 컨테이너에 요청할 수 있습니다.

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

네트워크 서비스로 실행 하는 경우 도메인 컨트롤러에서 SYSVOL에 연결 하 여 네트워크 인증을 gMSA으로 테스트할 수 있습니다.

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>다음 단계

컨테이너를 실행 하는 것 외에도 다음과 같은 작업을 수행할 수 있습니다.

- [앱 구성](gmsa-configure-app.md)
- [컨테이너 조정](gmsa-orchestrate-containers.md)

설치 하는 동안 문제가 발생 하는 경우 [문제 해결 가이드](gmsa-troubleshooting.md) 에서 가능한 해결 방법을 확인 하세요.
