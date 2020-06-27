---
title: Windows 컨테이너에 대한 gMSA 문제 해결
description: Windows 컨테이너에 대한 gMSA(그룹 관리 서비스 계정) 문제를 해결하는 방법입니다.
keywords: Docker, 컨테이너, Active Directory, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정, 문제 해결, 문제 해결
author: rpsqrd
ms.date: 10/03/2019
ms.topic: troubleshooting
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: e7cf5685620d3cb50c93f48e5aa6917d9044b860
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192830"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Windows 컨테이너에 대한 gMSA 문제 해결

## <a name="known-issues"></a>알려진 문제

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>컨테이너 호스트 이름이 Windows Server 2016 및 Windows 10 버전 1709 및 1803에 대한 gMSA 이름과 일치해야 함

Windows Server 2016 버전 1709 또는 1803을 실행하는 경우 컨테이너의 호스트 이름이 gMSA SAM 계정 이름과 일치해야 합니다.

호스트 이름이 gMSA 이름과 일치하지 않으면 인바운드 NTLM 인증 요청 및 이름/SID 변환(ASP.NET 멤버 자격 역할 공급자와 같은 많은 라이브러리에서 사용됨)이 실패합니다. 호스트 이름과 gMSA 이름이 일치하지 않는 경우에도 Kerberos는 정상적으로 계속 작동합니다.

이 제한은 Windows Server 2019에서 수정되었으며, 이제 컨테이너는 할당된 호스트 이름과 관계없이 네트워크에서 항상 gMSA 이름을 사용합니다.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>둘 이상의 컨테이너에서 gMSA를 동시에 사용하면 Windows Server 2016과 Windows 10 버전 1709 및 1803에서 일시적 오류가 발생함

모든 컨테이너에서 동일한 호스트 이름을 사용해야 하므로 두 번째 문제는 Windows Server 2019 및 Windows 10 버전 1809 이전의 Windows에 영향을 줍니다. 여러 컨테이너에 동일한 ID와 호스트 이름이 할당되면 두 컨테이너에서 동일한 도메인 컨트롤러와 동시에 통신할 때 경합 상태가 발생할 수 있습니다. 다른 컨테이너에서 동일한 도메인 컨트롤러와 통신하면 동일한 ID를 사용하는 이전 컨테이너와의 통신이 취소됩니다. 이로 인해 일시적 인증 실패가 발생할 수 있으며, 컨테이너 내에서 `nltest /sc_verify:contoso.com`을 실행하면 신뢰 오류로 관찰될 수 있는 경우도 있습니다.

컨테이너 ID를 머신 이름과 구분하여 여러 컨테이너에서 동일한 gMSA를 동시에 사용할 수 있도록 Windows Server 2019의 동작을 변경했습니다.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Windows 10 버전 1703, 1709 및 1803의 Hyper-V 격리 컨테이너에서 gMSA를 사용할 수 없음

Windows 10 및 Windows Server 버전 1703, 1709 및 1803의 Hyper-V 격리 컨테이너에서 gMSA를 사용하려고 하면 컨테이너 초기화가 중단되거나 실패합니다.

이 버그는 Windows Server 2019 및 Windows 10 버전 1809에서 수정되었습니다. 또한 Windows Server 2016 및 Windows 10 버전 1607에서 gMSA를 사용하여 Hyper-V 격리 컨테이너를 실행할 수도 있습니다.

## <a name="general-troubleshooting-guidance"></a>일반 문제 해결 지침

gMSA를 사용하여 컨테이너를 실행할 때 오류가 발생하면 다음 지침에 따라 근본 원인을 식별할 수 있습니다.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>호스트에서 gMSA를 사용할 수 있는지 확인

1. 호스트가 도메인에 조인되어 있고 도메인 컨트롤러에 연결할 수 있는지 확인합니다.
2. RSAT에서 AD PowerShell 도구를 설치하고 [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)를 실행하여 gMSA를 검색할 수 있는 액세스 권한이 컴퓨터에 있는지 확인합니다. cmdlet에서 **False**를 반환하는 경우 컴퓨터에서 gMSA 암호에 액세스할 수 없습니다.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. **Test-ADServiceAccount**에서 **False**를 반환하는 경우 호스트가 gMSA 암호에 액세스할 수 있는 보안 그룹에 속하는지 확인합니다.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 호스트가 gMSA 암호를 검색할 수 있는 권한이 있는 보안 그룹에 속하지만 **Test-ADServiceAccount**가 여전히 실패하는 경우 컴퓨터를 다시 시작하여 현재 그룹 멤버 자격을 반영하는 새 티켓을 얻어야 할 수도 있습니다.

