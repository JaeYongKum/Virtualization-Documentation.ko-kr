# Windows 10의 Hyper-V에 대한 새로운 기능

이 항목에서는 Windows 10®에서 Hyper-V의 새로운 기능 및 변경된 기능에 대해 설명합니다.

## Windows PowerShell Direct

이제 호스트 운영 체제에서 가상 컴퓨터 내 Windows PowerShell 명령을 실행하는 쉽고 신뢰할 수 있는 방법이 있습니다. 네트워크 또는 방화벽 요구 사항이나 특별한 구성이 없습니다. 
원격 관리 구성에 관계 없이 작동합니다. 이를 사용하려면 호스트와 가상 컴퓨터 게스트 운영 체제에서 Windows 10 또는 Windows Server 기술 미리 보기를 실행해야 합니다.

PowerShell Direct 세션을 만들려면 다음 명령 중 하나를 사용합니다.

``` PowerShell
Enter-PSSession -VMName VMName
Invoke-Command -VMName VMName -ScriptBlock { commands }
```

현재 Hyper-V 관리자는 해당 Hyper-V 호스트에서 가상 컴퓨터에 연결하기 위해 두 가지 범주의 도구를 사용합니다.
- PowerShell 또는 원격 데스크톱 등의 원격 관리 도구
- Hyper-V 가상 컴퓨터 연결(VM 연결)

이 두 기술은 잘 작동하지만 각각은 Hyper-V 배포가 증가하므로 장단점이 있습니다. VMConnect는 신뢰할 수 있지만 자동화하기가 어렵습니다. 원격 PowerShell은 강력한 반면 설정 및 유지 관리가 어려울 수 있습니다.

Windows PowerShell Direct는 VMConnect를 간소화하여 강력한 스크립팅 및 자동화 환경을 제공합니다. Windows PowerShell Direct는 호스트와 가상 컴퓨터 간에서 실행하기 때문에 네트워크 연결 또는 원격 관리를 활성화할 필요가 없습니다. 가상 컴퓨터에 로그인하려면 게스트 자격 증명이 필요합니다.

### 요구 사항

- 게스트로 Windows 10 또는 Windows Server 기술 미리 보기를 실행하는 가상 컴퓨터를 사용하여 Windows 10 또는 Windows Server 기술 미리 보기 호스트에 연결해야 합니다.
- 호스트에서 Hyper-V 관리자 자격 증명을 사용하여 로그인해야 합니다.
- 가상 컴퓨터에 대한 사용자 자격 증명이 필요합니다.
- 연결하려는 가상 컴퓨터는 실행 중이고 부팅되어야 합니다.


## 네트워크 어댑터 및 메모리에 대한 핫 추가 및 제거

이제 가동 중지 시간 없이 가상 컴퓨터가 실행되는 동안 네트워크 어댑터를 추가하거나 제거할 수 있습니다. 이는 Windows 및 Linux 운영 체제를 실행하는 2세대 가상 컴퓨터에 적합합니다.

또한 동적 메모리를 활성화하지 않은 경우라도 실행되는 동안 가상 컴퓨터에 할당된 메모리의 양을 조정할 수 있습니다. 이는 1세대와 2세대 가상 컴퓨터에 적합합니다.

## 프로덕션 검사점

프로덕션 검사점을 사용하여 모든 프로덕션 작업에 대해 완전히 지원되는 방식으로 나중에 복원할 수 있는 가상 컴퓨터의 "특정 시점" 이미지를 쉽게 만들 수 있습니다. 이는 저장된 상태 기술을 사용하는 대신 검사점을 만드는 게스트 내 백업 기술을 사용하여 수행됩니다. 프로덕션 검사점의 경우 Windows 가상 컴퓨터 내에서 VSS(볼륨 스냅숏 서비스)가 사용됩니다. Linux 가상 컴퓨터는 해당 파일 시스템 버퍼를 플러시하여 파일 시스템 일관성 검사점을 만듭니다. 저장된 상태 기술을 사용하여 검사점을 만들려면 가상 컴퓨터에 대한 표준 검사점을 사용하도록 선택할 수 있습니다.


> **중요:** 새 가상 컴퓨터에 대한 기본은 표준 검사점에 대체를 사용하여 프로덕션 검사점을 만드는 것입니다.


## Hyper-V 관리자의 향상된 기능

