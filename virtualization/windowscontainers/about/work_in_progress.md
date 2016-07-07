---
title: "Windows 컨테이너 작업 진행 중"
description: "Windows 컨테이너 작업 진행 중"
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5d9f1cb4-ffb3-433d-8096-b085113a9d7b
redirect_url: ../containers_welcome
translationtype: Human Translation
ms.sourcegitcommit: e3f5535594123f6b4f8931e41a91d92f3b837814
ms.openlocfilehash: 085bb8c0158aedf4270cf2423114ec1901af1ebd

---

# 작업 진행 중

여기서 해결되지 않은 문제가 있거나 질문이 있는 경우 [포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)에 게시하세요.

-----------------------

## 일반 기능

### 컨테이너와 호스트 빌드 번호 일치
Windows 컨테이너는 빌드 및 패치 수준을 기준으로 컨테이너 호스트와 일치하는 운영 체제 이미지가 필요합니다. 불일치로 인해 잠재적 불안정 및 또는 컨테이너 및/또는 호스트에 대한 예기치 않은 동작이 발생합니다.

Windows 컨테이너 호스트 OS에 대해 업데이트를 설치하는 경우 일치하는 업데이트를 갖도록 컨테이너 기반 OS 이미지를 업데이트해야 합니다.
<!-- Can we give examples of behavior or errors?  Makes it more searchable -->

**해결 방법:**   
컨테이너 호스트의 OS 버전 및 패치 수준과 일치하는 컨테이너 기반 이미지를 다운로드하고 설치합니다.

### 기본 방화벽 동작
컨테이너 호스트 및 컨테이너 환경에서는 컨테이너 호스트의 방화벽만을 가집니다. 컨테이너 호스트에 구성된 모든 방화벽 규칙은 모든 해당 컨테이너에 전파됩니다.

### Windows 컨테이너가 느리게 시작합니다.
컨테이너가 시작하는 데 30초 이상이 소요되는 경우 중복된 많은 바이러스 검사가 수행되고 있을 수 있습니다.

Windows Defender와 같은 많은 맬웨어 방지 솔루션은 컨테이너 OS 이미지의 모든 OS 바이너리와 파일을 포함하여 컨테이너 이미지의 파일을 불필요하게 검사하고 있을 수 있습니다.  이는 새 컨테이너가 생성될 때 맬웨어 방지의 관점에서 모든 "컨테이너 파일"이 이전에 검사되지 않은 새 파일과 같이 보이는 경우 발생합니다.  따라서 컨테이너 내의 프로세스가 이러한 파일을 읽으려고 할 때 맬웨어 방지 구성 요소는 파일에 대한 액세스를 허용하기 전에 먼저 검사합니다.  실제로 이러한 파일은 컨테이너 이미지를 서버에 가져올 때 이미 검사되었습니다. Windows Server Technical Preview 5에서는 Windows Defender를 포함한 해당 맬웨어 방지 솔루션이 이러한 상황을 인식하고 적절하게 작동하여 여러 검사를 방지할 수 있도록 인프라를 적절히 사용할 수 있습니다. 맬웨어 방지 솔루션은 [여기](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)에 설명된 지침을 사용하여 솔루션을 업데이트하고 여러 검사를 방지할 수 있습니다. 

### 메모리가 48MB 미만으로 제한되는 경우 시작/중지가 실패합니다.
메모리가 48MB 미만으로 제한되는 경우 Windows 컨테이너는 임의의 비일관적인 오류를 경험합니다.

다음 PowerShell을 실행하고 시작, 중지, 작업을 여러 번 반복하는 경우 시작 또는 중지 중 하나에서 실패를 야기합니다.

```PowerShell
new-container "Test" -containerimagename "WindowsServerCore" -MaximumBytes 32MB
start-container test
stop-container test
```

**해결 방법:**  
메모리 값을 48MB로 변경합니다. 


### 프로세서 수가 4코어 VM에서 1 또는 2인 경우 컨테이너 시작에 실패합니다.

Windows 컨테이너는 다음 오류로 시작에 실패합니다.  
`failed to start: This operation returned because the timeout period expired. (0x800705B4).`

이는 프로세서 수가 4코어 VM에서 1 또는 2로 설정되었을 때 발생합니다.

``` PowerShell
new-container "Test2" -containerimagename "WindowsServerCore"
Set-ContainerProcessor -ContainerName test2 -Maximum 2
Start-Container test2

Start-Container : 'test2' failed to start.
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4).
'test2' failed to start. (Container ID 133E9DBB-CA23-4473-B49C-441C60ADCE44)
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4). (Container ID
133E9DBB-CA23-4473-B49C-441C60ADCE44)
At line:1 char:1
+ Start-Container test2
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (:) [Start-Container], VirtualizationException
    + FullyQualifiedErrorId : OperationTimeout,Microsoft.Containers.PowerShell.Cmdlets.StartContainer
PS C:\> Set-ContainerProcessor -ContainerName test2 -Maximum 3
PS C:\> Start-Container test2
```

