---
title: Windows 컨테이너에 대해 gMSAs 만들기
description: Windows 컨테이너에 대해 gMSAs (그룹 관리 서비스 계정)를 만드는 방법
keywords: docker, 컨테이너, active directory, gmsa, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정
author: rpsqrd
ms.date: 01/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 36061cfc491dd9dd581d1e6bce92a29e4a6f217d
ms.sourcegitcommit: 530712469552a1ef458883001ee748bab2c65ef7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/26/2020
ms.locfileid: "77628938"
---
# <a name="create-gmsas-for-windows-containers"></a>Windows 컨테이너에 대해 gMSAs 만들기

Windows 기반 네트워크는 일반적으로 AD (Active Directory)를 사용 하 여 사용자, 컴퓨터 및 기타 네트워크 리소스 간의 인증 및 권한 부여를 용이 하 게 합니다. 엔터프라이즈 응용 프로그램 개발자는 Windows 통합 인증을 활용 하기 위해 Windows 통합 인증을 사용 하 여 Windows 통합 인증을 사용 하는 Windows 통합 인증을 사용 하는 응용 프로그램을 설계 하는 경우가 종종 있습니다. 해당 id를 사용 하는 응용 프로그램입니다.

Windows 컨테이너는 도메인에 가입할 수 없지만 Active Directory 도메인 id를 사용 하 여 다양 한 인증 시나리오를 지원할 수 있습니다.

이렇게 하려면 gMSA ( [그룹 관리 서비스 계정) (그룹 관리 서비스 계정](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) )을 사용 하 여 실행 되도록 windows 컨테이너를 구성할 수 있습니다 .이 계정에는 여러 컴퓨터 2012에서 암호를 알 필요 없이 id를 공유할 수 있도록 설계 되었습니다.

GMSA를 사용 하 여 컨테이너를 실행 하는 경우 컨테이너 호스트는 Active Directory 도메인 컨트롤러에서 gMSA 암호를 검색 하 여 컨테이너 인스턴스에 제공 합니다. 컨테이너는 컴퓨터 계정 (시스템)이 네트워크 리소스에 액세스 해야 할 때마다 gMSA 자격 증명을 사용 합니다.

이 문서에서는 Windows 컨테이너에서 그룹 관리 서비스 계정을 사용 하 Active Directory를 시작 하는 방법을 설명 합니다.

## <a name="prerequisites"></a>필수 조건

그룹 관리 서비스 계정을 사용 하 여 Windows 컨테이너를 실행 하려면 다음이 필요 합니다.

- Windows Server 2012 이상을 실행 하는 도메인 컨트롤러가 하나 이상 있는 Active Directory 도메인 GMSAs를 사용 하는 포리스트 또는 도메인 기능 수준 요구 사항은 없지만 gMSA 암호는 Windows Server 2012 이상을 실행 하는 도메인 컨트롤러 에서만 배포할 수 있습니다. 자세한 내용은 [gMSAs에 대 한 Active Directory 요구 사항](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)을 참조 하세요.
- GMSA 계정을 만들 수 있는 권한입니다. GMSA 계정을 만들려면 도메인 관리자 이거나 *Create의 msds-primary-computer-msds-groupmanagedserviceaccount objects* 권한이 위임 된 계정을 사용 해야 합니다.
- 인터넷에 액세스 하 여 CredentialSpec PowerShell 모듈을 다운로드 합니다. 연결 되지 않은 환경에서 작업 하는 경우 인터넷에 액세스할 수 있는 컴퓨터에 [모듈을 저장](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) 하 고 개발 컴퓨터 또는 컨테이너 호스트에 복사할 수 있습니다.

## <a name="one-time-preparation-of-active-directory"></a>Active Directory의 일회성 준비

도메인에 gMSA를 아직 만들지 않은 경우 KDS (키 배포 서비스) 루트 키를 생성 해야 합니다. KDS는 gMSA 암호를 생성 하 고, 회전 하 고, 권한 있는 호스트에 릴리스 하는 일을 담당 합니다. 컨테이너 호스트에서 컨테이너를 실행 하는 데 gMSA를 사용 해야 하는 경우 KDS에 연결 하 여 현재 암호를 검색 합니다.

KDS 루트 키가 이미 생성 되었는지 확인 하려면 AD PowerShell 도구가 설치 된 도메인 컨트롤러 또는 도메인 구성원에 대해 도메인 관리자 권한으로 다음 PowerShell cmdlet을 실행 합니다.

```powershell
Get-KdsRootKey
```

