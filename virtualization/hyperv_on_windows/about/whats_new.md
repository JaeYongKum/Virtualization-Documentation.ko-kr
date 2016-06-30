---
redirect_url: ../windows_welcome
translationtype: Human Translation
ms.sourcegitcommit: 3491d21a31a92f0a97de572afafc29ae8e661c12
ms.openlocfilehash: 54b496f535b94f0b9aa83cce3ae5504830faee65

---

## 네트워크 어댑터 및 메모리에 대한 핫 추가 및 제거

이제 가상 컴퓨터가 실행되는 동안 가동 중지 시간 없이 네트워크 어댑터를 추가하거나 제거할 수 있습니다. 이는 Windows 및 Linux 운영 체제를 실행하는 2세대 가상 컴퓨터에 적합합니다. 

또한 동적 메모리를 활성화하지 않은 경우라도 실행되는 동안 가상 컴퓨터에 할당된 메모리의 양을 조정할 수 있습니다. 이는 1세대와 2세대 가상 컴퓨터에 적합합니다.

## 프로덕션 검사점

프로덕션 검사점을 사용하여 모든 프로덕션 작업에 대해 완전히 지원되는 방식으로 나중에 복원할 수 있는 가상 컴퓨터의 "특정 시점" 이미지를 쉽게 만들 수 있습니다. 이는 저장된 상태 기술을 사용하는 대신 검사점을 만드는 게스트 내 백업 기술을 사용하여 수행됩니다. 프로덕션 검사점의 경우 Windows 가상 컴퓨터 내에서 VSS(볼륨 스냅숏 서비스)가 사용됩니다. Linux 가상 컴퓨터는 해당 파일 시스템 버퍼를 플러시하여 파일 시스템 일관성 검사점을 만듭니다. 저장된 상태 기술을 사용하여 검사점을 만들려면 가상 컴퓨터에 대한 표준 검사점을 사용하도록 선택할 수 있습니다. 


> **중요:** 새 가상 컴퓨터에 대한 기본은 표준 검사점에 대체를 사용하여 프로덕션 검사점을 만드는 것입니다. 
 

## Hyper-V 관리자의 향상된 기능

- **대체 자격 증명 지원** – 이제 다른 Windows 10 Technical Preview 원격 호스트에 연결할 때 Hyper-V 관리자에서 다른 자격 증명 집합을 사용할 수 있습니다. 나중에 로그온하기 쉽도록 이러한 자격 증명을 저장할 수도 있습니다. 

- **하위 수준 관리** - 이제 Hyper-V 관리자를 사용하여 더 많은 버전의 Hyper-V를 관리할 수 있습니다. Windows 10 Technical Preview에서 Hyper-V 관리자를 사용하여 Windows Server 2012, Windows 8, Windows Server 2012 R2 및 Windows 8.1에서 Hyper-V를 실행하는 컴퓨터를 관리할 수 있습니다.

- **업데이트된 관리 프로토콜** - Hyper-V 관리자가 CredSSP, Kerberos 또는 NTLM 인증을 허용하는 WS-MAN 프로토콜을 사용하여 원격 Hyper-V 호스트와 통신하도록 업데이트되었습니다. CredSSP를 사용하여 원격 Hyper-V 호스트에 연결할 때 Active Directory에서 먼저 제한된 위임을 활성화하지 않고 실시간 마이그레이션을 수행할 수 있습니다. WS-MAN 기반 인프라는 원격 관리에 대한 호스트를 활성화하는 데 필요한 구성을 단순화합니다. WS-MAN은 기본적으로 열리는 포트 80을 통해 연결합니다.


## 연결된 대기 상태 호환성 

AOAC(Always On/Always Connected) 전원 모델을 사용하는 컴퓨터에서 Hyper-V를 활성화하면 이제 연결된 대기 전원 상태를 사용할 수 있습니다.