**해결 방법:**  
컨테이너에 사용할 수 있는 프로세서를 늘립니다. 컨테이너에 사용할 수 있는 프로세서를 명시적으로 지정하거나 VM에 사용할 수 있는 프로세서를 줄이지 마세요.

--------------------------

## 네트워킹

### 네트워크 구획 격리 및 의미
각 컨테이너는 네트워크 구획을 사용하여 격리를 제공합니다. 지정된 컨테이너에 연결된 모든 컨테이너 네트워크 어댑터(끝점)는 동일한 네트워크 구획에 상주하게 됩니다. 사용된 네트워킹 모드(드라이버)에 따라 동일한 IP 주소나 포트를 사용하여 두 개의 다른 컨테이너 끝점에 액세스하지 못할 수 있습니다. 또한 Windows 방화벽 규칙은 구획이 아니거나 컨테이너를 인식하지 못하므로 연결된 모든 방화벽 규칙이 특정 끝점에 관계없이 컨테이너 호스트의 모든 컨테이너에 적용됩니다.

*** 투명 네트워킹 ***


*** NAT 네트워킹 *** 끝점 단위로 적용되는 NAT 포트 전달 규칙을 사용하여 단일 컨테이너에 연결된 여러 끝점이 노출될 수 있습니다. 이러한 전달 규칙은 컨테이너의 동일한 내부 포트에 매핑될 때 컨테이너 호스트의 서로 다른 외부 포트를 사용해야 합니다.  그러나 위에서 설명한 대로 연결된 방화벽 규칙은 컨테이너 호스트에서 전역 범위를 갖게 됩니다.



### 정적 NAT 매핑이 Docker를 통한 포트 매핑과 충돌 가능
Windows Server Technical Preview 5부터는 NAT 만들기 및 포트 매핑 규칙이 *ContainerNetwork* cmdlet 및 docker 명령에 통합됩니다. Windows HNS(호스트 네트워크 서비스)에서 자동으로 NAT를 관리합니다. 그러나 외부 클라이언트는 HNS에서 만든 동일한 NAT를 사용하여 중복된 포트 매핑 규칙을 시도하고 만들 수 있습니다.


다음은 포트 80에서 발생하는 정적 매핑과의 충돌 및 이런 경우 Docker에서 보고하는 오류에 대한 예입니다.
```
C:\Users\Administrator>docker run -it -p 80:80 windowsservercore cmd
docker: Error response from daemon: failed to create endpoint berserk_bassi on network nat: hnsCall failed in Win32: The remote procedure call failed. (0x6be).
```


***완화*** 일반적으로 HNS에서 NAT를 관리하므로 발생할 가능성이 거의 없습니다. `docker run -p <external>:<internal>` 또는 Add-ContainerNetworkAdapterStaticMapping을 사용하여 모든 포트 전달/매핑 규칙을 만들어야 합니다. 그러나 매핑이 HNS에서 자동으로 정리되지 않는 경우 PowerShell을 사용하여 포트 매핑을 제거하면 이 오류를 해결할 수 있습니다. 그러면 위의 예에서 발생한 포트 80이 제거됩니다.
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Windows 컨테이너가 IP를 가져오지 않습니다.
DHCP VM 스위치로 Windows 컨테이너에 연결하는 경우 컨테이너가 하지 않는 동안 컨테이너 호스트는 IP를 수신할 수 있습니다.

컨테이너에서 169.254.***.*** APIPA IP 주소를 가져옵니다.

**해결 방법:** 이는 커널 공유의 부작용으로 나타납니다.  모든 컨테이너는 동일한 mac 주소를 가지고 있습니다.

컨테이너 호스트 VM을 호스트하는 물리적 호스트에서 MAC 주소 스푸핑을 사용하도록 설정합니다.

PowerShell을 사용하여 수행할 수 있습니다
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
--------------------------

## 응용 프로그램 호환성

Windows 컨테이너에서 어떤 응용 프로그램이 작동하고 작동하지 않는지에 대한 많은 질문이 있습니다. 따라서 응용 프로그램 호환성 정보를 [고유한 문서](../reference/app_compat.md)로 구분하기로 결정했습니다.

가장 일반적인 문제 중 일부는 여기에서 확인할 수 있습니다.

### 이벤트 로그는 컨테이너 내에서 사용할 수 없습니다.

`get-eventlog Application` 같은 PowerShell 명령 및 이벤트 로그를 쿼리하는 API에서 다음과 같은 오류를 반환합니다.
```
get-eventlog : Cannot open log Application on machine .. Windows has not provided an error code.
At line:1 char:1
```

