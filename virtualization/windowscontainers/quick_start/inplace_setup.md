# 기존 가상 또는 실제 시스템에 Windows 컨테이너 호스트 배포

이 문서에서는 PowerShell 스크립트를 사용하여 기존 물리적 또는 가상 호스트에서 Windows 컨테이너 역할을 배포 및 구성하는 단계를 안내합니다.

Windows 컨테이너 호스트로 구성된 새 Hyper-V 가상 컴퓨터의 스크립트 배포를 단계별로 실행하려면 [새 Hyper-V Windows 컨테이너 호스트](./container_setup.md)를 참조하세요.

**컨테이너 OS 이미지를 설치하기 전에 반드시 읽으시기 바랍니다.** Microsoft Windows Server 시험판 소프트웨어의 사용 조건("사용 조건")은 Microsoft Windows 컨테이너 OS 이미지 추가(“추가 소프트웨어”) 사용에 적용됩니다. 추가 구성 소프트웨어를 다운로드하여 사용하면 이 사용 조건에 동의하는 것이며 사용 조건에 동의하지 않으면 해당 항목을 사용해서는 안 됩니다.  Windows Server 시험판 소프트웨어와 보조 소프트웨어 모두 Microsoft Corporation에서 사용을 허가합니다.

이 빠른 시작에서 Windows Server 컨테이너와 Hyper-V 컨테이너 연습을 모두 완료하려면 다음이 필요합니다.

* Windows Server Technical Preview 4 이상을 실행하는 시스템
* 컨테이너 호스트 이미지, OS 기본 이미지 및 설치 스크립트를 위해 사용 가능한 10GB 저장 공간
* 시스템에 대한 관리자 권한

## 컨테이너에 대해 기존 가상 컴퓨터나 베어 메탈 호스트 설정

Windows 컨테이너에는 컨테이너 OS 기본 이미지가 필요합니다. 편의를 위해 이 이미지를 다운로드하여 설치하는 스크립트를 하나로 모아 놓았습니다. 다음 단계에 따라 시스템을 Windows 컨테이너 호스트 시스템으로 구성합니다. 자세한 내용은 [Windows Server 2016 Technical Preview](https://tnstage.redmond.corp.microsoft.com/en-US/library/dn765471.aspx#BKMK_nested)의 새로운 Hyper-V 기능을 참조하세요.

관리자 권한으로 PowerShell 세션을 시작합니다. 명령줄에서 다음 명령을 실행하여 시작할 수 있습니다.

``` powershell
PS C:\> powershell.exe
```

Windows의 제목이 "관리자: Windows PowerShell"인지 확인합니다. 이 항목이 관리자가 아니면 이 명령을 실행하여 관리자 권한으로 실행합니다.

``` powershell
PS C:\> start-process powershell -Verb runas
```

다음 명령을 사용하여 설치 스크립트를 다운로드합니다. 이 스크립트는 [구성 스크립트](https://aka.ms/tp4/Install-ContainerHost) 위치에서 수동으로 다운로드할 수도 있습니다.

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/Install-ContainerHost -OutFile C:\Install-ContainerHost.ps1
```

 다운로드가 완료된 후 스크립트를 실행합니다.
``` PowerShell
PS C:\> powershell.exe -NoProfile C:\Install-ContainerHost.ps1 -HyperV
```

그러면 이 스크립트가 Windows 컨테이너 구성 요소를 다운로드하여 구성하기 시작합니다. 이 프로세스는 다운로드 크기가 크기 때문에 다소 시간이 걸릴 수 있습니다. 프로세스 도중 컴퓨터가 재부팅될 수 있습니다. 완료되면 이제 컴퓨터에서 PowerShell과 Docker를 모두 사용하여 Windows 컨테이너와 Windows 컨테이너 이미지를 만들어 관리할 준비가 된 것입니다.

 이러한 항목이 완료되면 시스템에서 Windows 컨테이너에 대한 준비가 끝났습니다.

## 다음 단계: 컨테이너 사용 시작

이제 Windows 컨테이너 기능을 실행하는 Windows Server 2016 시스템이 준비되었으므로 다음 가이드로 이동하여 Windows Server와 Hyper-V 컨테이너 작업을 시작합니다.

[빠른 시작: Windows 컨테이너 및 Docker](./manage_docker.md)

[빠른 시작: Windows 컨테이너 및 PowerShell](./manage_powershell.md)




<!--HONumber=Feb16_HO2-->