- **대체 자격 증명 지원** – 이제 다른 Windows 10 기술 미리 보기 원격 호스트에 연결할 때 Hyper-V 관리자에서 다른 자격 증명 집합을 사용할 수 있습니다. 나중에 로그온하기 쉽도록 이러한 자격 증명을 저장할 수도 있습니다.

- **하위 수준 관리** - 이제 Hyper-V 관리자를 사용하여 더 많은 버전의 Hyper-V를 관리할 수 있습니다. Windows 10 기술 미리 보기에서 Hyper-V 관리자를 사용하여 Windows Server 2012, Windows 8, Windows Server 2012 R2 및 Windows 8.1에서 Hyper-V를 실행하는 컴퓨터를 관리할 수 있습니다.

- **업데이트된 관리 프로토콜** - Hyper-V 관리자가 CredSSP, Kerberos 또는 NTLM 인증을 허용하는 WS-MAN 프로토콜을 사용하여 원격 Hyper-V 호스트와 통신하도록 업데이트되었습니다. CredSSP를 사용하여 원격 Hyper-V 호스트에 연결할 때 Active Directory에서 먼저 제한된 위임을 활성화하지 않고 실시간 마이그레이션을 수행할 수 있습니다. WS-MAN 기반 인프라는 원격 관리에 대한 호스트를 활성화하는 데 필요한 구성을 단순화합니다. WS-MAN은 기본적으로 열리는 포트 80을 통해 연결합니다.


## 연결된 대기 작업

AOAC(Always On/Always Connected) 전원 모델을 사용하는 컴퓨터에서 Hyper-V를 활성화하면 이제 연결된 대기 전원 상태를 사용할 수 있습니다.