해결 방법으로 다음 단계를 Dockerfile에 추가할 수 있습니다. 이 단계로 작성된 이미지에서 이벤트 로깅을 사용하도록 설정해야 합니다.
```
RUN powershell.exe -command Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\WMI\Autologger\EventLog-Application Start 1
```


### localdb 인스턴스 api 메서드 호출 내에서 예기치 않은 오류가 발생했습니다.
localdb 인스턴스 api 메서드 호출 내에서 예기치 않은 오류가 발생했습니다.

### RTerm이 작동하지 않습니다.
RTerm이 설치되지만 Windows Server 컨테이너에서 시작되지 않습니다.

오류:  
```
'C:\Program' is not recognized as an internal or external command,
operable program or batch file.
```


### 컨테이너: Visual C++ Runtime x64/x86 2015가 설치되지 않았습니다.

관찰된 동작: 컨테이너:
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

중복 제거 필터와의 상호 운용성 문제입니다. 중복 제거는 이름 바꾸기 대상이 중복 제거된 파일인지 확인합니다. Windows Server 컨테이너 필터가 위의 중복 제거에 있으므로 `STATUS_IO_REPARSE_TAG_NOT_HANDLED`로 생성이 실패합니다.


컨테이너화할 수 있는 응용 프로그램에 대한 자세한 내용은 [응용 프로그램 호환성 문서](../reference/app_compat.md)를 참조하세요.

--------------------------


## Docker 관리

### 모든 Docker 명령이 작동하지 않습니다.
* Docker exec가 Hyper-V 컨테이너에서 작동하지 않습니다.

이 실패 목록에 있지 않은 경우(또는 명령이 예상과 다르게 실패하는 경우) [포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)을 통해 알려 주세요.

### 대화형 Docker 세션에 대한 명령 붙여 넣기는 50자로 제한됩니다.
대화형 Docker 세션에 대한 명령 붙여 넣기는 50자로 제한됩니다.  
대화형 Docker 세션에 명령줄을 복사하는 경우 현재 50자로 제한됩니다. 붙여 넣은 문자열이 잘립니다.

의도적이 아니며 제한 해지를 진행 중입니다.

### 넷 사용 오류
넷 사용은 사용자 이름 또는 암호를 묻는 메시지를 표시하는 대신 시스템 오류 1223을 반환합니다.

**해결 방법:**  
넷 사용을 실행하는 경우 사용자 이름 및 암호를 지정합니다.

``` PowerShell
net use S: \\your\sources\here /User:shareuser [yourpassword]
``` 


--------------------------



## 원격 데스크톱 

TP5에서 RDP 세션을 통해 Windows 컨테이너를 관리/상호 작용할 수 없습니다.

--------------------------

### Nano Server 컨테이너 호스트에서 컨테이너 종료를 "exit"로 실행할 수 없음
Nano Server 컨테이너 호스트에 있는 컨테이너를 종료할 때 "exit"를 사용하면 Nano Server 컨테이너 호스트와 연결이 끊어지고 컨테이너가 종료되지 않습니다.

**해결 방법:** 대신 Exit-PSSession을 사용하여 컨테이너를 종료합니다.

[포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)에서 기능을 요청하세요. 


--------------------------


## 사용자 및 도메인

### 로컬 사용자
로컬 사용자 계정을 만들어 컨테이너에서 Windows 서비스 및 응용 프로그램을 실행하는 데 사용할 수 있습니다.


### 도메인 멤버 자격
컨테이너는 Active Directory 도메인에 가입할 수 없으며 도메인 사용자, 서비스 계정 또는 컴퓨터 계정으로 서비스 또는 응용 프로그램을 실행할 수 없습니다. 

컨테이너는 대체로 환경과 관계 없는 알려진 일관된 상태로 신속하게 시작하도록 설계되었습니다. 도메인에 가입하고 도메인에서 그룹 정책 설정을 적용하면 컨테이너를 시작하는 데 걸리는 시간이 늘고, 시간에 따라 컨테이너 작동 방식이 변경되고, 개발자 간과 배포 사이에 이미지를 이동하거나 공유하는 기능이 제한됩니다.

Microsoft에서는 서비스 및 응용 프로그램이 Active Directory를 사용하는 방식에 대한 피드백과 컨테이너에서 이들 배포의 공통 영역을 신중히 고려하고 있습니다. 가장 효율적인 구성에 대한 세부 정보가 있는 경우 [포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)에서 공유해 주세요. 

이러한 유형의 시나리오를 지원하는 솔루션을 적극적으로 찾고 있습니다.



<!--HONumber=Jun16_HO5-->