#### <a name="check-the-credential-spec-file"></a>자격 증명 사양 파일 확인

1. [CredentialSpec PowerShell 모듈](https://aka.ms/credspec)에서 **Get-CredentialSpec**을 실행하여 머신에서 모든 자격 증명 사양을 찾습니다. 자격 증명 사양은 Docker 루트 디렉터리 아래의 "CredentialSpecs" 디렉터리에 저장해야 합니다. **docker info -f "{{.DockerRootDir}}"** 을 실행하여 Docker 루트 디렉터리를 찾을 수 있습니다.
2. CredentialSpec 파일을 열고 다음 필드가 올바르게 채워졌는지 확인합니다.
    - **Sid**: gMSA 계정의 SID
    - **MachineAccountName**: gMSA SAM 계정 이름(전체 도메인 이름 또는 달러 기호를 포함하지 않음)
    - **DnsTreeName**: Active Directory 포리스트의 FQDN
    - **DnsName**: gMSA가 속한 도메인의 FQDN
    - **NetBiosName**: gMSA가 속한 도메인의 NETBIOS 이름
    - **GroupManagedServiceAccounts/Name**: gMSA SAM 계정 이름(전체 도메인 이름 또는 달러 기호를 포함하지 않음)
    - **GroupManagedServiceAccounts/Scope**: 도메인 FQDN에 대한 항목 1개 및 NETBIOS에 대한 항목 1개

    입력은 전체 자격 증명 사양의 다음 예제와 같습니다.

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

3. 오케스트레이션 솔루션에 대한 자격 증명 사양 파일의 경로가 올바른지 확인합니다. Docker를 사용하는 경우 `--security-opt="credentialspec=file://NAME.json"`이 컨테이너 실행 명령에 포함되어 있는지 확인합니다. 여기서는 **Get-CredentialSpec**에서 "NAME.json"을 이름 출력으로 바꿉니다. 이 이름은 Docker 루트 디렉터리 아래의 CredentialSpecs 폴더를 기준으로 하는 플랫 파일 이름입니다.

### <a name="check-the-firewall-configuration"></a>방화벽 구성 확인

컨테이너 또는 호스트 네트워크에서 엄격한 방화벽 정책을 사용하는 경우 Active Directory 도메인 컨트롤러 또는 DNS 서버에 대한 필수 연결을 차단할 수 있습니다.

| 프로토콜 및 포트 | 용도 |
|-------------------|---------|
| TCP 및 UDP 53 | DNS |
| TCP 및 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 및 UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

컨테이너에서 도메인 컨트롤러에 보내는 트래픽 유형에 따라 추가 포트에 대한 액세스를 허용해야 할 수도 있습니다.
Active Directory에서 사용하는 포트의 전체 목록은 [Active Directory 및 Active Directory Domain Services 포트 요구 사항](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)을 참조하세요.

### <a name="check-the-container"></a>컨테이너 확인

1. Windows Server 2019 또는 Windows 10 버전 1809 이전의 Windows 버전을 실행하는 경우 컨테이너 호스트 이름이 gMSA 이름과 일치해야 합니다. `--hostname` 매개 변수가 gMSA 약식 이름(예: "webapp01.contoso.com" 대신 "webapp01")과 일치하는지 확인합니다.

2. 컨테이너 네트워킹 구성을 확인하여 컨테이너에서 gMSA 도메인의 도메인 컨트롤러를 확인하고 액세스할 수 있는지 확인합니다. 컨테이너에서 잘못 구성된 DNS 서버는 ID 문제의 일반적인 원인입니다.

3. 컨테이너에서 다음 cmdlet을 실행하여(`docker exec` 또는 이와 동등한 cmdlet 사용) 컨테이너가 도메인에 올바르게 연결되어 있는지 확인합니다.

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    gMSA를 사용할 수 있고 네트워크 연결을 통해 컨테이너가 도메인과 통신할 수 있는 경우 트러스트 검증에서 `NERR_SUCCESS`를 반환해야 합니다. 실패하면 호스트 및 컨테이너의 네트워크 구성을 확인합니다. 둘 다 도메인 컨트롤러와 통신할 수 있어야 합니다.

4. 컨테이너에서 유효한 Kerberos TGT(허용 티켓)를 얻을 수 있는지 확인합니다.

    ```powershell
    klist get krbtgt
    ```

    이 명령은 "krbtgt에 대한 티켓이 성공적으로 검색되었습니다."를 반환하고 티켓을 검색하는 데 사용된 도메인 컨트롤러를 나열해야 합니다. TGT를 얻을 수 있지만 이전 단계에서 `nltest`가 실패하면 gMSA 계정이 잘못 구성되었음을 나타낼 수 있습니다. 자세한 내용은 [gMSA 계정 확인](#check-the-gmsa-account)을 참조하세요.

    컨테이너 내에서 TGT를 얻을 수 없는 경우 DNS 또는 네트워크 연결 문제를 나타낼 수 있습니다. 컨테이너에서 도메인 DNS 이름을 사용하여 도메인 컨트롤러를 확인할 수 있고 도메인 컨트롤러가 컨테이너에서 라우팅될 수 있는지 확인합니다.

5. 앱이 [gMSA를 사용하도록 구성](gmsa-configure-app.md)되어 있는지 확인합니다. gMSA를 사용하는 경우 컨테이너 내부의 사용자 계정은 변경되지 않습니다. 대신 시스템 계정은 다른 네트워크 리소스와 통신할 때 gMSA를 사용합니다. 즉 gMSA ID를 활용하기 위해 앱을 네트워크 서비스 또는 로컬 시스템으로 실행해야 합니다.

    > [!TIP]
    > `whoami`를 실행하거나 다른 도구를 사용하여 컨테이너에서 현재 사용자 컨텍스트를 식별하면 gMSA 이름 자체가 표시되지 않습니다. 이는 항상 도메인 ID 대신 로컬 사용자로 컨테이너에 로그인하기 때문입니다. gMSA는 네트워크 리소스와 통신할 때마다 컴퓨터 계정에서 사용되므로 앱을 네트워크 서비스 또는 로컬 시스템으로 실행해야 합니다.

### <a name="check-the-gmsa-account"></a>gMSA 계정 확인

1. 컨테이너가 올바르게 구성된 것 같지만 사용자 또는 다른 서비스에서 자동으로 컨테이너화된 앱에 인증할 수 없는 경우 gMSA 계정의 SPN을 확인합니다. 클라이언트는 애플리케이션에 도달하는 이름을 기준으로 gMSA 계정을 찾습니다. 예를 들어 클라이언트에서 부하 분산 장치 또는 다른 DNS 이름을 통해 앱에 연결하는 경우 `host` SPN이 gMSA에 추가로 필요할 수 있습니다.

2. gMSA 및 컨테이너 호스트가 동일한 Active Directory 도메인에 속하는지 확인합니다. gMSA가 다른 도메인에 속하는 경우 컨테이너 호스트에서 gMSA 암호를 검색할 수 없습니다.

3. gMSA와 동일한 이름을 사용하는 하나의 계정만 도메인에 있는지 확인합니다. gMSA 개체의 해당 SAM 계정 이름에는 $(달러) 기호가 추가되어 있으므로 gMSA를 "myaccount$"라는 이름으로 지정하고, 관련 없는 사용자 계정은 동일한 도메인에서 "myaccount"라는 이름으로 지정할 수 있습니다. 이로 인해 도메인 컨트롤러 또는 애플리케이션에서 이름을 기준으로 gMSA를 조회해야 하는 경우 문제가 발생할 수 있습니다. 다음 명령을 사용하여 AD에서 비슷한 이름의 개체를 검색할 수 있습니다.

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. gMSA 계정에서 제한 없는 위임을 사용하도록 설정된 경우 [UserAccountControl 특성](https://support.microsoft.com/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties)에서 `WORKSTATION_TRUST_ACCOUNT` 플래그가 여전히 사용하도록 설정되어 있는지 확인합니다. 앱에서 이름을 SID로 확인하거나 그 반대로 해석해야 하는 경우와 마찬가지로 이 플래그는 컨테이너의 NETLOGON에서 도메인 컨트롤러와 통신하는 데 필요합니다. 다음 명령을 사용하여 플래그가 올바르게 구성되어 있는지 확인할 수 있습니다.

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    위의 명령이 `False`를 반환하는 경우 다음을 사용하여 `WORKSTATION_TRUST_ACCOUNT` 플래그를 gMSA 계정의 UserAccountControl 속성에 추가합니다. 또한 이 명령은 UserAccountControl 속성에서 `NORMAL_ACCOUNT`, `INTERDOMAIN_TRUST_ACCOUNT` 및 `SERVER_TRUST_ACCOUNT` 플래그도 지웁니다.

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
