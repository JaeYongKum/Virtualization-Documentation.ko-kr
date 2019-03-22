---
title: Windows 컨테이너에 대 한 그룹 관리 서비스 계정
description: Windows 컨테이너에 대 한 그룹 관리 서비스 계정
keywords: docker, 컨테이너, active directory, gmsa
author: rpsqrd
ms.date: 03/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 5f80d245984b0cf5c4503971a74cc8bbcca0c19c
ms.sourcegitcommit: f53b8b3dc695cdf22106095b15698542140ae088
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/22/2019
ms.locfileid: "9257410"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Windows 컨테이너에 대 한 그룹 관리 서비스 계정

Windows 기반 네트워크에서는 인증 및 권한 부여 사용자, 컴퓨터 및 기타 네트워크 리소스 간에 용이 하 게 Active Directory (AD)를 사용 하 여 괜찮습니다. 엔터프라이즈 응용 프로그램 개발자는 종종 AD 통합 하 고 손쉽게 사용자 및 기타 서비스에 자동으로 로그인 하도록 통합 Windows 인증을 활용 하기 위해 도메인에 가입 된 서버에서 실행 되도록 앱 디자인 자신의 id 사용 하 여 응용 프로그램입니다.

Windows 컨테이너는 도메인에 가입 될 수, 다양 한 인증 시나리오를 지원 하도록 Active Directory 도메인 id를 여전히 사용할 수 있습니다.
이 위해 Windows 컨테이너를 사용 하 여 [그룹 관리 서비스 계정](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA)-실행을 구성할 수는 특수 한 형식의 여러 컴퓨터가 필요 없이 id를 공유할 수 있도록 설계 된 Windows Server 2012에서 도입 된 서비스 계정 암호를 알아야 합니다.
GMSA를 사용 하 여 컨테이너를 실행할 때 컨테이너 호스트는 Active Directory 도메인 컨트롤러에서 gMSA 암호를 검색 하 고 컨테이너 인스턴스를 제공 합니다.
컨테이너의 컴퓨터 계정 (SYSTEM) 네트워크 리소스에 액세스 해야 할 때마다 gMSA 자격 증명을 사용 합니다.

이 문서에서는 Windows 컨테이너를 사용 하 여 Active Directory 그룹 관리 서비스 계정을 사용 하 여 시작 하는 방법을 설명 합니다.

## <a name="prerequisites"></a>필수 구성 요소

그룹 관리 서비스 계정을 사용 하 여 Windows 컨테이너를 실행 하려면 다음이 필요 합니다.

**Windows Server 2012 이상을 실행 하는 하나 이상의 도메인 컨트롤러를 사용 하 여 Active Directory 도메인입니다.**
도메인 또는 포리스트 기능 수준 요구 사항이 없는 계정을 위임할을 사용 하는 Windows Server 2012를 실행 하는 도메인 컨트롤러에서 분산 이상 gMSA 암호 될 수 있습니다.
자세한 내용은 [계정을 위임할 Active Directory 요구 사항을](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)참조 하세요.



**GMSA 계정을 만들 수 있는 권한입니다.**
도메인 관리자 또는 된 계정을 사용 해야 gMSA 계정을 만들려면 *개체 만들기를 GroupManagedServiceAccount* 권한을 위임 합니다.



