---
title: PowerShell Direct를 사용하여 Windows 가상 컴퓨터 관리
description: PowerShell Direct를 사용하여 Windows 가상 컴퓨터 관리
keywords: windows 10, hyper-v, powershell, 통합 서비스, 통합 구성 요소, 자동화, powershell direct
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
ms.openlocfilehash: 779dcf51d4903c9467cc52dbadb865beb9929bd2
ms.sourcegitcommit: e7fa38bcb7744a34e7a58978b55af1fbf6353247
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/20/2018
---
# <a name="virtual-machine-automation-and-management-using-powershell"></a>PowerShell을 사용하여 가상 컴퓨터 자동화 및 관리
 
PowerShell Direct를 사용하여 네트워크 구성 또는 원격 관리 설정에 상관 없이 Hyper-V 호스트의 Windows 10 또는 Windows Server 2016 가상 머신에서 임의의 PowerShell을 실행할 수 있습니다.

**PowerShell Direct 실행 방법:**  
* 대화형 세션 -- Enter-PSSession을 사용하여 대화형 PowerShell 세션을 만들고 종료하려면 [여기를 클릭](#create-and-exit-an-interactive-powershell-session)하세요.
* 단일 명령이나 스크립트를 실행하는 일회용 세션 -- Invoke-Command를 사용하여 스크립트나 명령을 실행하려면 [여기를 클릭](#run-a-script-or-command-with-invoke-command)하세요.
* 영구 세션(빌드 14280 이상) -- New-PSSession을 사용하여 영구 세션을 만들려면 [여기를 클릭](#copy-files-with-new-pssession-and-copy-item)하세요.  
Copy-Item을 사용하여 가상 컴퓨터 간에 계속 파일을 복사한 다음 Remove-PSSession을 사용하여 연결을 끊습니다.

## <a name="requirements"></a>요구 사항
**운영 체제 요구 사항:**
* 호스트: Hyper-V를 실행하는 Windows 10, Windows Server 2016 이상.
* 게스트/가상 머신: Windows 10, Windows Server 2016 이상.

이전 가상 컴퓨터를 관리하는 경우 가상 컴퓨터 연결(VMConnect)을 사용하거나 [가상 컴퓨터에 대한 가상 네트워크를 구성합니다](http://technet.microsoft.com/library/cc816585.aspx). 

**구성 요구 사항:**    
* 가상 컴퓨터는 호스트에서 로컬로 실행되고 있어야 합니다.
* 하나 이상의 구성된 사용자 프로필로 가상 컴퓨터를 켜고 실행해야 합니다.
* Hyper-V 관리자로 호스트 컴퓨터에 로그인해야 합니다.
* 가상 컴퓨터에 대해 유효한 사용자 자격 증명을 제공해야 합니다.

-------------

## <a name="create-and-exit-an-interactive-powershell-session"></a>대화형 PowerShell 세션 만들기 및 종료

가상 컴퓨터에서 PowerShell 명령을 실행하는 가장 쉬운 방법은 대화형 세션을 시작하는 것입니다.

세션이 시작되면 입력한 명령이 가상 컴퓨터 자체의 PowerShell 세션에 직접 입력한 것처럼 가상 컴퓨터에서 실행됩니다.

**대화형 세션을 시작하려면**

1. Hyper-V 호스트에서 관리자 권한으로 PowerShell을 엽니다.

2. 다음 명령 중 하나를 실행하여 가상 컴퓨터 이름 또는 GUID를 사용하는 대화형 세션을 만듭니다.  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  메시지가 표시되면 가상 컴퓨터에 대한 자격 증명을 제공합니다.

3. 가상 컴퓨터에서 명령을 실행합니다.
  
  다음과 같이 PowerShell 프롬프트에 대한 접두사로 VMName이 표시됩니다.
  
  ``` 
  [VMName]: PS C:\>
  ```
  
  모든 명령 실행은 가상 컴퓨터에서 실행됩니다. 테스트하려면 `ipconfig`나 `hostname`을 실행하여 이러한 명령이 가상 컴퓨터에서 실행되고 있는지 확인할 수 있습니다.
  
4. 완료되면 다음 명령을 실행하여 세션을 닫습니다.  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> 참고: 세션이 연결되지 않으면 [문제 해결](#troubleshooting)에서 잠재적 원인을 확인하세요. 

이러한 cmdlet에 대한 자세한 내용은 [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) 및 [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx)을 참조하세요. 

-------------

## <a name="run-a-script-or-command-with-invoke-command"></a>Invoke-Command를 사용하여 스크립트 또는 명령 실행

Invoke-Command를 사용하는 PowerShell Direct는 가상 컴퓨터에서 하나의 명령이나 하나의 스크립트를 실행해야 하지만 그 후에는 가상 컴퓨터와 계속 상호 작용할 필요가 없는 경우에 매우 적합합니다.

**단일 명령을 실행하려면**

1. Hyper-V 호스트에서 관리자 권한으로 PowerShell을 엽니다.

2. 다음 명령 중 하나를 실행하여 가상 컴퓨터 이름 또는 GUID를 사용하는 세션을 만듭니다.  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { command } 
   Invoke-Command -VMId <VMId> -ScriptBlock { command }
   ```
   
   메시지가 표시되면 가상 컴퓨터에 대한 자격 증명을 제공합니다.
   
   명령이 가상 컴퓨터에서 실행되고 콘솔에 출력이 있을 경우 콘솔로 인쇄됩니다.  명령이 실행되는 즉시 연결이 자동으로 닫힙니다.
   
   
**스크립트를 실행하려면**

1. Hyper-V 호스트에서 관리자 권한으로 PowerShell을 엽니다.

2. 다음 명령 중 하나를 실행하여 가상 컴퓨터 이름 또는 GUID를 사용하는 세션을 만듭니다.  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   메시지가 표시되면 가상 컴퓨터에 대한 자격 증명을 제공합니다.
   
   스크립트는 가상 컴퓨터에서 실행됩니다.  명령이 실행되는 즉시 연결이 자동으로 닫힙니다.

이 cmdlet에 대한 자세한 내용은 [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx)를 참조하세요. 

-------------

## <a name="copy-files-with-new-pssession-and-copy-item"></a>New-PSSession 및 Copy-Item을 사용하여 파일 복사

> **참고:** PowerShell Direct는 Windows 빌드 14280 이상에서만 영구 세션을 지원합니다.

영구 PowerShell 세션은 하나 이상의 원격 컴퓨터에서 작업을 조정하는 스크립트를 작성할 때 매우 유용합니다.  만들어진 영구 세션은 삭제할 때까지 백그라운드에 존재합니다.  즉, 자격 증명을 전달하지 않고도 `Invoke-Command` 또는 `Enter-PSSession`을 사용하여 동일한 세션을 반복해서 참조할 수 있습니다.

세션은 동일한 토큰별로 상태를 저장합니다.  영구 세션은 계속 유지되므로 세션에서 만들거나 세션에 전달된 모든 변수가 여러 호출에서 그대로 유지됩니다. 영구 세션 작업에는 많은 도구를 사용할 수 있습니다.  이 예제에서는 [New-PSSession](https://technet.microsoft.com/en-us/library/hh849717.aspx) 및 [Copy-Item](https://technet.microsoft.com/en-us/library/hh849793.aspx)을 사용하여 호스트에서 가상 컴퓨터로, 가상 컴퓨터에서 호스트로 데이터를 이동합니다.

**세션을 만든 다음 파일을 복사하려면**  

1. Hyper-V 호스트에서 관리자 권한으로 PowerShell을 엽니다.

2. 다음 명령 중 하나를 실행하여 `New-PSSession`으로 가상 컴퓨터에 대한 영구 PowerShell 세션을 만듭니다.
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  메시지가 표시되면 가상 컴퓨터에 대한 자격 증명을 제공합니다.
  
  > **경고:**  
   14500 이전 빌드에는 버그가 있습니다.  `-Credential` 플래그를 사용하여 자격 증명을 명시적으로 지정하지 않으면 게스트의 서비스가 작동 중단되어 다시 시작해야 합니다.  이 문제가 발생할 경우 [여기](#error-a-remote-session-might-have-ended)에서 해결 지침을 사용할 수 있습니다.
  
3. 가상 컴퓨터에 파일을 복사합니다.
  
  `C:\host_path\data.txt`를 호스트 컴퓨터에서 가상 컴퓨터에 복사하려면 다음을 실행합니다.
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  가상 컴퓨터에서 호스트로 파일을 복사합니다. 
   
   `C:\guest_path\data.txt`를 가상 컴퓨터에서 호스트에 복사하려면 다음을 실행합니다.
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. `Remove-PSSession`을 사용하여 영구 세션을 중지합니다.
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## <a name="troubleshooting"></a>문제 해결

PowerShell Direct를 통해 표시되는 작은 집합의 일반적인 오류 메시지가 있습니다.  다음은 가장 일반적인, 몇 가지 원인 및 문제를 진단하기 위한 도구입니다.

### <a name="-vmname-or--vmid-parameters-dont-exist"></a>-VMName 또는 -VMID 매개 변수가 없습니다.
**문제:**  
`Enter-PSSession`, `Invoke-Command` 또는 `New-PSSession`에 `-VMName` 또는 `-VMId` 매개 변수가 없습니다.

**가능한 원인:**  
가장 가능성이 높은 문제는 PowerShell Direct가 호스트 운영 체제에서 지원되지 않는다는 것입니다.

다음 명령을 실행하여 Windows 빌드를 확인할 수 있습니다.

``` PowerShell
[System.Environment]::OSVersion.Version
```

지원되는 빌드가 실행되고 있는 경우 PowerShell 버전에서 PowerShell Direct를 실행하지 못할 수도 있습니다.  PowerShell Direct 및 JEA의 경우 주 버전이 5 이상이어야 합니다.

다음 명령을 실행하여 PowerShell 버전 빌드를 확인할 수 있습니다.

``` PowerShell
$PSVersionTable.PSVersion
```


### <a name="error-a-remote-session-might-have-ended"></a>오류: 원격 세션이 종료됐을 수 있습니다.
> **참고:**  
호스트 빌드 10240과 12400 사이의 Enter-PSSession에 대해 아래의 모든 오류가 " 원격 세션이 종료되었을 수 있습니다."로 보고되었습니다.

**오류 메시지:**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**가능한 원인:**
* 가상 컴퓨터가 존재하지만 실행되고 있지 않습니다.
* 게스트 OS는 PowerShell Direct를 지원하지 않습니다([요구 사항](#Requirements) 참조).
* PowerShell은 게스트에서 사용할 수 없습니다.
  * 운영 체제가 부팅을 완료하지 않았습니다.
  * 운영 체제가 올바르게 부팅할 수 없습니다.
  * 일부 부팅 시간 이벤트는 사용자 입력이 필요합니다.

[Get-VM](http://technet.microsoft.com/library/hh848479.aspx) cmdlet을 사용하여 호스트에서 실행되고 있는 VM을 확인할 수 있습니다.

**오류 메시지:**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**가능한 원인:**
* 위에 나열된 원인 중 하나. 모든 원인이 다음 명령에 동일하게 적용됩니다. `New-PSSession`  
* `-Credential`을 사용하여 자격 증명을 명시적으로 전달해야 하는 현재 빌드의 버그입니다.  이런 경우 전체 서비스가 게스트 운영 체제에서 중단되므로 다시 시작해야 합니다.  Enter-PSSession에서 세션을 계속 사용할 수 있는지 확인할 수 있습니다.

자격 증명 문제를 해결하려면 VMConnect를 사용하여 가상 컴퓨터에 로그인하고 PowerShell을 연 후 다음 PowerShell을 사용하여 vmicvmsession 서비스를 다시 시작합니다.

``` PowerShell
Restart-Service -Name vmicvmsession
```

### <a name="error-parameter-set-cannot-be-resolved"></a>오류: 매개 변수 집합을 확인할 수 없습니다.
**오류 메시지:**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**가능한 원인:**  
* `-RunAsAdministrator` 명령이 가상 컴퓨터에 연결할 때 지원되지 않습니다.
     
  Windows 컨테이너에 연결할 경우에는 `-RunAsAdministrator` 플래그에서 명시적 자격 증명 없이 관리자 연결을 허용합니다.  가상 컴퓨터는 호스트에 암시적 관리자 액세스 권한을 제공하지 않으므로 자격 증명을 명시적으로 입력해야 합니다.

관리자 자격 증명을 `-Credential` 매개 변수를 사용하여 가상 컴퓨터에 전달하거나 메시지가 표시될 때 수동으로 입력하여 전달할 수 있습니다.


### <a name="error-the-credential-is-invalid"></a>오류: 자격 증명이 잘못되었습니다.

**오류 메시지:**  
```
Enter-PSSession : The credential is invalid.
```

**가능한 원인:** 
* 게스트 자격 증명의 유효성을 검사할 수 없습니다.
  * 제공된 자격 증명이 잘못되었습니다.
  * 게스트에 사용자 계정이 없습니다.(OS가 이전에 부팅되지 않았습니다)
  * 관리자 권한으로 연결하는 경우: 관리자가 활성 사용자로 설정되지 않았습니다.  [여기](https://technet.microsoft.com/en-us/library/hh825104.aspx)에서 자세한 내용을 알아보세요.
  
### <a name="error-the-input-vmname-parameter-does-not-resolve-to-any-virtual-machine"></a>오류: 입력 VMName 매개 변수가 가상 컴퓨터로 확인되지 않습니다.

**오류 메시지:**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**가능한 원인:**  
* Hyper-V 관리자가 아닙니다.  
* 가상 컴퓨터가 없습니다.

[Get-VM](http://technet.microsoft.com/library/hh848479.aspx) cmdlet을 사용하여 사용 중인 자격 증명에 Hyper-V 관리자 역할이 있는지 확인하고 호스트에서 로컬로 실행 중이고 부팅된 VM을 확인할 수 있습니다.


-------------

## <a name="samples-and-user-guides"></a>샘플 및 사용자 가이드

PowerShell Direct는 JEA(Just Enough Administration)를 지원합니다.  사용해 보려면 이 사용자 가이드를 확인하세요.

[GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93)에서 샘플을 확인하세요.
