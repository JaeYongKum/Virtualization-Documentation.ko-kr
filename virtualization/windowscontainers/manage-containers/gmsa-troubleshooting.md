---
title: GMSAs Windows 컨테이너 문제 해결
description: Windows 컨테이너에 대 한 gMSAs (그룹 관리 서비스 계정)를 해결 하는 방법을 설명 합니다.
keywords: docker, 컨테이너, active directory, gmsa, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정, 문제 해결, 문제 해결
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209853"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>GMSAs Windows 컨테이너 문제 해결

## <a name="known-issues"></a>알려진 문제

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>컨테이너 hostname은 Windows Server 2016 및 Windows 10, 버전 1709 및 1803의 gMSA 이름과 일치 해야 합니다.

Windows Server 2016, 버전 1709 또는 1803을 실행 중인 경우 컨테이너의 호스트 이름은 gMSA SAM 계정 이름과 일치 해야 합니다.

GMSA 이름과 일치 하지 않는 경우, 인바운드 NTLM 인증 요청 및 이름/SID 번역이 (ASP.NET membership role provider 등의 여러 라이브러리에서 사용 됨)가 실패 합니다. 호스트 이름 및 gMSA 이름이 일치 하지 않는 경우에도 Kerberos는 정상적으로 계속 작동 합니다.

이 제한은 지정 된 호스트 이름에 관계 없이 컨테이너에서 항상 네트워크에 gMSA 이름을 사용 하는 Windows Server 2019에서 해결 되었습니다.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>두 개 이상의 컨테이너에 gMSA를 사용 하면 Windows Server 2016 및 Windows 10, 버전 1709 및 1803의 간헐적인 오류를 발생 시킬 수 있습니다.

모든 컨테이너는 동일한 호스트 이름을 사용 해야 하므로 두 번째 문제는 windows Server 2019 및 Windows 10 버전 1809 이전 버전에 영향을 줍니다. 여러 컨테이너에 동일한 id 및 호스트 이름이 할당 되 면 두 컨테이너가 같은 도메인 컨트롤러에 동시에 통신할 때 경쟁 조건이 발생할 수 있습니다. 다른 컨테이너가 동일한 도메인 컨트롤러와 통신 하는 경우 동일한 id를 사용 하는 이전 컨테이너와의 통신을 취소 합니다. 이렇게 하면 일시적인 인증 오류가 발생할 수 있으며 컨테이너 내에서 실행할 `nltest /sc_verify:contoso.com` 때 신뢰 오류로 나타날 수 있습니다.

Windows Server 2019의 동작을 변경 하 여 여러 컨테이너에서 동일한 gMSA 동시에 사용할 수 있도록 컴퓨터 이름에서 컨테이너 id를 구분 합니다.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>GMSAs는 Windows 10 버전 1703, 1709 및 1803의 Hyper-v 격리 컨테이너와 함께 사용할 수 없습니다.

Windows 10 및 Windows Server 버전 1703, 1709 및 1803에서 Hyper-v 격리 컨테이너와 함께 gMSA를 사용 하려고 하면 컨테이너 초기화가 중단 되거나 실패할 수 있습니다.

이 버그는 Windows Server 2019 및 Windows 10, 버전 1809에서 수정 되었습니다. 또한 Windows Server 2016 및 Windows 10, 버전 1607에서 gMSAs를 사용 하 여 Hyper-v 격리 컨테이너를 실행할 수 있습니다.

## <a name="general-troubleshooting-guidance"></a>일반적인 문제 해결 지침

GMSA를 사용 하 여 컨테이너를 실행할 때 오류가 발생 하는 경우 다음 지침을 참조 하 여 근본 원인을 확인할 수 있습니다.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>호스트가 gMSA를 사용할 수 있는지 확인

1. 호스트가 도메인에 가입 되어 있고 도메인 컨트롤러에 도달할 수 있는지 확인 합니다.
2. RSAT에서 AD PowerShell 도구를 설치 하 고 [ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) 를 실행 하 여 컴퓨터에 gMSA를 검색할 수 있는 권한이 있는지 확인 합니다. Cmdlet이 **False**를 반환 하는 경우 컴퓨터에 gMSA 암호에 대 한 액세스 권한이 없는 것입니다.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. **Test-ADServiceAccount** 가 **False**를 반환 하는 경우 gMSA 암호에 액세스할 수 있는 보안 그룹에 호스트가 속해 있는지 확인 합니다.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 호스트가 gMSA 암호를 검색 하도록 승인 된 보안 그룹에 속해 있지만 여전히 **테스트-ADServiceAccount**에 실패 하는 경우 현재 그룹 구성원을 반영 하는 새 티켓을 얻으려면 컴퓨터를 다시 시작 해야 할 수 있습니다.

