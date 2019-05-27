---
title: Windows 컨테이너에 대 한 그룹 관리 서비스 계정
description: Windows 컨테이너에 대 한 그룹 관리 서비스 계정
keywords: docker, 컨테이너, active directory, gmsa
author: rpsqrd
ms.date: 05/23/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 8f184e58743bd41ab208b530976772c5fcffd189
ms.sourcegitcommit: 8e7fba17c761bf8f80017ba7f9447f986a20a2a7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/24/2019
ms.locfileid: "9677322"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Windows 컨테이너에 대 한 그룹 관리 서비스 계정

Windows 기반 네트워크는 일반적으로 AD (Active Directory)를 사용 하 여 사용자, 컴퓨터 및 다른 네트워크 리소스 간의 인증 및 권한 부여를 지원 합니다. 엔터프라이즈 응용 프로그램 개발자는 사용자와 다른 서비스가 쉽게 자동으로 로그인 할 수 있도록 통합 Windows 인증을 활용 하기 위해 도메인 가입 서버에서 AD 통합 및 실행 되도록 앱을 디자인 합니다. id를 사용 하는 응용 프로그램

Windows 컨테이너는 도메인에 가입할 수 없지만, Active Directory 도메인 id를 사용 하 여 다양 한 인증 시나리오를 지원할 수 있습니다.

이를 위해 여러 컴퓨터가 필요 없이 id를 공유할 수 있도록 설계 된 [](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) windows Server 2012에서 도입 된 특별 한 유형의 서비스 계정으로 windows 컨테이너를 구성 하 여 실행할 수 있습니다. 암호를 확인 합니다.

GMSA를 사용 하 여 컨테이너를 실행 하는 경우 컨테이너 호스트는 Active Directory 도메인 컨트롤러에서 gMSA 암호를 검색 하 여 컨테이너 인스턴스에 게 제공 합니다. 컨테이너는 컴퓨터 계정 (시스템)이 네트워크 리소스에 액세스 해야 할 때마다 gMSA 자격 증명을 사용 합니다.

이 문서에서는 Windows 컨테이너에서 Active Directory 그룹 관리 서비스 계정 사용을 시작 하는 방법에 대해 설명 합니다.

## <a name="prerequisites"></a>필수 구성 요소

그룹 관리 서비스 계정을 사용 하 여 Windows 컨테이너를 실행 하려면 다음이 필요 합니다.

- Windows Server 2012 이상을 실행 하는 도메인 컨트롤러가 하나 이상 있는 Active Directory 도메인 GMSAs를 사용 하는 포리스트 또는 도메인 기능 수준의 요구 사항은 없지만 gMSA 암호는 Windows Server 2012 이상을 실행 하는 도메인 컨트롤러에 의해서만 배포 될 수 있습니다. 자세한 내용은 [gMSAs 대 한 Active Directory 요구 사항을](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)참조 하세요.
- GMSA 계정을 만들 수 있는 권한이 있어야 합니다. GMSA 계정을 만들려면 도메인 관리자 이거나 *msDS-GroupManagedServiceAccount Objects 만들기* 권한이 위임 된 계정을 사용 해야 합니다.
- CredentialSpec PowerShell 모듈을 다운로드 하기 위한 인터넷에 액세스할 수 있습니다. 연결이 끊긴 환경에서 작업 하는 경우 인터넷에 액세스 하는 컴퓨터에 [모듈을 저장](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) 하 고 개발 컴퓨터 또는 컨테이너 호스트에 복사할 수 있습니다.

## <a name="one-time-preparation-of-active-directory"></a>Active Directory에 대 한 일회성 준비

도메인에 gMSA를 아직 만들지 않은 경우 키 배포 서비스 (KDS) 루트 키를 생성 해야 합니다. KDS는 권한이 있는 호스트에 대 한 gMSA 암호 만들기, 회전 및 해제를 담당 합니다. 컨테이너 호스트가 컨테이너를 실행 하기 위해 gMSA를 사용 해야 하는 경우에는 KDS에 문의 하 여 현재 암호를 검색 해야 합니다.

KDS 루트 키가 이미 생성 되었는지 확인 하려면 다음 PowerShell cmdlet을 AD PowerShell 도구가 설치 된 도메인 컨트롤러 또는 도메인 구성원의 도메인 관리자로 실행 합니다.

