---
title: Windows 컨테이너의 gMSAs 문제 해결
description: Windows 컨테이너의 gMSAs (그룹 관리 서비스 계정) 문제를 해결 하는 방법입니다.
keywords: docker, 컨테이너, active directory, gmsa, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정, 문제 해결, 문제 해결
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910243"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Windows 컨테이너의 gMSAs 문제 해결

## <a name="known-issues"></a>알려진 문제

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>컨테이너 호스트 이름은 Windows Server 2016 및 Windows 10 버전 1709 및 1803에 대 한 gMSA 이름과 일치 해야 합니다.

Windows Server 2016, 버전 1709 또는 1803를 실행 하는 경우 컨테이너의 호스트 이름이 gMSA SAM 계정 이름과 일치 해야 합니다.

호스트 이름이 gMSA 이름과 일치 하지 않는 경우에는 인바운드 NTLM 인증 요청 및 이름/s s l 변환 (ASP.NET 멤버 자격 역할 공급자와 같은 많은 라이브러리에서 사용 됨)이 실패 합니다. Hostname 및 gMSA 이름이 일치 하지 않는 경우에도 Kerberos가 계속 정상적으로 작동 합니다.

이러한 제한 사항은 Windows Server 2019에서 수정 되었습니다. 그러면 컨테이너는 이제 할당 된 호스트 이름에 관계 없이 항상 네트워크에서 gMSA 이름을 사용 합니다.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>둘 이상의 컨테이너와 함께 gMSA을 동시에 사용 하면 Windows Server 2016 및 Windows 10 버전 1709 및 1803에서 간헐적 오류가 발생 합니다.

모든 컨테이너가 동일한 호스트 이름을 사용 해야 하기 때문에 두 번째 문제는 windows Server 2019 및 Windows 10 버전 1809 이전 버전의 Windows에 영향을 줍니다. 여러 컨테이너에 동일한 id 및 호스트 이름이 할당 되 면 두 컨테이너가 동일한 도메인 컨트롤러와 동시에 통신할 때 경합 상태가 발생할 수 있습니다. 다른 컨테이너가 동일한 도메인 컨트롤러와 통신 하는 경우 동일한 id를 사용 하 여 이전 컨테이너와 통신을 취소 합니다. 이로 인해 일시적인 인증 오류가 발생할 수 있으며, 컨테이너 내부에서 `nltest /sc_verify:contoso.com`를 실행 하면 신뢰 오류로 관찰 될 수 있습니다.

컴퓨터 이름과 컨테이너 id를 구분 하 여 여러 컨테이너가 동일한 gMSA를 동시에 사용할 수 있도록 Windows Server 2019의 동작을 변경 했습니다.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Windows 10 버전 1703, 1709 및 1803에서 Hyper-v 격리 컨테이너와 함께 gMSAs를 사용할 수 없습니다.

Windows 10 및 Windows Server 버전 1703, 1709 및 1803에서 Hyper-v 격리 컨테이너와 함께 gMSA을 사용 하려고 하면 컨테이너 초기화가 중단 되거나 실패 합니다.

이 버그는 Windows Server 2019 및 Windows 10 버전 1809에서 수정 되었습니다. Windows Server 2016 및 Windows 10 버전 1607에서 gMSAs를 사용 하 여 Hyper-v 격리 컨테이너를 실행할 수도 있습니다.

## <a name="general-troubleshooting-guidance"></a>일반 문제 해결 지침

GMSA를 사용 하 여 컨테이너를 실행할 때 오류가 발생 하는 경우 다음 지침을 참조 하 여 근본 원인을 식별할 수 있습니다.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>호스트에서 gMSA를 사용할 수 있는지 확인 합니다.

1. 호스트가 도메인에 가입 되어 있고 도메인 컨트롤러에 연결할 수 있는지 확인 하십시오.
2. RSAT에서 AD PowerShell 도구를 설치 하 고 [uninstall-adserviceaccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) 를 실행 하 여 컴퓨터에 gMSA 검색을 위한 액세스 권한이 있는지 확인 합니다. Cmdlet이 **False**를 반환 하는 경우 컴퓨터에 gMSA 암호에 대 한 액세스 권한이 없는 것입니다.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. **Uninstall-adserviceaccount** 가 **False**를 반환 하는 경우 호스트는 gMSA 암호에 액세스할 수 있는 보안 그룹에 속하는지 확인 합니다.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 호스트가 gMSA 암호를 검색할 수 있는 권한이 있는 보안 그룹에 속해 있지만 여전히 **테스트 uninstall-adserviceaccount**에 실패 하는 경우 컴퓨터를 다시 시작 하 여 현재 그룹 멤버 자격을 반영 하는 새 티켓을 가져와야 할 수 있습니다.

