# PowerShell 조각

PowerShell은 Hyper-V용 매력적인 스크립팅, 자동화 및 관리 도구입니다.  실행 가능한 유용한 기능 중 일부를 탐색하기 위한 도구 상자는 다음과 같습니다.

모든 Hyper-V 관리는 관리자 권한으로 실행되어야 하므로 모든 스크립트 및 조각은 Hyper-V Administrator 계정에서 관리자 권한으로 실행되어야 한다고 가정합니다.

적절한 사용 권한이 있는지 확실하지 않은 경우 `GET-VM`을 입력하고 오류 없이 실행되는 경우 준비가 완료된 것입니다.


## PowerShell Direct 도구

이 섹션의 모든 스크립트와 코드 조각은 다음 기본 사항에 의존합니다.

**요구 사항**:
*  PowerShell Direct. Windows 10 게스트 및 호스트 OS.

**일반적인 변수** :  
`$VMName` -- VMName의 문자열입니다. `GET-VM`으로 사용 가능한 VM의 목록 보기  
`$cred` -- 게스트 OS에 대한 자격 증명입니다. `$cred = Get-Credential`을 사용하여 채울 수 있음

### 게스트가 부팅되었는지 확인

Hyper-V 관리자는 게스트 OS가 부팅되었는지 여부를 확인하기 어려운 게스트 운영 체제에 대한 가시성을 제공하지 않습니다.

다음은 먼저 코드 조각으로 다음은 PowerShell 함수로 같은 기능의 두 가지 보기입니다.

코드 조각:
``` PowerShell
if((Invoke-Command -VMName $VMName -Credential $cred {"Test"}) -ne "Test"){Write-Host "Not Booted"} else {Write-Host "Booted"}
```

함수:
``` PowerShell
function waitForPSDirect([string]$VMName, $cred){
   Write-Output "[$($VMName)]:: Waiting for PowerShell Direct (using $($cred.username))"
   while ((Invoke-Command -VMName $VMName -Credential $cred {"Test"} -ea SilentlyContinue) -ne "Test") {Sleep -Seconds 1}}
```

**결과**  
VM에 대한 연결이 성공할 때까지 친숙한 메시지를 인쇄하고 잠금을 설정합니다.  
자동으로 성공합니다.

### 게스트가 네트워크를 가질 때까지 스크립트 잠금

PowerShell Direct를 사용하여 가상 컴퓨터가 IP 주소를 수신하기 전에 가상 컴퓨터 내 PowerShell 세션에 연결할 수 있습니다.

``` PowerShell
# Wait for DHCP
while ((Get-NetIPAddress | ? AddressFamily -eq IPv4 | ? IPAddress -ne 127.0.0.1).SuffixOrigin -ne "Dhcp") {sleep -Milliseconds 10}
```

** 결과 **
DHCP 임대를 받을 때까지 잠금을 설정합니다. 이 스크립트는 특정 서브넷 또는 IP 주소를 찾지 않으므로 사용 중인 네트워크 구성에 관계 없이 작동합니다.  
자동으로 성공합니다.

## Powershell로 자격 증명 관리

Hyper-V 스크립트는 하나 이상의 가상 컴퓨터, Hyper-V 호스트 또는 둘 모두에 대한 자격 증명 처리가 자주 필요합니다.

PowerShell Direct 또는 표준 PowerShell 원격 기능을 사용하는 경우 여러 가지 방법으로 이를 수행할 수 있습니다.

1. 첫 번째(및 가장 간단한) 방법은 호스트와 게스트 또는 로컬과 원격 호스트에서 유효한 동일한 사용자 자격 증명을 갖는 것입니다.  
    Microsoft 계정으로 로그인하는 경우 또는 도메인 환경에 있는 경우 상당히 쉽습니다.  
    이 시나리오에서는 `Invoke-Command -VMName "test" {get-process}`를 실행하기만 하면 됩니다.

2. 자격 증명을 묻는 메시지가 표시되는 PowerShell 사용  
    자격 증명이 일치하지 않는 경우 가상 컴퓨터에 대한 적절한 자격 증명을 제공하라는 자격 증명 프롬프트가 자동으로 표시됩니다.

3. 다시 사용할 수 있도록 변수에서 자격 증명을 저장합니다.
    다음과 같은 간단한 명령을 실행합니다.
  ``` PowerShell
  $localCred = Get-Credential
  ```
  그런 다음 다음과 같은 코드를 실행합니다.
  ``` PowerShell
  Invoke-Command -VMName "test" -Credential $localCred  {get-process} 
  ```
  자격 증명에 대한 스크립트/PowerShell 세션당 한 번 프롬프트가 표시되는 것을 의미합니다.

4. 자격 증명을 스크립트에 코딩합니다. **실제 작업 또는 시스템에 대해 실행하지 마십시오.**
    > 경고: _프로덕션 시스템에서 실행하지 마십시오. 실제 암호로 실행하지 마십시오.

    다음과 같은 몇 가지 코드로 PSCredential 개체를 개발할 수 있습니다.
  ``` PowerShell
  $localCred = New-Object -typename System.Management.Automation.PSCredential -argumentlist "Administrator", (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) 
  ```
  매우 안전하지 않지만 테스트에 유용합니다. 이제 이 세션에서는 아무런 프롬프트가 표시되지 않습니다.





