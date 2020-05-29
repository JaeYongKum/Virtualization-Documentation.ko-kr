---
title: gMSA를 사용하여 컨테이너 실행
description: gMSA(그룹 관리 서비스 계정)로 Windows 컨테이너를 실행하는 방법을 설명합니다.
keywords: docker, 컨테이너, active directory, gmsa, 그룹 관리 서비스 계정, 그룹관리서비스 계정
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b997cf79cdf7f1782b6299198859714563c45f8c
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853937"
---
# <a name="run-a-container-with-a-gmsa"></a>gMSA를 사용하여 컨테이너 실행

gMSA(그룹 관리 서비스 계정)를 사용하여 컨테이너를 실행하려면 [docker run](https://docs.docker.com/engine/reference/run)의 `--security-opt` 매개 변수에 자격 증명 사양 파일을 제공합니다.

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Windows Server 2016 버전 1709 및 1803에서는 컨테이너의 호스트 이름이 gMSA 짧은 이름과 일치해야 합니다.

이전 예제에서 gMSA SAM 계정 이름이 "webapp01"이므로 컨테이너 호스트 이름 역시 "webapp01"입니다.

Windows Server 2019 이상에서는 호스트 이름 필드가 필수는 아니지만, 사용자가 다른 이름을 명시적으로 제공하더라도 컨테이너는 호스트 이름 대신 gMSA 이름으로 자신을 식별합니다.

gMSA가 올바르게 작동하는지 확인하려면 컨테이너에서 다음 cmdlet을 실행합니다.

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

신뢰할 수 있는 DC 연결 상태 및 신뢰 확인 상태가 `NERR_Success`가 아니라면 [문제 해결 지침](gmsa-troubleshooting.md#check-the-container)에 따라 문제를 디버그합니다.

컨테이너 내에서 다음 명령을 실행하고 클라이언트 이름을 검사하여 gMSA ID를 확인할 수 있습니다.

```powershell
PS C:\> klist get webapp01

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

PowerShell 또는 다른 콘솔 앱을 gMSA 계정으로 열려면 일반 ContainerAdministrator(또는 NanoServer의 ContainerUser) 계정 대신 네트워크 서비스 계정에서 실행하도록 컨테이너에 요청하면 됩니다.

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

네트워크 서비스로 실행하는 경우 도메인 컨트롤러에서 SYSVOL에 연결을 시도하여 gMSA로 네트워크 인증을 테스트할 수 있습니다.

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>다음 단계

컨테이너를 실행하는 것 외에도, gMSAs를 사용하여 다음을 수행할 수 있습니다.

- [앱 구성](gmsa-configure-app.md)
- [컨테이너 오케스트레이션](gmsa-orchestrate-containers.md)

설정하는 동안 문제가 발생하면 [문제 해결 지침](gmsa-troubleshooting.md)에서 가능한 해결 방법을 확인하세요.
