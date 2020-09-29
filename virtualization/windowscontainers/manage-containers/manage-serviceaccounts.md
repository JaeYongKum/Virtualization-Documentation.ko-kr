---
title: Windows 컨테이너에 대한 gMSA 만들기
description: Windows 컨테이너에 대한 gMSA(그룹 관리 서비스 계정)를 만드는 방법입니다.
keywords: Docker, 컨테이너, Active Directory, gMSA, 그룹 관리 서비스 계정, 그룹 관리 서비스 계정
author: rpsqrd
ms.author: jgerend
ms.date: 01/03/2019
ms.topic: how-to
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b8a224a34d5cb666c55190e30b7002e2483cf89c
ms.sourcegitcommit: 160405a16d127892b6e2897efa95680f29f0496a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/22/2020
ms.locfileid: "90990916"
---
# <a name="create-gmsas-for-windows-containers"></a>Windows 컨테이너에 대한 gMSA 만들기

Windows 기반 네트워크는 일반적으로 AD(Active Directory)를 사용하여 사용자, 컴퓨터 및 기타 네트워크 리소스 간의 인증과 권한 부여를 용이하게 합니다. 엔터프라이즈 애플리케이션 개발자는 Windows 통합 인증을 활용하기 위해 애플리케이션이 AD와 통합되어 도메인 조인 서버에서 실행되도록 설계하는 경우가 많습니다. 이를 통해 사용자 및 기타 서비스에서 ID를 사용하여 애플리케이션에 투명하게 자동으로 로그인할 수 있습니다.

Windows 컨테이너는 도메인에 조인할 수 없지만 Active Directory 도메인 ID를 계속 사용하여 다양한 인증 시나리오를 지원할 수 있습니다.

이를 위해 Windows 컨테이너가 [gMSA(그룹 관리 서비스 계정)](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)로 실행되도록 구성할 수 있습니다. 이는 여러 컴퓨터에서 암호를 인식할 필요 없이 ID를 공유할 수 있도록 설계된 Windows Server 2012에 도입된 특수한 유형의 서비스 계정입니다.

gMSA를 사용하여 컨테이너를 실행하는 경우 컨테이너 호스트에서 Active Directory 도메인 컨트롤러로부터 gMSA 암호를 검색하여 컨테이너 인스턴스에 제공합니다. 컨테이너는 컴퓨터 계정(SYSTEM)에서 네트워크 리소스에 액세스해야 할 때마다 gMSA 자격 증명을 사용합니다.

이 문서에서는 Windows 컨테이너에서 Active Directory 그룹 관리 서비스 계정 사용을 시작하는 방법에 대해 설명합니다.

## <a name="prerequisites"></a>필수 구성 요소

그룹 관리 서비스 계정으로 Windows 컨테이너를 실행하는 데 필요한 항목은 다음과 같습니다.

- Windows Server 2012 이상을 실행하는 하나 이상의 도메인 컨트롤러가 있는 Active Directory 도메인. gMSA를 사용하기 위한 포리스트 또는 도메인 기능 수준 요구 사항은 없지만, gMSA 암호는 Windows Server 2012 이상을 실행하는 도메인 컨트롤러에서만 배포할 수 있습니다. 자세한 내용은 [gMSA용 Active Directory 요구 사항](/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)을 참조하세요.
- gMSA 계정을 만들 수 있는 권한. gMSA 계정을 만들려면 도메인 관리자이거나 *msDS-GroupManagedServiceAccount 개체 만들기* 권한이 위임된 계정을 사용해야 합니다.
- 인터넷에 액세스하여 CredentialSpec PowerShell 모듈을 다운로드할 수 있는 권한. 연결이 끊긴 환경에서 작업하는 경우 인터넷 액세스 권한이 있는 컴퓨터에서 [모듈을 저장](/powershell/module/powershellget/save-module?view=powershell-5.1)하고 개발 머신 또는 컨테이너 호스트에 복사할 수 있습니다.

## <a name="one-time-preparation-of-active-directory"></a>일회성 Active Directory 준비

gMSA를 도메인에 아직 만들지 않은 경우 KDS(키 배포 서비스) 루트 키를 생성해야 합니다. KDS는 gMSA 암호를 권한 있는 호스트에 만들고, 회전하고, 릴리스하는 역할을 담당합니다. 컨테이너 호스트에서 컨테이너를 실행하는 데 gMSA를 사용해야 하는 경우 KDS에 연결하여 현재 암호를 검색합니다.

KDS 루트 키가 이미 만들어졌는지 확인하려면 AD PowerShell 도구가 설치된 도메인 컨트롤러 또는 도메인 멤버에서 다음 PowerShell cmdlet을 도메인 관리자 권한으로 실행합니다.

```powershell
Get-KdsRootKey
```