#### <a name="check-the-credential-spec-file"></a>자격 증명 사양 파일 확인

1. [Credentialspec PowerShell 모듈](https://aka.ms/credspec) 에서 **Get-credentialspec** 을 실행 하 여 컴퓨터에서 모든 자격 증명 사양을 찾습니다. 자격 증명 사양이 Docker 루트 디렉터리 아래의 "CredentialSpecs" 디렉터리에 저장 되어 있어야 합니다. Docker **정보-f "{{을 (를) 실행 하 여 docker 루트 디렉토리를 찾을 수 있습니다. DockerRootDir}} "** 을 (를)
2. CredentialSpec 파일을 열고 다음 필드가 올바르게 입력 되었는지 확인 합니다.
    - **Sid**: GMSA 계정의 sid
    - **Machineaccountname**: GMSA SAM 계정 이름 (전체 도메인 이름 또는 달러 기호 포함 안 함)
    - **Dnstreename**: Active DIRECTORY 포리스트의 FQDN
    - **DnsName**: gMSA이 속한 도메인의 FQDN입니다.
    - **NetBiosName**: gMSA이 속한 도메인의 NETBIOS 이름
    - **Groupmanagedserviceaccounts/Name**: gMSA SAM 계정 이름 (전체 도메인 이름 또는 달러 기호 포함 안 함)
    - **Groupmanagedserviceaccounts/Scope**: 도메인 FQDN 및 NETBIOS에 대 한 한 가지 항목

    입력 내용이 완전 한 자격 증명 사양의 다음 예제와 같이 표시 되어야 합니다.

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. 오케스트레이션 솔루션에 대 한 자격 증명 사양 파일의 경로가 올바른지 확인 합니다. Docker를 사용 하는 경우 컨테이너 실행 명령 `--security-opt="credentialspec=file://NAME.json"`에 "이름-json"이 **Get credentialspec**의 이름 출력으로 대체 되는지 확인 합니다. 이름은 Docker 루트 디렉터리 아래의 CredentialSpecs 폴더를 기준으로 하는 플랫 파일 이름입니다.

### <a name="check-the-firewall-configuration"></a>방화벽 구성 확인

컨테이너 또는 호스트 네트워크에서 엄격한 방화벽 정책을 사용 하는 경우 Active Directory 도메인 컨트롤러나 DNS 서버에 대 한 필수 연결이 차단 될 수 있습니다.

| 프로토콜 및 포트 | 목적 |
|-------------------|---------|
| TCP 및 UDP 53 | DNS |
| TCP 및 UDP 88 | Kerberos |
| TCP 139 | 시키려면 |
| TCP 및 UDP 389 | DAP |
| TCP 636 | LDAP SSL |

컨테이너가 도메인 컨트롤러로 보내는 트래픽 유형에 따라 추가 포트에 대 한 액세스를 허용 해야 할 수 있습니다.
Active directory에 사용 되는 전체 포트 목록은 active [directory 및 Active Directory 도메인 서비스 포트 요구 사항을](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) 참조 하세요.

### <a name="check-the-container"></a>컨테이너 확인

1. Windows Server 2019 또는 Windows 10, 버전 1809 이전 버전의 Windows를 실행 하는 경우 컨테이너 호스트 이름은 gMSA 이름과 일치 해야 합니다. `--hostname` 매개 변수가 gMSA 약식 이름 (예: "webapp01.contoso.com"이 아닌 "webapp01")과 일치 하는지 확인 합니다.

2. 컨테이너 네트워킹 구성을 확인 하 여 컨테이너가 gMSA의 도메인에 대 한 도메인 컨트롤러를 확인 하 고 액세스할 수 있는지 확인 합니다. 컨테이너에서 잘못 구성 된 DNS 서버는 id 문제의 일반적인 원인입니다.

3. 컨테이너에 다음 cmdlet을 실행 하 여 컨테이너의 도메인에 대 한 올바른 연결이 있는지 확인 합니다 (사용 `docker exec` 또는 해당 하는 경우).

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    GMSA을 사용할 수 있고 `NERR_SUCCESS` 네트워크 연결을 통해 컨테이너가 도메인과 통신할 수 있는지에 따라 신뢰 확인이 반환 됩니다. 실패 하는 경우 호스트와 컨테이너의 네트워크 구성을 확인 합니다. 둘 다 도메인 컨트롤러와 통신할 수 있어야 합니다.

4. 컨테이너가 올바른 TGT (Kerberos 티켓 허용 티켓)를 얻을 수 있는지 확인 합니다.

    ```powershell
    klist get krbtgt
    ```

    이 명령은 "krbtgt에 대 한 티켓이 성공적으로 검색 되었습니다."를 반환 하 고 티켓을 검색 하는 데 사용 되는 도메인 컨트롤러를 나열 해야 합니다. TGT를 가져올 수 있지만 `nltest` 이전 단계에서 실패 하는 경우 gMSA 계정이 잘못 구성 되었음을 나타내는 것일 수 있습니다. 자세한 내용은 [gMSA 계정 확인](#check-the-gmsa-account) 을 참조 하세요.

    컨테이너 내에서 TGT를 받을 수 없는 경우 DNS 또는 네트워크 연결 문제를 나타낼 수 있습니다. 컨테이너가 도메인 DNS 이름을 사용 하 여 도메인 컨트롤러를 확인 하 고 컨테이너에서 도메인 컨트롤러를 라우팅할 수 있는지 확인 합니다.

5. 앱이 gMSA를 [사용 하도록 구성](gmsa-configure-app.md)되어 있는지 확인 합니다. GMSA를 사용 하는 경우 컨테이너 내부의 사용자 계정이 변경 되지 않습니다. 대신 시스템 계정은 다른 네트워크 리소스와 통신 하는 경우 gMSA를 사용 합니다. 즉, gMSA id를 활용 하려면 앱이 네트워크 서비스 또는 로컬 시스템으로 실행 되어야 합니다.

    > [!TIP]
    > 컨테이너에서 현재 `whoami` 사용자 컨텍스트를 식별 하기 위해 다른 도구를 실행 하거나 사용 하는 경우 gMSA 이름이 표시 되지 않습니다. 이는 도메인 id 대신 로컬 사용자로 항상 컨테이너에 로그인 하기 때문입니다. GMSA는 네트워크 리소스와 통신 하는 경우 컴퓨터 계정에서 사용 되며,이 경우 앱을 네트워크 서비스 또는 로컬 시스템으로 실행 해야 합니다.

### <a name="check-the-gmsa-account"></a>GMSA 계정 확인

1. 컨테이너가 올바르게 구성 된 것 처럼 보이지만 사용자 또는 다른 서비스가 containerized 앱에 자동으로 인증할 수 없는 경우 gMSA 계정에서 Spn을 확인 하세요. 클라이언트는 응용 프로그램에 도달 하는 이름으로 gMSA 계정을 찾습니다. 예를 들어 클라이언트가 부하 분산 장치 또는 `host` 다른 DNS 이름을 통해 앱에 연결 하는 경우 gMSA에 대 한 추가 spn이 필요할 수 있습니다.

2. GMSA 및 컨테이너 호스트가 동일한 Active Directory 도메인에 속해 있는지 확인 합니다. GMSA가 다른 도메인에 속해 있으면 컨테이너 호스트가 gMSA 암호를 검색할 수 없습니다.

3. 도메인에 gMSA와 동일한 이름을 가진 계정이 하나만 있는지 확인 합니다. gMSA 개체에는 해당 SAM 계정 이름에 달러 기호 ($)가 추가 되므로, gMSA에 "내 계정 $" 이라는 이름을 지정 하 고 관련 없는 사용자 계정에 "내 계정" 라는 이름을 지정할 수 있습니다. 이로 인해 도메인 컨트롤러나 응용 프로그램이 gMSA를 이름으로 조회 해야 하는 경우 문제가 발생할 수 있습니다. 다음 명령을 사용 하 여 유사한 이름의 개체에 대해 광고를 검색할 수 있습니다.

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. GMSA 계정에서 무제한 위임을 사용 하도록 설정한 경우 [UserAccountControl 특성](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties) 에 `WORKSTATION_TRUST_ACCOUNT` 플래그가 설정 되어 있는지 확인 합니다. 이 플래그는 앱에서 SID에 대 한 이름을 확인 하거나 그 반대의 경우와 같이 도메인 컨트롤러와 통신 하기 위해 컨테이너의 NETLOGON에 필요 합니다. 다음 명령을 사용 하 여 플래그가 올바르게 구성 되어 있는지 확인할 수 있습니다.

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    위의 명령이 반환 `False`되는 경우 다음을 사용 하 여 GMSA `WORKSTATION_TRUST_ACCOUNT` 계정의 UserAccountControl 속성에 플래그를 추가 합니다. 이 명령은 또한 UserAccountControl 속성의 `NORMAL_ACCOUNT`, `INTERDOMAIN_TRUST_ACCOUNT`및 `SERVER_TRUST_ACCOUNT` 플래그를 지웁니다.

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
