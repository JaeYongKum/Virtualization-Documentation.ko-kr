# 새 Hyper-V 가상 컴퓨터에 Windows 컨테이너 호스트 배포

이 문서에서는 PowerShell 스크립트를 사용하여 새 Hyper-V 가상 컴퓨터를 배포한 다음 Windows 컨테이너 호스트로 사용하는 절차를 안내합니다.

기존 가상 또는 물리적 시스템에 대한 Windows 컨테이너 호스트의 스크립트화된 배포 단계는 [현재 Windows 컨테이너 호스트 배포](./inplace_setup.md)를 참조하세요.

**컨테이너 OS 이미지를 설치하기 전에 반드시 읽으십시오.** Microsoft Windows Server 시험판 소프트웨어("사용 조건")의 사용 조건은 Microsoft Windows 컨테이너 OS 이미지 추가(“추가 소프트웨어) 사용에 적용됩니다. 추가 구성 소프트웨어를 다운로드하여 사용하면 이 사용 조건에 동의하는 것이며 사용 조건에 동의하지 않으면 해당 항목을 사용해서는 안 됩니다.  Windows Server 시험판 소프트웨어와 보조 소프트웨어 모두 Microsoft Corporation에서 사용을 허가합니다.

이 빠른 시작에서 **Windows Server**와 **Hyper-V 컨테이너** 연습을 모두 완료하려면 다음이 필요합니다.

* Windows 10 빌드 1056 이상을 실행하는 시스템/Windows Server Technical Preview 4 이상
* Hyper-V 역할 활성화([지침 참조](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install#UsingPowerShell))
* 컨테이너 호스트 이미지, OS 기본 이미지 및 설치 스크립트를 위해 사용 가능한 20GB 저장 공간
* Hyper-V 호스트에 대한 관리자 권한

> Hyper-V 컨테이너를 실행하는 가상화된 호스트에는 중첩된 가상화가 필요합니다. 물리적 호스트와 가상 호스트 모두 중첩된 가상화를 지원하는 OS를 실행 중이어야 합니다. 자세한 내용은 [Windows Server 2016 기술 미리 보기](https://technet.microsoft.com/library/dn765471.aspx#BKMK_nested)의 새로운 Hyper-V 기능을 참조하세요.

## 새 가상 컴퓨터에 새 컨테이너 호스트 설치

Windows 컨테이너는 Windows 컨테이너 호스트와 컨테이너 OS 기본 이미지 등의 몇 가지 구성 요소로 구성됩니다. 편의를 위해 이러한 항목을 다운로드하여 구성하는 스크립트를 하나로 모아 놓았습니다. 다음 단계를 통해 새 Hyper-V 가상 컴퓨터를 배포하고 이 시스템을 Windows 컨테이너 호스트로 구성합니다.

관리자 권한으로 PowerShell 세션을 시작합니다. PowerShell 아이콘을 마우스 오른쪽 단추로 클릭하고 '관리자 권한으로 실행'을 선택하거나 아무 PowerShell 세션에서나 다음 명령을 실행하면 됩니다.

``` powershell
PS C:\> start-process powershell -Verb runAs
```

스크립트를 다운로드하여 실행하기 전에 외부 Hyper-V 가상 스위치를 만들었는지 확인합니다. 없으면 이 스크립트가 실패합니다.

다음을 실행하여 외부 가상 스위치의 목록을 반환합니다. 반환되는 것이 없으며 새 외부 가상 스위치를 만들고 이 가이드의 다음 단계를 진행합니다.

```powershell
PS C:\> Get-VMSwitch | where {$_.SwitchType –eq “External”}
```

다음 명령을 사용하여 구성 스크립트를 다운로드합니다. 이 스크립트는 [구성 스크립트](https://aka.ms/tp4/New-ContainerHost) 위치에서 수동으로 다운로드할 수도 있습니다.

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/New-ContainerHost -OutFile c:\New-ContainerHost.ps1
```

다음 명령을 실행하여 컨테이너 호스트를 만들고 구성합니다. 여기서 `<containerhost>`는 가상 컴퓨터 이름이 됩니다.

``` powershell
PS C:\> c:\New-ContainerHost.ps1 –VmName <containerhost> -WindowsImage ServerDatacenterCore -Hyperv
```

스크립트가 시작되면 암호를 입력하라는 메시지가 표시됩니다. 관리자 계정에 할당된 암호를 말합니다.

다음으로 사용 조건을 읽고 동의하라는 메시지가 표시됩니다.

```
Before installing and using the Windows Server Technical Preview 4 with Containers virtual machine you must:
    1. Review the license terms by navigating to this link: http://aka.ms/tp4/containerseula
    2. Print and retain a copy of the license terms for your records.
By downloading and using the Windows Server Technical Preview 4 with Containers virtual machine you agree to such
license terms. Please confirm you have accepted and agree to the license terms.
[N] No  [Y] Yes  [?] Help (default is "N"):
```

그러면 이 스크립트가 Windows 컨테이너 구성 요소를 다운로드하여 구성하기 시작합니다. 이 프로세스는 다운로드 크기가 크기 때문에 다소 시간이 걸릴 수 있습니다. 완료되면 이제 가상 컴퓨터에서 PowerShell과 Docker를 모두 사용하여 Windows 컨테이너와 Windows 컨테이너 이미지를 만들어 관리할 준비가 된 것입니다.

구성 스크립트가 완료되면 구성 프로세스 중에 지정한 암호를 사용하여 가상 컴퓨터에 로그인하고 가상 컴퓨터의 IP 주소가 올바른지 확인합니다. 이러한 항목이 완료되면 시스템에서 Windows 컨테이너에 대한 준비가 끝났습니다.

## 다음 단계: 컨테이너 사용 시작

이제 Windows 컨테이너 기능을 실행하는 Windows Server 2016 시스템이 준비되었으므로 다음 가이드로 이동하여 Windows Server와 Hyper-V 컨테이너 작업을 시작합니다.

[빠른 시작: Windows 컨테이너 및 PowerShell](./manage_powershell.md)  
[빠른 시작: Windows 컨테이너 및 Docker](./manage_docker.md)



