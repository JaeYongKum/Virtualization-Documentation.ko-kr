# PowerShell Direct를 사용하여 Windows 관리

PowerShell Direct를 사용하여 Windows 10 또는 Windows Server 기술 미리 보기 Hyper-V 호스트에서 원격으로 Windows 10 또는 Windows Server 기술 미리 보기 가상 컴퓨터를 관리할 수 있습니다. PowerShell Direct는 Hyper-V 호스트 또는 가상 컴퓨터에서 네트워크 구성 또는 원격 관리 설정에 관계 없이 가상 컴퓨터 내의 PowerShell 관리를 허용합니다. 이를 통해 Hyper-V 관리자는 스크립트 가상 컴퓨터 관리 및 구성을 더욱 쉽게 자동화할 수 있습니다.

PowerShell Direct를 실행하는 방법은 두 가지가 있습니다.
* 대화형 세션으로 -- [이 섹션으로 이동](vmsession.md#create-and-exit-an-interactive-powershell-session)하여 PSSession cmdlet을 사용하여 PowerShell Direct 세션을 만들고 종료합니다.
* 명령 또는 스크립트 집합을 실행하려면 -- [이 섹션으로 이동](vmsession.md#run-a-script-or-command-with-invoke-command)하여 Invoke-Command cmdlet으로 스크립트 또는 명령을 실행합니다.


## 요구 사항

**운영 체제 요구 사항:**
* 호스트 운영 체제는 Windows 10, Windows Server 기술 미리 보기 또는 더 높은 버전을 실행해야 합니다.
* 가상 컴퓨터는 Windows 10, Windows Server 기술 미리 보기 또는 더 높은 버전을 실행해야 합니다.

이전 가상 컴퓨터를 관리하는 경우 가상 컴퓨터 연결(VMConnect)을 사용하거나 [가상 컴퓨터에 대한 가상 네트워크를 구성합니다](http://technet.microsoft.com/library/cc816585.aspx).

가상 컴퓨터에서 PowerShell Direct 세션을 만들려면
* 가상 컴퓨터는 호스트에서 로컬로 실행 중이고 부팅되어야 합니다.
* Hyper-V 관리자로 호스트 컴퓨터에 로그인해야 합니다.
* 가상 컴퓨터에 대해 유효한 사용자 자격 증명을 제공해야 합니다.

## 대화형 PowerShell 세션 만들기 및 종료

1. Hyper-V 호스트에서 관리자 권한으로 Windows PowerShell을 엽니다.

3. 다음 명령 중 하나를 실행하여 가상 컴퓨터 이름 또는 GUID를 사용한 세션을 만듭니다.
``` PowerShell
Enter-PSSession -VMName <VMName>
Enter-PSSession -VMGUID <VMGUID>
```

4. 실행해야 하는 명령을 실행합니다. 이러한 명령은 세션을 만든 가상 컴퓨터에서 실행됩니다.
5. 완료되면 다음 명령을 실행하여 세션을 닫습니다.
``` PowerShell
Exit-PSSession 
```

> 참고: 세션이 연결되지 않는 경우 Hyper-V 호스트가 아닌 연결 중인 가상 컴퓨터에 대한 자격 증명을 사용하고 있는지 확인합니다.

이러한 cmdlet에 대한 자세한 내용은 [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) 및 [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx)을 참조하세요.

## Invoke-Command를 사용하여 스크립트 또는 명령 실행

[Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx) cmdlet을 사용하여 가상 컴퓨터에서 미리 결정된 명령 집합을 실행할 수 있습니다. 다음은 PSTest가 가상 컴퓨터 이름인 Invoke-Command cmdlet을 사용하는 방법의 예제이며 실행할 스크립트(foo.ps1)는 C:/ 드라이브의 스크립트 폴더에 있습니다.

 ``` PowerShell
 Invoke-Command -VMName PSTest -FilePath C:\script\foo.ps1 
 ```

단일 명령을 실행하려면 **-ScriptBlock** 매개 변수를 사용합니다.

 ``` PowerShell
 Invoke-Command -VMName PSTest -ScriptBlock { cmdlet } 
 ```

## 문제 해결

PowerShell direct를 통해 표시되는 작은 집합의 일반적인 오류 메시지가 있습니다. 다음은 가장 일반적인, 몇 가지 원인 및 문제를 진단하기 위한 도구입니다.

### 오류: 원격 세션이 종료됐을 수 있습니다.

오류 메시지:
```
An error has occured which Windows PowerShell cannot handle.  A remote session might have ended. 
```

가능한 원인:
* VM이 실행 중이 아닙니다.
* 게스트 OS는 PowerShell Direct를 지원하지 않습니다([요구 사항](#Requirements) 참조).
* PowerShell은 게스트에서 사용할 수 없습니다.
    * 운영 체제가 부팅을 완료하지 않았습니다.
    * 운영 체제가 올바르게 부팅할 수 없습니다.
    * 일부 부팅 시간 이벤트는 사용자 입력이 필요합니다.
* 게스트 자격 증명의 유효성을 검사할 수 없습니다.

[GET-VM](http://technet.microsoft.com/library/hh848479.aspx) cmdlet을 사용하여 사용 중인 자격 증명에 Hyper-V 관리자 역할이 있는지 확인하고 호스트에서 로컬로 실행 중이고 부팅된 VM을 확인할 수 있습니다.

## 샘플

[GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93)에서 샘플을 확인하세요.

사용자 환경에서 PowerShell Direct를 사용하는 방법의 다양한 예제와 PowerShell로 Hyper-V 스크립트를 작성하는 팁과 트릭에 대해 [PowerShell Direct 조각](../develop/powershell_snippets.md)을 참조하세요.