명령에서 키 ID를 반환하면 모든 설정이 완료되고 [그룹 관리 서비스 계정 만들기](#create-a-group-managed-service-account) 섹션으로 건너뛸 수 있습니다. 그렇지 않으면 계속 진행하여 KDS 루트 키를 만듭니다.

여러 도메인 컨트롤러가 있는 프로덕션 환경 또는 테스트 환경의 PowerShell에서 다음 cmdlet을 도메인 관리자 권한으로 실행하여 KDS 루트 키를 만듭니다.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

이 명령은 키가 즉시 적용됨을 의미하지만 KDS 루트 키를 복제하여 모든 도메인 컨트롤러에서 사용할 수 있을 때까지 10시간 정도 기다려야 합니다.

하나의 도메인 컨트롤러만 도메인에 있는 경우 10시간 전에 키를 유효하게 설정하여 프로세스를 빠르게 수행할 수 있습니다.

>[!IMPORTANT]
>프로덕션 환경에서는 이 기술을 사용하지 마세요.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>그룹 관리 서비스 계정 만들기

Windows 통합 인증을 사용하는 모든 컨테이너에는 하나 이상의 gMSA가 필요합니다. 기본 gMSA는 시스템 또는 네트워크 서비스로 실행되는 앱에서 네트워크의 리소스에 액세스할 때마다 사용됩니다. gMSA의 이름은 컨테이너에 할당된 호스트 이름과 관계없이 네트워크에서 컨테이너의 이름이 됩니다. 컨테이너에서 서비스 또는 애플리케이션을 컨테이너 컴퓨터 계정과 다른 ID로 실행하려는 경우 추가 gMSA를 사용하여 컨테이너를 구성할 수도 있습니다.

gMSA를 만드는 경우 다른 여러 머신에서 동시에 사용할 수 있는 공유 ID도 만듭니다. gMSA 암호에 대한 액세스는 Active Directory 액세스 제어 목록으로 보호됩니다. 각 gMSA 계정에 대한 보안 그룹을 만들고 관련 컨테이너 호스트를 보안 그룹에 추가하여 암호에 대한 액세스를 제한하는 것이 좋습니다.

마지막으로, 컨테이너에서는 SPN(서비스 사용자 이름)을 자동으로 등록하지 않으므로 gMSA 계정에 대해 하나 이상의 호스트 SPN을 수동으로 만들어야 합니다.

일반적으로 호스트 또는 http SPN은 gMSA 계정과 동일한 이름을 사용하여 등록되지만, 다른 서비스 이름(클라이언트가 부하 분산 장치 내부에서 컨테이너화된 애플리케이션에 액세스하는 경우) 또는 gMSA 이름과 다른 DNS 이름을 사용해야 할 수 있습니다.

예를 들어 gMSA 계정 이름이 "WebApp01"이지만 사용자가 `mysite.contoso.com`의 사이트에 액세스하는 경우 gMSA 계정에 대한 `http/mysite.contoso.com` SPN을 등록해야 합니다.

일부 애플리케이션에는 고유한 프로토콜에 대한 추가 SPN이 필요할 수 있습니다. 예를 들어 SQL Server에는 `MSSQLSvc/hostname` SPN이 필요합니다.

다음 표에는 gMSA를 만드는 데 필요한 특성이 나와 있습니다.

|gMSA 속성 | 필수 값 | 예제 |
|--------------|----------------|--------|
|이름 | 유효한 계정 이름입니다. | `WebApp01` |
|DnsHostName | 계정 이름에 추가된 도메인 이름입니다. | `WebApp01.contoso.com` |
|ServicePrincipalNames | 호스트 SPN 이상을 설정하고, 필요에 따라 다른 프로토콜을 추가합니다. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 컨테이너 호스트를 포함하는 보안 그룹입니다. | `WebApp01Hosts` |

gMSA 이름이 결정되었으면 PowerShell에서 다음 cmdlet을 실행하여 보안 그룹 및 gMSA를 만듭니다.

> [!TIP]
> 다음 명령을 실행하려면 **도메인 관리자** 보안 그룹에 속하거나 **msDS-GroupManagedServiceAccount 개체 만들기** 권한이 위임된 계정을 사용해야 합니다.
> [New-ADServiceAccount](/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) cmdlet은 [원격 서버 관리 도구](/troubleshoot/windows-server/system-management-components/remote-server-administration-tools)에 있는 AD PowerShell 도구의 일부입니다.

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

개발, 테스트 및 프로덕션 환경에 대해 별도의 gMSA 계정을 만드는 것이 좋습니다.

## <a name="prepare-your-container-host"></a>컨테이너 호스트 준비

gMSA를 사용하여 Windows 컨테이너를 실행할 각 컨테이너 호스트는 도메인에 조인되어 있어야 하며 gMSA 암호를 검색할 수 있는 액세스 권한이 있어야 합니다.

1. 컴퓨터를 Active Directory 도메인에 조인합니다.
2. 호스트가 gMSA 암호에 대한 액세스를 제어하는 보안 그룹에 속하는지 확인합니다.
3. 컴퓨터를 다시 시작하여 새 그룹 멤버 자격을 가져옵니다.
4. [Windows 10용 Docker 데스크톱](https://docs.docker.com/docker-for-windows/install/) 또는 [Windows Server용 Docker](https://docs.docker.com/install/windows/docker-ee/)를 설치합니다.
5. (추천) [Test-ADServiceAccount](/powershell/module/activedirectory/test-adserviceaccount)를 실행하여 호스트에서 gMSA 계정을 사용할 수 있는지 확인합니다. 명령에서 **False**를 반환하면 [문제 해결 지침](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)을 따르세요.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>자격 증명 사양 만들기

자격 증명 사양 파일은 컨테이너에서 사용하려는 gMSA 계정에 대한 메타데이터가 포함된 JSON 문서입니다. ID 구성을 컨테이너 이미지와 별도로 유지하면 코드를 변경할 필요 없이 자격 증명 사양 파일을 교체하기만 하여 컨테이너에서 사용하는 gMSA를 변경할 수 있습니다.

자격 증명 사양 파일은 도메인 조인 컨테이너 호스트에서 [CredentialSpec PowerShell 모듈](https://aka.ms/credspec)을 사용하여 만들어집니다.
파일이 만들어지면 다른 컨테이너 호스트 또는 컨테이너 오케스트레이터에 복사할 수 있습니다.
컨테이너 호스트는 컨테이너를 대신하여 gMSA를 검색하므로 자격 증명 사양 파일에는 gMSA 암호와 같은 비밀이 포함되어 있지 않습니다.

Docker는 Docker 데이터 디렉터리의 **CredentialSpecs** 디렉터리 아래에서 자격 증명 사양 파일을 찾으려고 합니다. 기본 설치의 경우 이 폴더는 `C:\ProgramData\Docker\CredentialSpecs`에서 찾을 수 있습니다.

컨테이너 호스트에서 자격 증명 사양 파일을 만들려면 다음을 수행합니다.

1. RSAT AD PowerShell 도구를 설치합니다.
    - Windows Server의 경우 **Install-WindowsFeature RSAT-AD-PowerShell**을 실행합니다.
    - Windows 10 버전 1809 이상의 경우 **Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'** 을 실행합니다.
    - 이전 버전의 Windows 10인 경우 <https://aka.ms/rsat>를 참조하세요.
2. 다음 cmdlet을 실행하여 최신 버전의 [CredentialSpec PowerShell 모듈](https://aka.ms/credspec)을 설치합니다.

    ```powershell
    Install-Module CredentialSpec
    ```

    컨테이너 호스트에서 인터넷에 액세스할 수 없으면 인터넷에 연결된 머신에서 `Save-Module CredentialSpec`을 실행하고 모듈 폴더를 컨테이너 호스트의 `C:\Program Files\WindowsPowerShell\Modules` 또는 `$env:PSModulePath`의 다른 위치에 복사합니다.

3. 다음 cmdlet을 실행하여 새 자격 증명 사양 파일을 만듭니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    이 cmdlet은 기본적으로 제공된 gMSA 이름을 컨테이너의 컴퓨터 계정으로 사용하여 자격 증명 사양을 만듭니다. 이 파일은 gMSA 도메인 및 파일 이름에 대한 계정 이름을 사용하여 Docker CredentialSpecs 디렉터리에 저장됩니다.

    파일을 다른 디렉터리에 저장하려면 `-Path` 매개 변수를 사용합니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -Path "C:\MyFolder\WebApp01_CredSpec.json"
    ```

    컨테이너에서 보조 gMSA로 서비스 또는 프로세스를 실행하는 경우 추가 gMSA 계정이 포함된 자격 증명 사양을 만들 수도 있습니다. 이렇게 하려면 `-AdditionalAccounts` 매개 변수를 사용합니다.

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    지원되는 매개 변수의 전체 목록을 보려면 `Get-Help New-CredentialSpec -Full`을 실행합니다.

4. 다음 cmdlet을 사용하여 모든 자격 증명 사양 및 해당 전체 경로에 대한 목록을 표시할 수 있습니다.

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>다음 단계

이제 gMSA 계정이 설정되었으므로 이를 사용하여 다음 작업을 수행할 수 있습니다.

- [앱 구성](gmsa-configure-app.md)
- [컨테이너 실행](gmsa-run-container.md)
- [컨테이너 오케스트레이션](gmsa-orchestrate-containers.md)

설정하는 동안 문제가 발생하면 [문제 해결 지침](gmsa-troubleshooting.md)에서 가능한 해결 방법을 확인하세요.

## <a name="additional-resources"></a>추가 리소스

- gMSA에 대한 자세한 내용은 [그룹 관리 서비스 계정 개요](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)를 참조하세요.
- 비디오 데모를 보려면 Ignite 2016의 [녹화 데모](https://youtu.be/cZHPz80I-3s?t=2672)를 시청하세요.