Windows 8 및 8.1에서 Hyper-V는 AOAC(Always On/Always Connected) 전원 모델(InstantON이라고도 함)을 사용하는 컴퓨터가 절전 모드가 되지 않도록 합니다. 전체 설명을 보려면 이 [기술 자료 문서](
https://support.microsoft.com/en-us/kb/2973536)를 참조하세요.


## Linux 보안 부팅 

2세대 가상 컴퓨터에서 실행되는 더 많은 Linux 운영 체제는 이제 활성화된 보안 부팅 옵션으로 부팅할 수 있습니다.  Ubuntu 14.04 이상 및 SUSE Linux Enterprise Server 12는 기술 미리 보기를 실행하는 호스트에서 보안 부팅에 대해 활성화됩니다. 처음으로 가상 컴퓨터를 부팅하기 전에 가상 컴퓨터가 Microsoft UEFI 인증 기관을 사용하도록 지정해야 합니다.  관리자 권한 Windows PowerShell 프롬프트에서 다음을 입력합니다.

    Set-VMFirmware [-VMName] <VMName> [-SecureBootTemplate] <MicrosoftUEFICertificateAuthority>

Hyper-V에서 실행 중인 Linux 가상 컴퓨터에 자세한 내용은 [Linux and FreeBSD Virtual Machines on Hyper-V(Hyper-V의 Linux 및 FreeBSD 가상 컴퓨터)](http://technet.microsoft.com/library/dn531030.aspx)를 참조하세요.
 
 
## 가상 컴퓨터 구성 버전

Windows 8.1을 실행하는 호스트에서 Windows 10에서 Hyper-V를 실행하는 호스트로 가상 컴퓨터를 이동하거나 가져오는 경우 가상 컴퓨터의 구성 파일이 자동으로 업그레이드되지 않습니다. 이렇게 하면 가상 컴퓨터를 Windows 8.1을 실행하는 호스트로 다시 이동할 수 있습니다. 가상 컴퓨터 구성 버전을 수동으로 업데이트한 다음에야 가상 컴퓨터에서 새로운 Hyper-v 기능을 사용할 수 있습니다. 

가상 컴퓨터 구성 버전은 가상 컴퓨터의 구성, 저장된 상태 및 스냅숏 파일이 호환되는 Hyper-V의 버전을 나타냅니다. 구성 버전 5의 가상 컴퓨터는 Windows 8.1과 호환되며 Windows 8.1 및 Windows 10에서 실행할 수 있습니다. 구성 버전 6의 가상 컴퓨터는 Windows 10과 호환되며 Windows 8.1에서 실행할 수 없습니다.

### 구성 버전 확인

관리자 권한 명령 프롬프트에서 다음 명령을 실행합니다.

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### 구성 버전 업그레이드 

관리자 권한 Windows PowerShell 프롬프트에서 다음 명령 중 하나를 실행합니다.

``` 
Update-VmConfigurationVersion <VMName>
```

또는

``` 
Update-VmConfigurationVersion <VMObject>
```

> **중요:**
>
- 가상 컴퓨터 구성 버전을 업그레이드한 후 Windows 8.1을 실행하는 호스트로 가상 컴퓨터를 이동할 수 없습니다.
- 버전 6에서 버전 5로 가상 컴퓨터 구성 버전을 다운그레이드할 수 없습니다.
- 가상 컴퓨터 구성을 업그레이드하려면 가상 컴퓨터를 꺼야 합니다.
- 업그레이드 후 가상 컴퓨터는 새 구성 파일 형식을 사용합니다. 자세한 내용은 [구성 파일 형식](#configuration-file-format)을 참조하세요.


## <a name="configuration-file-format"></a>구성 파일 형식

가상 컴퓨터는 이제 가상 컴퓨터 구성 데이터 읽기 및 쓰기의 효율성을 높이기 위해 설계된 새 구성 파일 형식을 가집니다. 도한 저장소 오류가 발생하는 경우 데이터 손상 가능성을 줄이기 위해 디자인되었습니다. 새 구성 파일은 가상 컴퓨터 구성 데이터에 대해 .VMCX 확장을 런타임 상태 데이터에 대해 .VMRS 확장을 사용합니다. 

> **중요:** .VMCX 파일은 이진 형식입니다. .VMCX 또는 .VMRS 파일을 직접 편집하는 기능은 지원되지 않습니다.


<!--HONumber=Jun16_HO4-->