**CredentialSpec PowerShell 모듈을 다운로드 하려면 인터넷에 액세스 합니다.**
연결이 끊긴된 환경에서 작업할 수 있습니다 [모듈 저장 하 고](https://docs.microsoft.com/en-us/powershell/module/powershellget/save-module?view=powershell-5.1) 인터넷을 사용 하 여 컴퓨터에 액세스 하 고 개발 컴퓨터 또는 컨테이너 호스트에 복사 합니다.

## <a name="one-time-preparation-of-active-directory"></a>Active Directory의 일회성 준비

아직 만들지 않은 gMSA 도메인, 키 배포 서비스 루트 키 생성 해야 할 가능성이 있습니다.
KDS는 만들기, 회전 및 권한 있는 호스트에 gMSA 암호를 해제 해야 합니다.
컨테이너 호스트 컨테이너를 실행 하려면 gMSA를 사용 하는 경우 현재 암호를 검색 KDS 연락 합니다.

KDS 루트 키 하는지 확인 하기 이미 만들어진 AD PowerShell 도구가 설치 된 도메인 컨트롤러 또는 도메인 컨트롤러에서 *도메인 관리자* 는 다음 PowerShell 명령을 실행 합니다.

```powershell
Get-KdsRootKey
```

명령 키를 반환 하는 경우 ID를 모두 설정 하 고 [그룹 관리 서비스 계정 만들기](#create-a-group-managed-service-account) 단계로 넘어갈 수 있습니다.
그렇지 않은 경우 KDS 루트 키에 계속 합니다.

프로덕션 환경 또는 여러 도메인 컨트롤러를 사용 하 여 테스트 환경에서 *도메인 관리자* KDS 루트 키를 PowerShell에서 다음 명령을 실행 합니다.
명령 키 즉시 적용 됩니다 처럼, 10 시간의 KDS 루트 키를 복제 하 고 모든 도메인 컨트롤러에서 사용할 수 있는 하기 전에 대기 해야 합니다.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

도메인의 도메인 컨트롤러가 하나만 있는 경우 10 시간 전에 적용 되기 키를 설정 하 여 프로세스를 촉진할 수 있습니다.
프로덕션 환경에서이 기술을 사용 하지 마세요.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>그룹 관리 서비스 계정 만들기

Windows 통합 인증을 사용 하는 모든 컨테이너 gMSA를 하나 이상 있어야 합니다.
*시스템* 또는 *네트워크 서비스로* 실행 되는 앱에서 네트워크 리소스에 액세스할 때마다 기본 gMSA 사용 됩니다.
GMSA의 이름에는 컨테이너에 할당 된 호스트 이름에 관계 없이 네트워크에 컨테이너의 이름이 될 것입니다.
컨테이너 컴퓨터 계정에서 다른 id로 컨테이너에서 서비스 또는 응용 프로그램을 실행 하려면 하는 경우 추가 계정을 위임할을 사용 하 여 컨테이너를 구성할 수도 있습니다.

GMSA를 만들 경우 동시에 여러 컴퓨터에서 사용할 수 있는 공유 id를 만들게 됩니다.
Active Directory 액세스 제어 목록에서 gMSA 암호에 대 한 액세스 보호 됩니다.
각 gMSA 계정에 대 한 보안 그룹 만들기 및 암호에 대 한 액세스를 제한 하려면 관련 컨테이너 호스트 보안 그룹에 추가 하는 것이 좋습니다.

마지막으로, 컨테이너는 모든 서비스 사용자 이름 (SPN)를 자동으로 등록 하지 않으면, 이후를 수동으로 최소한 "호스트" SPN gMSA 계정을 생성 해야 합니다.
일반적으로 호스트 또는 http SPN 동일한 이름을 사용 하 여 gMSA 계정으로 등록 되어 있지만 클라이언트가 부하 분산 장치 또는 다른 gMSA 이름에서 DNS 이름 뒤에서 컨테이너 화 응용 프로그램에 액세스 하는 경우 다른 서비스 이름을 사용 해야 할 수 있습니다.
예를 들어 gMSA 계정을 *WebApp01* 이지만 사용자가 *mysite.contoso.com* 에서 사이트에 액세스 하는 경우를 등록 해야는 `http/mysite.contoso.com` gMSA 계정의 SPN 합니다.
일부 응용 프로그램의 고유한 프로토콜에 대 한 추가 Spn 필요할 수 있습니다. 즉, 예를 들어, SQL 서버 "MSSQLSvc/호스트 이름이" SPN에 필요 합니다.

다음 표에서 gMSA를 만들 때 필요한 특성을 나열 합니다.

gMSA 속성 | 필요한 값 | 예
--------------|----------------|--------
이름 | 모든 유효한 계정 이름입니다. | `WebApp01`
DnsHostName | 계정 이름에 추가 된 도메인 이름입니다. | `WebApp01.contoso.com`
ServicePrincipalNames | 최소한 SPN 호스트 설정, 필요에 따라 다른 프로토콜을 추가 합니다. | `'host/WebApp01', 'host/WebApp01.contoso.com'`
PrincipalsAllowedToRetrieveManagedPassword | 컨테이너 호스트를 포함 하는 보안 그룹. | `WebApp01Hosts`

GMSA에 호출 하려는 알면 보안 그룹 및 gMSA를 만드는 PowerShell에서 다음 명령을 실행 합니다.

> [!TIP]
> 사용 해야 합니다. 또는 **Domain Admins** 보안 그룹에 속하는 된 계정을 위임 **개체 만들기를 GroupManagedServiceAccount** 사용 권한을 다음 명령을 실행 합니다.
> [새로 만들기-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) cmdlet은 [원격 서버 관리 도구](https://aka.ms/rsat)에서 AD PowerShell 도구의 일부입니다.

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

개발, 테스트 및 프로덕션 환경에 대 한 별도 gMSA 계정을 만드는 것이 좋습니다.

## <a name="prepare-your-container-host"></a>컨테이너 호스트 준비

GMSA를 사용 하 여 Windows 컨테이너를 실행 되는 각 컨테이너 호스트 도메인 가입 되 고 검색 gMSA 암호에 액세스할 수 이어야 합니다.

1.  Active Directory 도메인에 컴퓨터를 가입 합니다.
2.  호스트에 gMSA 암호에 대 한 액세스를 제어 하는 보안 그룹에 속해 있는지 확인 합니다.
3.  컴퓨터를 다시 시작 하는 새 그룹 구성원을 가져옵니다.
4.  [Windows 10 용 Docker 데스크톱](https://docs.docker.com/docker-for-windows/install/) 또는 [Windows Server에 대 한 Docker를](https://docs.docker.com/install/windows/docker-ee/)설정 합니다.
5.  (권장) 호스트 [ADServiceAccount 테스트를](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount)실행 하 여 gMSA 계정을 사용할 수 있는지 확인 합니다. 명령 **False**를 반환 하는 경우 진단 단계 [문제 해결](#troubleshooting) 섹션을 참조 하세요.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>자격 증명 사양 만들기

자격 증명 사양 파일 gMSA 계정 사용 하 여 컨테이너에 대 한 메타 데이터가 포함 된 JSON 문서입니다.
Identity 구성 컨테이너 이미지를 분리를 유지 하 여 자격 증명 사양 파일 필요한 코드 변경 간단 하 게 바꾸어 컨테이너에 의해 사용 되는 gMSA를 쉽게 변경할 수 있습니다.

자격 증명 사양 파일 [CredentialSpec PowerShell 모듈](https://aka.ms/credspec) 을 사용 하 여 도메인에 가입 된 컨테이너 호스트에 만들어집니다.
파일을 만든 후 컨테이너 조정자 또는 다른 컨테이너 호스트에 복사할 수 있습니다.
자격 증명 사양 파일 컨테이너 호스트 컨테이너를 대신 하 여 gMSA를 검색 하므로 gMSA 암호 등의 모든 비밀 정보를 포함 되지 않습니다.

Docker는 Docker 데이터 디렉터리에 **CredentialSpecs** 디렉터리에서 자격 증명 사양 파일을 찾은 필요 합니다.
기본 설치에서이 폴더에서 찾을 수 `C:\ProgramData\Docker\CredentialSpecs`.

컨테이너 호스트에서 자격 증명 사양 파일을 생성 하려면 다음 단계를 수행 합니다.
1.  RSAT AD PowerShell 도구를 설치
    -   Windows server에서 실행 `Install-WindowsFeature RSAT-AD-PowerShell`
    -   Windows 10, 버전 1809 이상 실행 `Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'`
    -   이전 버전의 Windows 10을 참조 하세요.https://aka.ms/rsat
2.  최신 버전의 [CredentialSpec PowerShell 모듈](https://aka.ms/credspec)을 설치 합니다.

    ```powershell
    Install-Module CredentialSpec
    ```

    컨테이너 호스트에 인터넷 액세스 권한이 없으면 실행 `Save-Module CredentialSpec` 인터넷에 연결 된 컴퓨터에서 모듈 폴더를 복사 하 고 `C:\Program Files\WindowsPowerShell\Modules` 또는 다른 위치에 `$env:PSModulePath` 컨테이너 호스트에서.

3.  새 자격 증명 사양 파일을 만듭니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    기본적으로 cmdlet에 제공 된 gMSA 이름을 컨테이너에 대 한 컴퓨터 계정을 사용 하 고 자격 증명 사양을 만들어집니다.
    파일의 파일 이름에 대 한 gMSA 도메인 및 계정 이름을 사용 하 여 Docker CredentialSpecs 디렉터리에 저장 됩니다.

    컨테이너에서 보조 gMSA로 서비스 또는 프로세스를 실행 하는 경우 추가 gMSA 계정을 포함 하는 자격 증명 사양을 만들 수 있습니다.
    이 위해 사용 하 여는 `-AdditionalAccounts` 매개 변수:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    지원 되는 매개 변수의 전체 목록은, 실행 `Get-Help New-CredentialSpec`.

4.  모든 자격 증명 사양 및 다음 명령 사용 하 여의 전체 경로 목록을 표시할 수 있습니다.

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configuring-your-application-to-use-the-gmsa"></a>GMSA를 사용 하 여 응용 프로그램을 구성

일반적인 구성에서 컨테이너는 컨테이너 컴퓨터 계정을 네트워크 리소스에 인증 하려고 할 때마다 사용 되는 하나의 gMSA 계정은 지정 합니다.
즉, 앱 gMSA id를 사용 하는 경우 **로컬 시스템** 또는 **네트워크 서비스로** 실행 해야 합니다.

### <a name="run-an-iis-app-pool-as-network-service"></a>IIS 응용 프로그램 풀을 네트워크 서비스로 실행

컨테이너에는 IIS 웹 사이트를 호스팅하는 gMSA를 활용 하기 위해 수행 해야 하는 응용 프로그램 풀 id **네트워크 서비스**에 설정 됩니다.
에서 할 수 있는 Dockerfile에 다음 명령을 추가 하 여:

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

이전에 정적 사용자 자격 증명을 사용 하는 IIS 응용 프로그램 풀에 대 한 경우 해당 자격 증명에 대 한 대체 gMSA를 고려 합니다.
컨테이너 이미지를 변경 하지 않고 현재 id IIS가 자동으로 선택 및 개발, 테스트 및 프로덕션 환경 간에 gMSA를 변경할 수 있습니다.

### <a name="run-a-windows-service-as-network-service"></a>Windows 서비스 네트워크 서비스로 실행

컨테이너 화 된 응용 프로그램 Windows 서비스로 실행 하는 경우 **네트워크 서비스** 에 Dockerfile의로 실행할 서비스를 설정할 수 있습니다.

```dockerfile
RUN cmd /c 'sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""'
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>임의의 콘솔 앱 네트워크 서비스로 실행

하지 IIS 또는 서비스 관리자에서 호스팅되는 일반 콘솔 앱에 대 한는 앱 gMSA 컨텍스트를 자동으로 상속 되도록 컨테이너 **네트워크** 서비스로 실행 하는 가장 쉬운 경우가 많습니다.
이 기능은 Windows Server 버전 1709부터 사용할 수 있습니다.

기본적으로 네트워크 서비스로 실행 하 여 Dockerfile에 다음 줄을 추가 합니다.

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

또한에 연결할 수 있습니다 컨테이너에 네트워크 서비스로 사용 하 여 한 번만 `docker exec`.
이 컨테이너 네트워크 서비스로 정상적으로 실행 되지 않을 때에 실행 중인 컨테이너에 연결 문제를 해결 하는 경우에 특히 유용 합니다.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>GMSA를 사용 하 여 컨테이너를 실행 합니다.

GMSA를 사용 하 여 컨테이너를 실행 하려면 자격 증명 사양 파일을 제공 합니다 `--security-opt` [docker 실행](https://docs.docker.com/engine/reference/run)의 매개 변수:

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Windows Server 2016에서 1709 및 1803은 컨테이너 *와 일치 해야* gMSA의 호스트 이름을 간단한 합니다.
위의 예제에서는 gMSA SAM 계정 이름 이므로 "webapp01" 컨테이너 호스트 이름을 동일 하 게 설정 됩니다.
Windows Server 2019 이상, 호스트 이름 필드 필요 하지 않고 컨테이너 여전히 식별 하는 자체 호스트 이름 대신 gMSA 이름으로 다른 명시적으로 제공 하는 경우에 합니다.

GMSA 제대로 작동 하는 경우를 확인 하려면 컨테이너에 다음 명령을 실행 합니다.

```
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

*신뢰할 수 있는 DC 연결 상태* 및 *신뢰 검증 상태* 없는 경우 *NERR_Success*문제를 디버그 하는 방법에 대 한 팁 [문제 해결](#troubleshooting) 섹션을 확인 합니다.

다음 명령을 실행 하 고 클라이언트 이름을 확인 하 여 컨테이너 내에서 gMSA id를 확인할 수 있습니다.

```
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

PowerShell 또는 gMSA 계정으로 다른 콘솔 응용 프로그램을 열려면 시스템 일반 ContainerAdministrator (또는 NanoServer에 대 한 ContainerUser) 대신 계정에서 실행 되도록 컨테이너를 요청할 수 있습니다.

```powershell
# NOTE: you can only run as SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\SYSTEM" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

시스템으로를 실행 중인 경우 gMSA로 도메인 컨트롤러에 SYSVOL에 연결 하 여 네트워크 인증을 테스트할 수 있습니다.

```
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrating-containers-with-gmsa"></a>GMSA 사용 하 여 컨테이너를 조정합니다.

프로덕션 환경에서 배포 하 고 앱 및 서비스 관리 컨테이너 조정자를 자주 사용 합니다.
각 orchestrator 자체 관리 패러다임 있으며 자격 증명 사양 Windows 컨테이너 플랫폼을 제공 하도록 허용 하는 것에 대 한 책임이 있습니다.

이 계정을 위임할을 사용 하 여 컨테이너를 오케스트레이션 하면는 있는지 확인 합니다.
> [!div class="checklist"]
> * 이 계정을 위임할을 사용 하 여 컨테이너를 실행 하 여 예약할 수 있는 모든 컨테이너 호스트는 도메인에 가입
> * 컨테이너에서 사용 되는 모든이 계정을 위임할에 대 한 암호를 검색할 수 있는 권한이 컨테이너 호스트
> * 자격 증명 사양 파일이 작성 및 orchestrator에 업로드 되거나 처리 조정자 선호 하는 방법에 따라 모든 컨테이너 호스트에 복사 합니다.
> * 컨테이너 네트워크를 검색 gMSA 티켓 Active Directory 도메인 컨트롤러와 통신 하는 컨테이너 허용

### <a name="using-gmsa-with-service-fabric"></a>서비스 패브릭으로 gMSA를 사용 하 여

Service Fabric 응용 프로그램 매니페스트에서 자격 증명 사양 위치를 지정 하면 gMSA 사용 하 여 실행 중인 Windows 컨테이너를 지원 합니다.
자격 증명 사양 파일을 만들고 서비스 패브릭에서 찾을 수 있도록 각 호스트에서 Docker 데이터 디렉터리의 **CredentialSpecs** 하위 디렉터리에 배치 해야 합니다.
`Get-CredentialSpec` cmdlet을 [CredentialSpec PowerShell 모듈](https://aka.ms/credspec)일부 정확한 위치에 자격 증명 사양 확인 하기 위해 사용할 수 있습니다.

참조 [빠른 시작: Windows 컨테이너 서비스 패브릭 배포](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-quickstart-containers) 응용 프로그램을 구성 하는 방법에 대 한 자세한 내용은 [서비스 패브릭에서 실행 되는 Windows 컨테이너에 대 한 gMSA를 설정](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) 합니다.

### <a name="using-gmsa-with-docker-swarm"></a>GMSA를 사용 하 여 Docker Swarm을 통해

Docker Swarm에서 관리 하는 컨테이너를 사용 하 여 gMSA를 사용 하려면 [docker 서비스를 만들](https://docs.docker.com/engine/reference/commandline/service_create/) 명령을 사용 하 여 사용 하 여는 `--credential-spec` 매개 변수:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Docker 서비스를 사용 하 여 자격 증명 사양을 사용에 대 한 자세한 내용은 [Docker Swarm 예제](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) 를 참조 하세요.

### <a name="using-gmsa-with-kubernetes"></a>GMSA를 사용 하 여 Kubernetes를 사용한

Kubernetes에서이 계정을 위임할와 Windows 컨테이너 예약에 대 한 지원은 Kubernetes 1.14 부터는 알파 지원 됩니다.
이 기능에 대 한 최신 정보 및 Kubernetes 배포에서 테스트 하는 방법에 대 한 정보에 대 한 [컨테이너 Identity 웹에 대 한 Windows 그룹 관리 서비스 계정](https://github.com/kubernetes/enhancements/blob/master/keps/sig-windows/20181221-windows-group-managed-service-accounts-for-container-identity.md) 을 확인 합니다.

## <a name="example-uses"></a>사용 예

### <a name="sql-connection-strings"></a>SQL 연결 문자열
서비스가 컨테이너에서 로컬 시스템 또는 네트워크 서비스로 실행되는 경우 Windows 통합 인증을 사용하여 Microsoft SQL Server에 연결할 수 있습니다.

SQL Server에 인증 하는 컨테이너 id를 사용 하는 연결 문자열의 예는 다음과 같습니다.

```
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Microsoft SQL Server에서 도메인과 gMSA 이름 다음에 $를 사용하여 로그인을 만듭니다.
만든 로그인을 데이터베이스의 사용자에 추가하고 적절한 액세스 권한을 부여할 수 있습니다.

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

작동을 확인하려면 Microsoft Ignite 2016의 "Walk the Path to Containerization - transforming workloads into containers"(컨테이너화 안내 - 워크로드를 컨테이너로 변환) 세션에서 사용 가능한 [녹화된 데모](https://youtu.be/cZHPz80I-3s?t=2672)를 참조하세요.

## <a name="troubleshooting"></a>문제 해결

### <a name="known-issues"></a>알려진 문제

**컨테이너 호스트 이름 WS2016, 1709 및 1803에 대 한 gMSA 이름과 일치 해야 합니다.**

Windows Server 2016을 실행 중인 경우 1709 또는 1803 컨테이너의 호스트 이름 일치 해야 SAM 계정 이름에 gMSA 합니다.
호스트 이름 gMSA 이름과 일치 하지 않으면 NTLM 인증 요청 및 (ASP.NET 역할 제한과 같은 많은 라이브러리에서 사용) 이름/SID 번역 인바운드 실패 합니다.
Kerberos 계속 호스트와 일치 하지 않는 경우에 정상적으로 작동 합니다.

이러한 제한 있는 컨테이너는 이제 항상 사용 gMSA 이름 할당 된 호스트 이름에 관계 없이 네트워크에서 Windows Server 2019에서 해결 되었습니다.

**동시에 둘 이상의 컨테이너를 사용 하 여 gMSA를 사용 하 WS2016, 1709 및 1803에 일시적인 오류가 발생 합니다.**

두 번째 문제는 동일한 호스트 이름을 사용 하는 모든 컨테이너를 요구 하는 이전 제한으로 인해 버전을의 Windows Server 2019 하기 전에 Windows 및 Windows 10 버전 1809에 영향을 줍니다.
여러 컨테이너가 동일한 id 및 호스트 이름에 할당 되는 경합 상태 두 컨테이너 동일한 도메인 컨트롤러에 동시에 통신할 때 발생할 수 있습니다.
다른 컨테이너와 동일한 도메인 컨트롤러를 통신 하는 경우 통신 동일한 id를 사용 하 여 모든 이전 컨테이너를 사용 하 여 아니면 취소 됩니다.
일시적인 인증 오류가 발생할 수 있으므로이 고 때로는 관찰할 수는 트러스트 실패도 실행 하면 `nltest /sc_verify:contoso.com` 컨테이너 내에서 합니다.

우리 동작을 변경 컴퓨터 이름, 컨테이너 id를 분리 하 여 Windows Server 2019의 여러 컨테이너가 동일한 gMSA를 동시에 사용할 수 있도록 합니다.

**Windows 버전 1703, 1709 및 1803에 Hyper-v 격리 된 컨테이너를 사용 하 여이 계정을 위임할을 사용할 수 없습니다.**

컨테이너 초기화 중단 또는 Windows 10 및 Windows Server 버전 1703, 1709 및 1803에 Hyper-v 격리 된 컨테이너를 사용 하 여 gMSA를 사용 하려고 하면 실패 합니다.

이 버그 Windows Server 2019 및 Windows 10, 버전 1809에서 해결 되었습니다. Windows Server 2016 및 Windows 10 버전 1607에서이 계정을 위임할을 사용 하 여 Hyper-v 격리 된 컨테이너를 실행할 수 있습니다.

### <a name="general-troubleshooting-guidance"></a>일반적인 문제 해결 지침

GMSA를 사용 하 여 컨테이너를 실행할 때 오류가 발생 하는 경우 다음 단계를 근본 원인을 파악 하면 도움이 될 수 있습니다.

**호스트 gMSA를 사용할 수 있는지 확인**

1.  호스트는 도메인 가입 및 도메인 컨트롤러에 연결할 수를 확인 합니다.
2.  Rsat에서 AD PowerShell 도구를 설치 하 고 [테스트 ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount) 확인 컴퓨터에 gMSA 검색에 대 한 액세스를 실행 합니다. 컴퓨터에 gMSA 암호에 액세스할 수 없는 경우 cmdlet **False**를 반환 합니다.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```
3.  테스트 ADServiceAccount **False**를 반환 하는 경우 호스트 gMSA 암호를 검색할 수 있는 권한이 있는 보안 그룹에 속하는지 확인 합니다.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4.  호스트에 gMSA 암호를 검색할 수 있는 보안 그룹에 속한 하지만 여전히 테스트 ADServiceAccount 실패 하는 경우를 가져오려면 현재 그룹 구성원을 반영 하는 새 티켓에 대 한 컴퓨터를 다시 시작 해야 합니다.

**자격 증명 사양 파일 검사**

1.  실행 `Get-CredentialSpec` 컴퓨터의 사양 자격 증명 모든 [CredentialSpec PowerShell 모듈](https://aka.ms/credspec) 을 찾습니다. 자격 증명 사양은 Docker 루트 디렉터리에서 "CredentialSpecs" 디렉터리에 저장 되어야 합니다. 실행 하 여 루트 디렉터리는 Docker를 찾을 수 있습니다 `docker info -f "{{.DockerRootDir}}"`.
2.  CredentialSpec 파일을 열고 다음 필드를 올바르게 입력 된 있는지 확인 합니다.
    -   **Sid**: gMSA 계정 SID
    -   **MachineAccountName**: gMSA SAM 계정 이름 (전체 도메인 이름 또는 달러 기호 포함 않음)
    -   **DnsTreeName**: AD 포리스트의 FQDN
    -   **DnsName**: gMSA 속한 도메인의 FQDN
    -   **NetBiosName**: gMSA 속해 있는 도메인에 대 한 NETBIOS 이름
    -   **GroupManagedServiceAccounts/이름**: gMSA SAM 계정 이름 (전체 도메인 이름 또는 달러 기호 포함 않음)
    -   **GroupManagedServiceAccounts/범위**: FQDN 도메인 및 NETBIOS에 대 한 항목

    전체 자격 증명 사양에 대 한 아래 전체 예제를 참조 하세요.

    ```json
    {
    "CmsPlugins":[
        "ActiveDirectory"
    ],
    "DomainJoinConfig":{
        "Sid":"S-1-5-21-702590844-1001920913-2680819671",
        "MachineAccountName":"webapp01",
        "Guid":"56d9b66c-d746-4f87-bd26-26760cfdca2e",
        "DnsTreeName":"contoso.com",
        "DnsName":"contoso.com",
        "NetBiosName":"CONTOSO"
    },
    "ActiveDirectoryConfig":{
        "GroupManagedServiceAccounts":[
        {
            "Name":"webapp01",
            "Scope":"contoso.com"
        },
        {
            "Name":"webapp01",
            "Scope":"CONTOSO"
        }
        ]
    }
    }
    ```

3.  오케스트레이션 솔루션에 대 한 올바른 자격 증명 사양 파일의 경로를 확인 합니다. Docker를 사용 하는 경우 명령을 실행 하는 컨테이너에 있는지 확인 `--security-opt="credentialspec=file://NAME.json"`, 여기서 "NAME.json" **Get CredentialSpec**하 여 이름 출력으로 대체 됩니다. 이 이름은 Docker 루트 디렉터리에서 CredentialSpecs 폴더를 기준으로 플랫 파일 이름입니다.

**컨테이너를 확인 합니다.**

1.  버전의 Windows Server 2019 하기 전에 Windows 또는 Windows 10, 버전 1809 실행 하는 경우 컨테이너 호스트에 gMSA 이름과 일치 해야 합니다. 확인 합니다 `--hostname` 매개 변수와 일치 gMSA 짧은 이름 (도메인 구성 요소가 없는, 예: "webapp01" 하지 "webapp01.contoso.com").

2.  컨테이너를 확인 하려면 컨테이너 네트워킹 구성을 해결 및 gMSA의 도메인에 대 한 도메인 컨트롤러에 액세스할 수를 확인 합니다. 잘못 구성 된 DNS 서버는 컨테이너에서 발생 id 문제로의 일반적인 원인 됩니다.

3.  경우 컨테이너에 대 한 유효한 연결 도메인 컨테이너에서 다음 명령을 실행 하 여 확인 (사용 하 여 `docker exec` 또는 이와 동등한):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    GMSA를 사용할 수 있고 네트워크 연결 도메인에 연락 하는 컨테이너 트러스트 검증 NERR_SUCCESS를 반환 해야 합니다.
    실패 하면 호스트와 컨테이너의 네트워크 구성을 확인-모두 도메인 컨트롤러와 통신할 수 있도록 해야 합니다.

4.  앱은 [gMSA를 사용 하도록 구성](#configuring-your-application-to-use-the-gmsa)확인 합니다. 다른 네트워크 리소스와 통신 하는 경우 시스템 계정이 gMSA를 사용 하는 대신,-gMSA를 사용 하는 경우에 컨테이너 내에서 사용자 계정 변경 되지 않습니다. 따라서 앱 gMSA id를 활용 하 여 네트워크 서비스 또는 로컬 시스템으로 실행 해야 합니다.

    > [!TIP]
    > 실행 하는 경우 `whoami` 또는 다른 도구를 사용 하 여 시도 하 고 컨테이너에서 현재 사용자 컨텍스트를 식별, 자체 gMSA 이름 표시 되지 것입니다.
    > 항상에 로그인 하면 컨테이너는 도메인 id – 로컬 사용자로 때문입니다.
    > GMSA 것에 통신 네트워크 리소스에는 로컬 시스템 또는 네트워크 서비스로 실행 해야 하는 이유 때마다 컴퓨터 계정에서 사용 됩니다.

5.  마지막으로 컨테이너를 올바르게 구성 되어야 보이지만 사용자나 다른 서비스 컨테이너 화 된 응용 프로그램에 자동으로 인증할 수 없는 경우 계좌 gMSA에 Spn을 확인 합니다. 클라이언트는 응용 프로그램에 도달 하기는 이름으로 gMSA 계정을 찾습니다. 추가 필요할 즉 `host` 클라이언트가 부하 분산 장치 또는 다른 DNS 이름을 통해 앱에 연결 하는 예를 들어 경우 gMSA에 Spn 합니다.

## <a name="additional-resources"></a>추가 리소스
-   [그룹 관리 서비스 계정 개요](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