#### <a name="check-the-credential-spec-file"></a>자격 증명 사양 파일 확인

1. [Credentialspec PowerShell 모듈](https://aka.ms/credspec) 에서 **Get credentialspec** 을 실행 하 여 컴퓨터의 모든 자격 증명 사양을 찾습니다. 자격 증명 사양은 Docker 루트 디렉터리 아래의 "CredentialSpecs" 디렉터리에 저장 해야 합니다. Docker **정보-f "{{를 실행 하 여 docker 루트 디렉터리를 찾을 수 있습니다. DockerRootDir}} "** .
2. CredentialSpec 파일을 열고 다음 필드를 올바르게 입력 했는지 확인 합니다.
    - **Sid**: GMSA 계정의 sid입니다.
    - **Machineaccountname**: GMSA SAM 계정 이름 (전체 도메인 이름 또는 달러 기호 포함 안 함)
    - **Dnstreename**: ACTIVE DIRECTORY 포리스트의 FQDN
    - **DnsName**: gMSA가 속한 도메인의 FQDN
    - **NetBiosName**: gMSA가 속한 도메인의 NETBIOS 이름
    - **Groupmanagedserviceaccounts/Name**: gMSA SAM 계정 이름 (전체 도메인 이름 또는 달러 기호 포함 안 함)
    - **Groupmanagedserviceaccounts/Scope**: 도메인 FQDN에 대 한 항목 한 개 및 NETBIOS 용

    입력은 전체 자격 증명 사양의 다음 예제와 같이 표시 됩니다.

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

3. 오케스트레이션 솔루션에 대 한 자격 증명 사양 파일의 경로가 올바른지 확인 합니다. Docker를 사용 하는 경우 컨테이너 실행 명령에 `--security-opt="credentialspec=file://NAME.json"`이 포함 되어 있는지 확인 합니다. 여기서 "NAME. json"은 **Get CredentialSpec**에 의해 이름으로 출력 됩니다. 이름은 Docker 루트 디렉터리 아래의 CredentialSpecs 폴더를 기준으로 하는 플랫 파일 이름입니다.

### <a name="check-the-firewall-configuration"></a>방화벽 구성 확인

컨테이너 또는 호스트 네트워크에서 엄격한 방화벽 정책을 사용 하는 경우 Active Directory 도메인 컨트롤러 또는 DNS 서버에 대 한 필수 연결을 차단할 수 있습니다.

| 프로토콜 및 포트 | 용도 |
|-------------------|---------|
| TCP 및 UDP 53 | DNS |
| TCP 및 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 및 UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

컨테이너가 도메인 컨트롤러에 보내는 트래픽 유형에 따라 추가 포트에 대 한 액세스를 허용 해야 할 수도 있습니다.
Active Directory에서 사용 하는 포트의 전체 목록은 [Active Directory 및 Active Directory Domain Services 포트 요구 사항](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) 을 참조 하세요.

### <a name="check-the-container"></a>컨테이너를 확인 합니다.

1. Windows Server 2019 또는 Windows 10 버전 1809 이전 버전의 Windows를 실행 하는 경우 컨테이너 호스트 이름이 gMSA 이름과 일치 해야 합니다. `--hostname` 매개 변수가 gMSA short 이름과 일치 하는지 확인 합니다 (예: "webapp01.contoso.com" 대신 "webapp01").

2. 컨테이너 네트워킹 구성을 검사 하 여 컨테이너에서 gMSA의 도메인에 대 한 도메인 컨트롤러를 확인 하 고 액세스할 수 있는지 확인 합니다. 컨테이너에서 잘못 구성 된 DNS 서버는 id 문제의 일반적인 원인입니다.

3. 컨테이너에서 다음 cmdlet을 실행 하 여 (`docker exec` 또는 이와 동등한) 컨테이너에 올바른 도메인 연결이 있는지 확인 합니다.

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    GMSA를 사용할 수 있고 네트워크 연결을 통해 컨테이너에서 도메인에 연결할 수 있는 경우 트러스트 확인은 `NERR_SUCCESS`를 반환 해야 합니다. 실패 한 경우 호스트 및 컨테이너의 네트워크 구성을 확인 합니다. 둘 다 도메인 컨트롤러와 통신할 수 있어야 합니다.

4. 컨테이너가 유효한 Kerberos TGT (허용 티켓)를 얻을 수 있는지 확인 합니다.

    ```powershell
    klist get krbtgt
    ```

    이 명령은 "krbtgt에 대 한 티켓을 검색 했습니다."를 반환 하 고 티켓을 검색 하는 데 사용 되는 도메인 컨트롤러를 나열 합니다. TGT를 얻을 수 있지만 이전 단계의 `nltest` 실패 한 경우 gMSA 계정이 잘못 구성 되었음을 나타낼 수 있습니다. 자세한 내용은 [gMSA 계정 확인](#check-the-gmsa-account) 을 참조 하세요.

    컨테이너 내에서 TGT를 가져올 수 없는 경우 DNS 또는 네트워크 연결 문제가 있음을 나타낼 수 있습니다. 컨테이너가 도메인 DNS 이름을 사용 하 여 도메인 컨트롤러를 확인 하 고 컨테이너에서 도메인 컨트롤러를 라우팅할 수 있는지 확인 합니다.

5. 앱이 gMSA를 [사용 하도록 구성](gmsa-configure-app.md)되어 있는지 확인 합니다. GMSA를 사용 하는 경우 컨테이너 내부의 사용자 계정이 변경 되지 않습니다. 대신 시스템 계정은 다른 네트워크 리소스와 통신 하는 경우 gMSA를 사용 합니다. 즉, gMSA id를 활용 하려면 앱을 네트워크 서비스 또는 로컬 시스템으로 실행 해야 합니다.

    > [!TIP]
    > `whoami`를 실행 하거나 다른 도구를 사용 하 여 컨테이너에서 현재 사용자 컨텍스트를 식별 하는 경우 gMSA 이름이 표시 되지 않습니다. 이는 항상 도메인 id 대신 로컬 사용자로 컨테이너에 로그인 하기 때문입니다. GMSA는 네트워크 리소스에 대 한 통신을 수행할 때마다 컴퓨터 계정에서 사용 됩니다. 따라서 응용 프로그램이 네트워크 서비스 또는 로컬 시스템으로 실행 되어야 합니다.

### <a name="check-the-gmsa-account"></a>GMSA 계정을 확인 합니다.

1. 컨테이너가 올바르게 구성 된 것 같지만 사용자 또는 다른 서비스에서 자동으로 컨테이너 화 된 앱에 인증할 수 없는 경우 gMSA 계정의 Spn을 확인 하세요. 클라이언트는 응용 프로그램에 도달 하는 이름으로 gMSA 계정을 찾습니다. 이는 예를 들어 클라이언트에서 부하 분산 장치 또는 다른 DNS 이름을 통해 앱에 연결 하는 경우 gMSA에 대 한 추가 `host` Spn이 필요 함을 의미할 수 있습니다.

2. GMSA 및 container host가 동일한 Active Directory 도메인에 속해 있는지 확인 하세요. GMSA가 다른 도메인에 속하는 경우 컨테이너 호스트는 gMSA 암호를 검색할 수 없습니다.

3. 도메인에 gMSA와 동일한 이름을 가진 계정이 하나만 있는지 확인 합니다. gMSA 개체에는 SAM 계정 이름에 달러 기호 ($)가 추가 되어 있으므로 gMSA의 이름을 "myaccount $"로 지정 하 고 관련 없는 사용자 계정을 동일한 도메인에 "myaccount"로 명명할 수 있습니다. 도메인 컨트롤러 또는 응용 프로그램이 이름으로 gMSA를 조회 해야 하는 경우이로 인해 문제가 발생할 수 있습니다. 다음 명령을 사용 하 여 유사한 이름의 개체에 대 한 광고를 검색할 수 있습니다.

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. GMSA 계정에서 제한 되지 않은 위임을 사용 하도록 설정한 경우에는 [UserAccountControl 특성](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties) 에 `WORKSTATION_TRUST_ACCOUNT` 플래그를 사용 하도록 설정 되어 있는지 확인 합니다. 이 플래그는 앱이 도메인 컨트롤러와 통신 하기 위해 도메인 컨트롤러와 통신 하는 경우에 필요 합니다. 다음 명령을 사용 하 여 플래그가 올바르게 구성 되었는지 확인할 수 있습니다.

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    위의 명령이 `False`을 반환 하는 경우 다음을 사용 하 여 `WORKSTATION_TRUST_ACCOUNT` 플래그를 gMSA 계정의 UserAccountControl 속성에 추가 합니다. 또한이 명령은 UserAccountControl 속성에서 `NORMAL_ACCOUNT`, `INTERDOMAIN_TRUST_ACCOUNT`및 `SERVER_TRUST_ACCOUNT` 플래그를 지웁니다.

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