명령에서 키 ID를 반환 하는 경우 모든 설정이 완료 되며 [그룹 관리 서비스 계정 만들기](#create-a-group-managed-service-account) 섹션으로 건너뛸 수 있습니다. 그렇지 않으면 계속 해 서 KDS 루트 키를 만듭니다.

여러 도메인 컨트롤러를 사용 하는 프로덕션 환경 또는 테스트 환경에서는 PowerShell에서 도메인 관리자 권한으로 다음 cmdlet을 실행 하 여 KDS 루트 키를 만듭니다.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

이 명령은 키가 즉시 적용 되는 것을 암시 하지만, KDS 루트 키가 복제 되 고 모든 도메인 컨트롤러에서 사용할 수 있도록 10 시간을 기다려야 합니다.

도메인에 도메인 컨트롤러가 하나만 있는 경우 10 시간 전에 키를 효과적으로 설정 하 여 프로세스를 신속 하 게 수행할 수 있습니다.

>[!IMPORTANT]
>프로덕션 환경에서는이 기법을 사용 하지 마세요.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>그룹 관리 서비스 계정 만들기

Windows 통합 인증을 사용 하는 모든 컨테이너에는 하나 이상의 gMSA 필요 합니다. 기본 gMSA 시스템 또는 네트워크 서비스로 실행 되는 앱이 네트워크에서 리소스에 액세스할 때마다 사용 됩니다. 컨테이너에 할당 된 호스트 이름에 관계 없이 gMSA의 이름은 네트워크에서 컨테이너 이름이 됩니다. 컨테이너 컴퓨터 계정의 다른 id로 컨테이너의 서비스 또는 응용 프로그램을 실행 하려는 경우에는 추가 gMSAs를 사용 하 여 컨테이너를 구성할 수도 있습니다.

GMSA를 만들 때 여러 컴퓨터에서 동시에 사용할 수 있는 공유 id도 만듭니다. GMSA 암호에 대 한 액세스는 Active Directory Access Control 목록에 의해 보호 됩니다. 각 gMSA 계정에 대 한 보안 그룹을 만들고 보안 그룹에 관련 컨테이너 호스트를 추가 하 여 암호에 대 한 액세스를 제한 하는 것이 좋습니다.

마지막으로, 컨테이너는 SPN (서비스 사용자 이름)을 자동으로 등록 하지 않으므로 gMSA 계정에 대해 하나 이상의 호스트 SPN을 수동으로 만들어야 합니다.

일반적으로 호스트 또는 http SPN은 gMSA 계정과 동일한 이름을 사용 하 여 등록 되지만 클라이언트에서 부하 분산 장치 뒤에 있는 컨테이너 화 된 응용 프로그램에 액세스 하거나 gMSA 이름과 다른 DNS 이름을 사용 하는 경우 다른 서비스 이름을 사용 해야 할 수 있습니다.

예를 들어 gMSA 계정 이름이 "WebApp01" 이지만 사용자가 `mysite.contoso.com`에서 사이트에 액세스 하는 경우 gMSA 계정에 대 `http/mysite.contoso.com` 한 SPN을 등록 해야 합니다.

일부 응용 프로그램에는 고유한 프로토콜에 대 한 추가 Spn이 필요할 수 있습니다. 예를 들어 SQL Server에 `MSSQLSvc/hostname` SPN이 필요 합니다.

다음 표에서는 gMSA를 만드는 데 필요한 특성을 보여 줍니다.

|gMSA 속성 | 필요한 값 | 예제 |
|--------------|----------------|--------|
|이름 | 모든 유효한 계정 이름입니다. | `WebApp01` |
|DnsHostName | 계정 이름에 추가 된 도메인 이름입니다. | `WebApp01.contoso.com` |
|ServicePrincipalNames | 호스트 SPN을 하나 이상 설정 하 고 필요에 따라 다른 프로토콜을 추가 합니다. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 컨테이너 호스트를 포함 하는 보안 그룹입니다. | `WebApp01Hosts` |

GMSA의 이름을 결정 했으면 PowerShell에서 다음 cmdlet을 실행 하 여 보안 그룹 및 gMSA를 만듭니다.

> [!TIP]
> **Domain Admins** 보안 그룹에 속해 있거나 **Create의 msds-primary-computer-msds-groupmanagedserviceaccount objects** 권한이 위임 된 계정을 사용 하 여 다음 명령을 실행 해야 합니다.
> [Uninstall-adserviceaccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) cmdlet은 [원격 서버 관리 도구](https://aka.ms/rsat)의 AD PowerShell 도구에 포함 되어 있습니다.

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01$", "ContainerHost02$", "ContainerHost03$"
```

개발, 테스트 및 프로덕션 환경에 대 한 별도의 gMSA 계정을 만드는 것이 좋습니다.

## <a name="prepare-your-container-host"></a>컨테이너 호스트 준비

GMSA를 사용 하 여 Windows 컨테이너를 실행 하는 각 컨테이너 호스트는 도메인에 가입 되어 있어야 하며 gMSA 암호를 검색할 수 있는 액세스 권한이 있어야 합니다.

1. Active Directory 도메인에 컴퓨터를 가입 시킵니다.
2. 호스트가 gMSA 암호에 대 한 액세스를 제어 하는 보안 그룹에 속하는지 확인 합니다.
3. 컴퓨터를 다시 시작 하 여 새 그룹 멤버 자격을 가져옵니다.
4. [Docker Desktop For Windows 10](https://docs.docker.com/docker-for-windows/install/) 또는 [Windows용 Docker Server](https://docs.docker.com/install/windows/docker-ee/)를 설정 합니다.
5. 바람직하지 [Uninstall-adserviceaccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)를 실행 하 여 호스트에서 gMSA 계정을 사용할 수 있는지 확인 합니다. 명령에서 **False**를 반환 하는 경우 [문제 해결 지침](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)을 따릅니다.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>자격 증명 사양 만들기

자격 증명 사양 파일은 컨테이너에서 사용 하려는 gMSA 계정에 대 한 메타 데이터를 포함 하는 JSON 문서입니다. Id 구성을 컨테이너 이미지와 별도로 유지 하 여 컨테이너에서 사용 하는 gMSA를 변경할 수 있습니다 .이는 자격 증명 사양 파일을 교체 하 여 코드를 변경 하지 않아도 됩니다.

자격 증명 사양 파일은 도메인에 가입 된 컨테이너 호스트에서 [Credentialspec PowerShell 모듈](https://aka.ms/credspec) 을 사용 하 여 만듭니다.
파일을 만든 후에는 다른 컨테이너 호스트나 컨테이너 orchestrator에 복사할 수 있습니다.
컨테이너 호스트에서 컨테이너를 대신 하 여 gMSA를 검색 하므로 자격 증명 사양 파일에는 gMSA 암호와 같은 암호가 포함 되지 않습니다.

Docker는 Docker 데이터 디렉터리의 **credentialspecs** 디렉터리에서 자격 증명 사양 파일을 찾을 것으로 예상 합니다. 기본 설치에서는 `C:\ProgramData\Docker\CredentialSpecs`에서이 폴더를 찾을 수 있습니다.

컨테이너 호스트에 자격 증명 사양 파일을 만들려면 다음을 수행 합니다.

1. RSAT AD PowerShell 도구 설치
    - Windows Server의 경우에는 **설치-ADD-WINDOWSFEATURE RSAT-PowerShell**을 실행 합니다.
    - Windows 10 버전 1809 이상에서는 **추가 WindowsCapability ' ActiveDirectory ~ ~ ~ '** 를 실행 합니다 .이 경우
    - 이전 버전의 Windows 10의 경우 <https://aka.ms/rsat>를 참조 하세요.
2. 다음 cmdlet을 실행 하 여 최신 버전의 [Credentialspec PowerShell 모듈](https://aka.ms/credspec)을 설치 합니다.

    ```powershell
    Install-Module CredentialSpec
    ```

    컨테이너 호스트에서 인터넷에 액세스할 수 없는 경우 인터넷에 연결 된 컴퓨터에서 `Save-Module CredentialSpec`를 실행 하 고 모듈 폴더를 `C:\Program Files\WindowsPowerShell\Modules` 또는 컨테이너 호스트에 `$env:PSModulePath`의 다른 위치로 복사 합니다.

3. 새 자격 증명 사양 파일을 만들려면 다음 cmdlet을 실행 합니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    기본적으로이 cmdlet은 제공 된 gMSA 이름을 컨테이너의 컴퓨터 계정으로 사용 하 여 자격 증명 사양을 만듭니다. 파일은 gMSA 도메인 및 파일 이름에 대 한 계정 이름을 사용 하 여 Docker CredentialSpecs 디렉터리에 저장 됩니다.

    다른 디렉터리에 파일을 저장 하려면 `-Path` 매개 변수를 사용 합니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -Path "C:\MyFolder\WebApp01_CredSpec.json"
    ```

    컨테이너에서 서비스 또는 프로세스를 보조 gMSA으로 실행 하는 경우 추가 gMSA 계정을 포함 하는 자격 증명 사양을 만들 수도 있습니다. 이렇게 하려면 `-AdditionalAccounts` 매개 변수를 사용 합니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    지원 되는 매개 변수의 전체 목록을 보려면 `Get-Help New-CredentialSpec -Full`를 실행 합니다.

4. 다음 cmdlet을 사용 하 여 모든 자격 증명 사양 목록과 전체 경로를 표시할 수 있습니다.

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>다음 단계

이제 gMSA 계정을 설정 했으므로 다음을 수행 하는 데 사용할 수 있습니다.

- [앱 구성](gmsa-configure-app.md)
- [컨테이너 실행](gmsa-run-container.md)
- [컨테이너 오케스트레이션](gmsa-orchestrate-containers.md)

설치 하는 동안 문제가 발생 하는 경우 [문제 해결 가이드](gmsa-troubleshooting.md) 에서 가능한 해결 방법을 확인 하세요.

## <a name="additional-resources"></a>추가 리소스

- GMSAs에 대 한 자세한 내용은 [그룹 관리 서비스 계정 개요](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)를 참조 하세요.
- 비디오 데모를 보려면 Ignite 2016에서 [기록 된 데모](https://youtu.be/cZHPz80I-3s?t=2672) 를 시청 하세요.