Windows 8 및 8.1에서 Hyper-V는 AOAC(Always On/Always Connected) 전원 모델(InstantON이라고도 함)을 사용하는 컴퓨터가 절전 모드가 되지 않도록 합니다. 전체 설명은 이 [기술 자료 문서](
https://support.microsoft.com/en-us/kb/2973536)를 참조하십시오.


## Linux 보안 부팅

2세대 가상 컴퓨터에서 실행되는 더 많은 Linux 운영 체제는 이제 활성화된 보안 부팅 옵션으로 부팅할 수 있습니다. Ubuntu 14.04 이상 및 SUSE Linux Enterprise Server 12는 기술 미리 보기를 실행하는 호스트에서 보안 부팅에 대해 활성화됩니다. 처음으로 가상 컴퓨터를 부팅하기 전에 가상 컴퓨터가 Microsoft UEFI 인증 기관을 사용하도록 지정해야 합니다. 관리자 권한 Windows PowerShell 프롬프트에서 다음을 입력합니다.

    Set-VMFirmware vmname -SecureBootTemplate MicrosoftUEFICertificateAuthority

Hyper-V에서 실행 중인 Linux 가상 컴퓨터에 자세한 내용은 [Hyper-V의 Linux 및 FreeBSD 가상 컴퓨터](http://technet.microsoft.com/library/dn531030.aspx)를 참조하세요.


## 가상 컴퓨터 구성 버전

Windows 10에서 Hyper-V를 실행하는 호스트에서 Windows 8.1을 실행하는 호스트로 가상 컴퓨터를 이동하거나 가져오는 경우 가상 컴퓨터의 구성 파일이 자동으로 업그레이드되지 않습니다. 이렇게 하면 가상 컴퓨터를 Windows 8.1을 실행하는 호스트로 다시 이동할 수 있습니다. 가상 컴퓨터 구성 버전을 수동으로 업데이트할 때까지 새 가상 컴퓨터 기능에 대한 액세스를 가질 수 없습니다.

가상 컴퓨터 구성 버전은 가상 컴퓨터의 구성, 저장된 상태 및 스냅숏 파일과 호환되는 Hyper-V의 버전을 나타냅니다. 구성 버전 5의 가상 컴퓨터는 Windows 8.1과 호환되며 Windows 8.1 및 Windows 10에서 실행할 수 있습니다. 구성 버전 6의 가상 컴퓨터는 Windows 10과 호환되며 Windows 8.1에서 실행할 수 없습니다.

### 구성 버전 확인

관리자 권한 명령 프롬프트에서 다음 명령을 실행합니다.

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### 구성 버전 업그레이드

관리자 권한 Windows PowerShell 명령 프롬프트에서 다음 명령 중 하나를 실행합니다.

``` PowerShell
Update-VmConfigurationVersion <vmname>
```

또는

``` PowerShell
Update-VmConfigurationVersion <vmobject>
```

**중요: **
- 가상 컴퓨터 구성 버전을 업그레이드한 후 Windows 8.1을 실행하는 호스트로 가상 컴퓨터를 이동할 수 없습니다.
- 버전 6에서 버전 5로 가상 컴퓨터 구성 버전을 다운그레이드할 수 없습니다.
- 가상 컴퓨터 구성을 업그레이드하려면 가상 컴퓨터를 꺼야 합니다.
- 업그레이드 후 가상 컴퓨터는 새 구성 파일 형식을 사용합니다. 자세한 내용은 새 가상 컴퓨터 구성 파일 형식을 참조하세요.


## 구성 파일 형식

가상 컴퓨터는 이제 가상 컴퓨터 구성 데이터 읽기 및 쓰기의 효율성을 높이기 위해 설계된 새 구성 파일 형식을 가집니다. 도한 저장소 오류가 발생하는 경우 데이터 손상 가능성을 줄이기 위해 디자인되었습니다. 새 구성 파일은 가상 컴퓨터 구성 데이터에 대해 .VMCX 확장을 런타임 상태 데이터에 대해 .VMRS 확장을 사용합니다.


> **중요:** .VMCX 파일은 이진 형식입니다. .VMCX 또는 .VMRS 파일을 직접 편집하는 기능은 지원되지 않습니다.

## Windows Update를 통한 통합 서비스

Windows 게스트에 대한 통합 서비스로 업데이트는 이제 Windows Update를 통해 배포됩니다.

통합 구성 요소(통합 서비스라고도 함)는 가상 컴퓨터가 호스트 운영 체제와 통신할 수 있도록 하는 가상 드라이버 집합입니다. 서비스를 시간 동기화에서 게스트 파일 복사까지의 범위로 제어합니다. 지난 몇 년 동안 고객이 통합 구성 요소 설치 및 업데이트에 대해 업그레이드 프로세스 중 문제를 발견했는지 논의해 왔습니다.


지금까지 모든 버전의 Hyper-V는 새로운 통합 구성 요소와 함께 제공되었습니다. Hyper-V 호스트 업그레이드는 가상 컴퓨터의 통합 구성 요소에 대한 업그레이드 또한 필요합니다. 새 통합 구성 요소는 Hyper-V 호스트에 포함된 다음 vmguest.iso를 사용하여 가상 컴퓨터에 설치되었습니다. 이 프로세스는 가상 컴퓨터를 다시 시작해야 했고 다른 Windows 업데이트와 일괄 처리할 수 없었습니다. Hyper-V 관리자는 vmguest.iso를 제공하고 가상 컴퓨터 관리자는 이를 설치해야 했으므로 통합 구성 요소 업그레이드는 가상 컴퓨터에서 Hyper-V 관리자의 관리자 자격 증명을 필요로 했습니다. 항상 그런 것은 아닙니다.
　　


Windows 10 이상에서 모든 통합 구성 요소는 Windows Update를 통해 다른 중요한 업데이트와 함께 가상 컴퓨터에 제공됩니다.


현재 다음을 실행 중인 가상 컴퓨터에 대해 사용 가능한 업데이트가 있습니다.
*  Windows Server 2012
*  Windows Server 2008 R2
*  Windows 8
*  Windows 7

가상 컴퓨터는 Windows Update 또는 WSUS 서버에 연결되어야 합니다. 나중에 통합 구성 요소 업데이트는 KB로 나열되는 이 릴리스에 대한 범주 ID를 가집니다.

적하용 가능성을 결정하는 방법에 대한 내용은 이 [블로그 게시물](http://blogs.technet.com/b/virtualization/archive/2014/11/24/integration-components-how-we-determine-windows-update-applicability.aspx)을 참조세요.


통합 서비스 설치에 대한 자세한 연습은 [이 블로그](http://blogs.msdn.com/b/virtual_pc_guy/archive/2014/11/12/updating-integration-components-over-windows-update.aspx) 게시물을 참조하세요.


> **중요:** ISO 이미지 파일 vmguest.iso는 통합 구성 요소를 업데이트하기 위해 더 이상 필요하지 않습니다. Windows 10의 Hyper-V에 포함되지 않았습니다.


## 다음 단계

[Windows 10에서 Hyper-V 연습](..\quick_start\walkthrough.md)