```powershell
Get-KdsRootKey
```

명령이 키 ID를 반환 하는 경우 모두 설정 되며 [그룹 관리 서비스 계정 만들기](#create-a-group-managed-service-account) 섹션으로 건너뛸 수 있습니다. 그렇지 않으면 계속 해 서 KDS 루트 키를 만듭니다.

여러 도메인 컨트롤러가 있는 프로덕션 환경 또는 테스트 환경에서 PowerShell의 다음 cmdlet을 도메인 관리자로 실행 하 여 KDS 루트 키를 만듭니다.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

명령에는 키가 즉시 효력을 내포 하 고 있지만, KDS 루트 키를 복제 하 고 모든 도메인 컨트롤러에서 사용할 수 있으려면 10 시간 동안 기다려야 합니다.

도메인에 하나의 도메인 컨트롤러만 있으면 키를 10 시간 이전으로 설정 하 여 프로세스를 신속 하 게 처리할 수 있습니다.

>[!IMPORTANT]
>프로덕션 환경에서는이 방법을 사용 하지 마세요.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>그룹 관리 서비스 계정 만들기

통합 Windows 인증을 사용 하는 모든 컨테이너에는 하나 이상의 gMSA 필요 합니다. 기본 gMSA 네트워크에서 시스템 또는 네트워크 서비스에 대 한 액세스 리소스로 실행 되는 앱이 있는 경우에 사용 됩니다. GMSA의 이름은 컨테이너에 지정 된 호스트 이름에 관계 없이 네트워크에서 컨테이너의 이름이 됩니다. 컨테이너 컴퓨터 계정에서 컨테이너의 서비스 또는 응용 프로그램을 다른 id로 실행 하려는 경우에는 컨테이너를 추가 gMSAs와 함께 구성할 수도 있습니다.

GMSA를 만들 때 여러 컴퓨터에서 동시에 사용할 수 있는 공유 id를 만들 수도 있습니다. GMSA 암호에 대 한 액세스는 Active Directory 액세스 제어 목록에 의해 보호 됩니다. 암호에 대 한 액세스를 제한 하기 위해 각 gMSA 계정에 대 한 보안 그룹을 만들고 보안 그룹에 관련 컨테이너 호스트를 추가 하는 것이 좋습니다.

마지막으로 컨테이너는 SPN (서비스 사용자 이름)을 자동으로 등록 하지 않으므로 gMSA 계정에 대해 적어도 하나 이상의 호스트 SPN을 수동으로 만들어야 합니다.

일반적으로 호스트 또는 http SPN은 gMSA 계정과 동일한 이름을 사용 하 여 등록 되지만 클라이언트가 containerized 응용 프로그램에 액세스 하는 경우에는 다른 서비스 이름을 사용 해야 하지만 부하 분산 장치 또는 gMSA 이름과는 다른 DNS 이름에 액세스할 수는 없습니다.

예를 들어 gMSA 계정 이름이 "WebApp01" 이지만 사용자가에서 `mysite.contoso.com`사이트에 액세스 하는 경우에는 gMSA 계정에 `http/mysite.contoso.com` SPN을 등록 해야 합니다.

일부 응용 프로그램에는 고유 프로토콜에 대 한 추가 Spn이 필요할 수 있습니다. 예를 들어, SQL Server에 `MSSQLSvc/hostname` SPN이 필요 합니다.

다음 표에는 gMSA을 만들기 위한 필수 특성이 나와 있습니다.

|gMSA 속성 | 필수 값 | 예 |
|--------------|----------------|--------|
|이름 | 유효한 계정 이름입니다. | `WebApp01` |
|DnsHostName | 계정 이름에 추가 된 도메인 이름입니다. | `WebApp01.contoso.com` |
|ServicePrincipalNames | 최소한 호스트 SPN을 설정 하 고 필요에 따라 다른 프로토콜을 추가 합니다. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 컨테이너 호스트가 포함 된 보안 그룹입니다. | `WebApp01Hosts` |

GMSA의 이름을 결정 했으면 PowerShell에서 다음 cmdlet을 실행 하 여 보안 그룹 및 gMSA를 만듭니다.

> [!TIP]
> 다음 명령을 실행 하려면 **Domain Admins** 보안 그룹에 속해 있거나 **msDS-GroupManagedServiceAccount objects 만들기** 권한이 위임 된 계정을 사용 해야 합니다.
> [ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) Cmdlet은 [원격 서버 관리 도구의](https://aka.ms/rsat)AD PowerShell 도구에 포함 되어 있습니다.

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

개발자, 테스트 및 프로덕션 환경에 대해 별도의 gMSA 계정을 만드는 것이 좋습니다.

## <a name="prepare-your-container-host"></a>컨테이너 호스트 준비

GMSA를 사용 하 여 Windows 컨테이너를 실행 하는 각 컨테이너 호스트는 도메인에 가입 되어 있고 gMSA 암호를 검색할 수 있는 권한이 있어야 합니다.

1. Active Directory 도메인에 컴퓨터를 참가 시킵니다.
2. GMSA 암호에 대 한 액세스를 제어 하는 보안 그룹에 호스트가 속해 있는지 확인 합니다.
3. 컴퓨터를 다시 시작 하 여 새 그룹 구성원 자격을 가져옵니다.
4. Windows 10 또는 [windows 용 docker](https://docs.docker.com/install/windows/docker-ee/)용 docker [데스크톱을](https://docs.docker.com/docker-for-windows/install/) 설정 합니다.
5. 권장 호스트가 [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)를 실행 하 여 gMSA 계정을 사용할 수 있는지 확인 합니다. 명령에서 **False**를 반환 하는 경우 진단 단계에 대 한 [문제 해결](#troubleshooting) 섹션을 참조 하세요.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>자격 증명 사양 만들기

자격 증명 사양 파일은 컨테이너를 사용 하려는 gMSA 계정에 대 한 메타 데이터가 포함 된 JSON 문서입니다. Id 구성을 컨테이너 이미지와 별도로 유지 하면 자격 증명 사양 파일을 바꾸어 컨테이너에서 사용 하는 gMSA 변경할 수 있으며, 필요한 코드 변경 사항은 없습니다.

자격 증명 사양 파일은 도메인 가입 컨테이너 호스트에서 [Credentialspec PowerShell 모듈](https://aka.ms/credspec) 을 사용 하 여 만들어집니다.
파일을 만든 후에는 다른 컨테이너 호스트나 컨테이너 orchestrator에 복사할 수 있습니다.
컨테이너 호스트가 컨테이너를 대신 하 여 gMSA를 검색 하므로 자격 증명 사양 파일에는 gMSA password와 같은 비밀이 포함 되어 있지 않습니다.

Docker는 Docker 데이터 디렉터리의 **credentialspecs** 디렉터리 아래에서 자격 증명 사양 파일을 찾습니다. 기본 설치의에서이 폴더를 찾습니다 `C:\ProgramData\Docker\CredentialSpecs`.

컨테이너 호스트에서 자격 증명 사양 파일을 만들려면 다음을 수행 합니다.

1. RSAT AD PowerShell 도구 설치
    - Windows Server의 경우 **설치-ADD-WINDOWSFEATURE RSAT-AD PowerShell**을 실행 합니다.
    - Windows 10 버전 1809 이상에 대해 **설치-WindowsCapability-Online ' ActiveDirectory-LDS. 도구 ~ ~ ~ ~ '** 을 실행 합니다.
    - 이전 버전의 Windows 10의 경우을 <https://aka.ms/rsat>참조 하세요.
2. 다음 cmdlet을 실행 하 여 최신 버전의 [Credentialspec PowerShell 모듈](https://aka.ms/credspec)을 설치 합니다.

    ```powershell
    Install-Module CredentialSpec
    ```

    컨테이너 호스트에서 인터넷에 액세스할 수 없는 경우 인터넷에 연결 `Save-Module CredentialSpec` 된 컴퓨터에서 실행 하 고 모듈 폴더를 `C:\Program Files\WindowsPowerShell\Modules` 또는 컨테이너 호스트의 다른 위치 `$env:PSModulePath` 에 복사 합니다.

3. 다음 cmdlet을 실행 하 여 새 자격 증명 사양 파일을 만듭니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    기본적으로 cmdlet은 제공 된 gMSA 이름을 컨테이너에 대 한 컴퓨터 계정으로 사용 하 여 자격 증명 사양을 만듭니다. 파일이 파일 이름에 대 한 gMSA 도메인 및 계정 이름을 사용 하 여 Docker CredentialSpecs 디렉터리에 저장 됩니다.

    컨테이너의 보조 gMSA 서비스 또는 프로세스를 실행 하는 경우 추가 gMSA 계정을 포함 하는 자격 증명 사양을 만들 수 있습니다. 이렇게 하려면 다음 매개 변수를 `-AdditionalAccounts` 사용 합니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    지원 되는 매개 변수의 전체 목록은를 실행 `Get-Help New-CredentialSpec`합니다.

4. 다음 cmdlet을 사용 하 여 모든 자격 증명 사양의 목록과 전체 경로를 표시할 수 있습니다.

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configure-your-application-to-use-the-gmsa"></a>GMSA를 사용 하도록 응용 프로그램 구성

일반적인 구성에서 컨테이너는 컨테이너 컴퓨터 계정이 네트워크 리소스에 대 한 인증을 시도할 때마다 사용 되는 gMSA 계정만 하나만 지정 합니다. 즉, gMSA id를 사용 해야 하는 경우 앱이 **로컬 시스템** 또는 **네트워크 서비스로** 실행 되어야 합니다.

### <a name="run-an-iis-app-pool-as-network-service"></a>IIS 앱 풀을 네트워크 서비스로 실행

컨테이너에서 IIS 웹 사이트를 호스트 하는 경우에는 gMSA를 이용 하는 데 필요한 모든 작업은 앱 풀 id를 **네트워크 서비스**에 설정 합니다. 다음 명령을 추가 하 여 Dockerfile에서이 작업을 수행할 수 있습니다.

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

이전에 IIS 앱 풀에 대 한 정적 사용자 자격 증명을 사용한 경우 해당 자격 증명을 대체 하는 gMSA을 고려해 보세요. 개발자, 테스트 및 프로덕션 환경 간에 gMSA을 변경할 수 있으며, IIS는 컨테이너 이미지를 변경할 필요 없이 현재 id를 자동으로 선택 합니다.

### <a name="run-a-windows-service-as-network-service"></a>네트워크 서비스로 Windows 서비스 실행

Containerized 앱이 Windows 서비스로 실행 되는 경우 Dockerfile에서 **네트워크 서비스로** 실행 되도록 서비스를 설정할 수 있습니다.

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>네트워크 서비스로 임의 콘솔 앱 실행

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

## <a name="run-a-container-with-a-gmsa"></a>GMSA를 사용 하 여 컨테이너 실행

GMSA를 사용 하 여 컨테이너를 실행 하려면 `--security-opt` [docker 실행](https://docs.docker.com/engine/reference/run)의 매개 변수에 자격 증명 사양 파일을 제공 합니다.

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

신뢰할 수 있는 DC 연결 상태 및 신뢰 확인 상태가 아닌 `NERR_Success`경우 문제를 디버그 하는 방법에 대 한 팁을 보려면 [문제 해결](#troubleshooting) 섹션을 참조 하세요.

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

## <a name="orchestrate-containers-with-gmsa"></a>GMSA를 사용 하 여 컨테이너 조정

프로덕션 환경에서는 컨테이너 조정자를 사용 하 여 앱과 서비스를 배포 하 고 관리 하는 경우가 많습니다. 각 orchestrator에는 고유한 관리 패러다임과 Windows 컨테이너 플랫폼에 제공 하는 자격 증명 사양을 수락 하는 책임이 있습니다.

GMSAs를 사용 하는 컨테이너를 오케스트레이션 하는 경우 다음을 확인 합니다.

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

## <a name="example-uses"></a>예제에서는

### <a name="sql-connection-strings"></a>SQL 연결 문자열

서비스가 컨테이너에서 로컬 시스템 또는 네트워크 서비스로 실행되는 경우 Windows 통합 인증을 사용하여 Microsoft SQL Server에 연결할 수 있습니다.

다음은 컨테이너 id를 사용 하 여 SQL Server에 인증 하는 연결 문자열의 예입니다.

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Microsoft SQL Server에서 도메인과 gMSA 이름 다음에 $를 사용하여 로그인을 만듭니다. 로그인을 만든 후에는 데이터베이스의 사용자에 게 추가 하 고 적절 한 액세스 권한을 부여할 수 있습니다.

예제:

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

실행 중인 Microsoft Ignite 2016에서 제공 되는 [기록 데모](https://youtu.be/cZHPz80I-3s?t=2672) 를 확인 하려면 "Containerization에 대 한 경로를 탐색 하 고 컨테이너에 복사"를 참조 하세요.

## <a name="troubleshooting"></a>문제 해결

### <a name="known-issues"></a>알려진 문제

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>컨테이너 hostname은 Windows Server 2016 및 Windows 10, 버전 1709 및 1803의 gMSA 이름과 일치 해야 합니다.

Windows Server 2016, 버전 1709 또는 1803을 실행 중인 경우 컨테이너의 호스트 이름은 gMSA SAM 계정 이름과 일치 해야 합니다.

GMSA 이름과 일치 하지 않는 경우, 인바운드 NTLM 인증 요청 및 이름/SID 번역이 (ASP.NET membership role provider 등의 여러 라이브러리에서 사용 됨)가 실패 합니다. 호스트 이름 및 gMSA 이름이 일치 하지 않는 경우에도 Kerberos는 정상적으로 계속 작동 합니다.

이 제한은 지정 된 호스트 이름에 관계 없이 컨테이너에서 항상 네트워크에 gMSA 이름을 사용 하는 Windows Server 2019에서 해결 되었습니다.

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>두 개 이상의 컨테이너에 gMSA를 사용 하면 Windows Server 2016 및 Windows 10, 버전 1709 및 1803의 간헐적인 오류를 발생 시킬 수 있습니다.

모든 컨테이너는 동일한 호스트 이름을 사용 해야 하므로 두 번째 문제는 windows Server 2019 및 Windows 10 버전 1809 이전 버전에 영향을 줍니다. 여러 컨테이너에 동일한 id 및 호스트 이름이 할당 되 면 두 컨테이너가 같은 도메인 컨트롤러에 동시에 통신할 때 경쟁 조건이 발생할 수 있습니다. 다른 컨테이너가 동일한 도메인 컨트롤러와 통신 하는 경우 동일한 id를 사용 하는 이전 컨테이너와의 통신을 취소 합니다. 이렇게 하면 일시적인 인증 오류가 발생할 수 있으며 컨테이너 내에서 실행할 `nltest /sc_verify:contoso.com` 때 신뢰 오류로 나타날 수 있습니다.

Windows Server 2019의 동작을 변경 하 여 여러 컨테이너에서 동일한 gMSA 동시에 사용할 수 있도록 컴퓨터 이름에서 컨테이너 id를 구분 합니다.

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>GMSAs는 Windows 10 버전 1703, 1709 및 1803의 Hyper-v 격리 컨테이너와 함께 사용할 수 없습니다.

Windows 10 및 Windows Server 버전 1703, 1709 및 1803에서 Hyper-v 격리 컨테이너와 함께 gMSA를 사용 하려고 하면 컨테이너 초기화가 중단 되거나 실패할 수 있습니다.

이 버그는 Windows Server 2019 및 Windows 10, 버전 1809에서 수정 되었습니다. 또한 Windows Server 2016 및 Windows 10, 버전 1607에서 gMSAs를 사용 하 여 Hyper-v 격리 컨테이너를 실행할 수 있습니다.

### <a name="general-troubleshooting-guidance"></a>일반적인 문제 해결 지침

GMSA를 사용 하 여 컨테이너를 실행할 때 오류가 발생 하는 경우 다음 지침을 참조 하 여 근본 원인을 확인할 수 있습니다.

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>호스트가 gMSA를 사용할 수 있는지 확인

1. 호스트가 도메인에 가입 되어 있고 도메인 컨트롤러에 도달할 수 있는지 확인 합니다.
2. RSAT에서 AD PowerShell 도구를 설치 하 고 [ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) 를 실행 하 여 컴퓨터에 gMSA를 검색할 수 있는 권한이 있는지 확인 합니다. Cmdlet이 **False**를 반환 하는 경우 컴퓨터에 gMSA 암호에 대 한 액세스 권한이 없는 것입니다.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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

#### <a name="check-the-firewall-configuration"></a>방화벽 구성 확인

컨테이너 또는 호스트 네트워크에서 엄격한 방화벽 정책을 사용 하는 경우 Active Directory 도메인 컨트롤러나 DNS 서버에 대 한 필수 연결이 차단 될 수 있습니다.

| 프로토콜 및 포트 | 목적 |
|-------------------|---------|
| TCP 및 UDP 53 | DNS |
| TCP 및 UDP 88 | Kerberos |
| TCP 139 | 시키려면 |
| TCP 및 UDP 389 | DAP |
| TCP 636 | LDAP SSL |

컨테이너가 도메인 컨트롤러로 보내는 트래픽 유형에 따라 추가 포트에 대 한 액세스를 허용 해야 할 수 있습니다.
Active directory에 사용 되는 전체 포트 목록은 active [directory 및 Active Directory 도메인 서비스 포트 요구 사항을](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) 참조 하세요.

#### <a name="check-the-container"></a>컨테이너 확인

1. Windows Server 2019 또는 Windows 10, 버전 1809 이전 버전의 Windows를 실행 하는 경우 컨테이너 호스트 이름은 gMSA 이름과 일치 해야 합니다. `--hostname` 매개 변수가 gMSA 약식 이름 (예: "webapp01.contoso.com"이 아닌 "webapp01")과 일치 하는지 확인 합니다.

2. 컨테이너 네트워킹 구성을 확인 하 여 컨테이너가 gMSA의 도메인에 대 한 도메인 컨트롤러를 확인 하 고 액세스할 수 있는지 확인 합니다. 컨테이너에서 잘못 구성 된 DNS 서버는 id 문제의 일반적인 원인입니다.

3. 컨테이너에 다음 cmdlet을 실행 하 여 컨테이너의 도메인에 대 한 올바른 연결이 있는지 확인 합니다 (사용 `docker exec` 또는 해당 하는 경우).

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    GMSA을 사용할 수 있고 `NERR_SUCCESS` 네트워크 연결을 통해 컨테이너가 도메인과 통신할 수 있는지에 따라 신뢰 확인이 반환 됩니다. 실패 하는 경우 호스트와 컨테이너의 네트워크 구성을 확인 합니다. 둘 다 도메인 컨트롤러와 통신할 수 있어야 합니다.

4. 앱이 gMSA를 [사용 하도록 구성](#configure-your-application-to-use-the-gmsa)되어 있는지 확인 합니다. GMSA를 사용 하는 경우 컨테이너 내부의 사용자 계정이 변경 되지 않습니다. 대신 시스템 계정은 다른 네트워크 리소스와 통신 하는 경우 gMSA를 사용 합니다. 즉, gMSA id를 활용 하려면 앱이 네트워크 서비스 또는 로컬 시스템으로 실행 되어야 합니다.

    > [!TIP]
    > 컨테이너에서 현재 `whoami` 사용자 컨텍스트를 식별 하기 위해 다른 도구를 실행 하거나 사용 하는 경우 gMSA 이름이 표시 되지 않습니다. 이는 도메인 id 대신 로컬 사용자로 항상 컨테이너에 로그인 하기 때문입니다. GMSA는 네트워크 리소스와 통신 하는 경우 컴퓨터 계정에서 사용 되며,이 경우 앱을 네트워크 서비스 또는 로컬 시스템으로 실행 해야 합니다.

5. 마지막으로, 컨테이너가 올바르게 구성 된 것 처럼 보이지만 사용자 또는 다른 서비스가 containerized 앱에 자동으로 인증할 수 없는 경우 gMSA 계정에서 Spn을 확인 하세요. 클라이언트는 응용 프로그램에 도달 하는 이름으로 gMSA 계정을 찾습니다. 예를 들어 클라이언트가 부하 분산 장치 또는 `host` 다른 DNS 이름을 통해 앱에 연결 하는 경우 gMSA에 대 한 추가 spn이 필요할 수 있습니다.

## <a name="additional-resources"></a>추가 리소스

- [그룹 관리 서비스 계정 개요](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
