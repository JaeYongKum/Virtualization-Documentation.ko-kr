# Windows 컨테이너 빠른 시작 - PowerShell

Windows 컨테이너는 단일 컴퓨터 시스템에서 격리된 여러 응용 프로그램을 신속하게 배포하는 데 사용할 수 있습니다. 이 빠른 시작은 PowerShell을 사용한 Windows 서버와 Hyper-V 컨테이너의 배포 및 관리를 보여 줍니다. 이 연습에서 Windows Server와 Hyper- V 컨테이너에서 실행 중인 매우 간단한 'hello world' 응용 프로그램을 기초부터 만들 수 있습니다. 이 과정에서 컨테이너 공유 폴더와 함께 사용할 컨테이너 이미지를 만들고 컨테이너 수명 주기를 관리합니다. 완료되면 Widows 컨테이너 배포 및 관리에 대한 기본적인 이해를 하게 됩니다.

이 연습에서는 Windows Server 컨테이너와 Hyper-V 컨테이너를 자세히 설명합니다. 컨테이너의 각 형식에는 자체 기본 요구 사항이 있습니다. 컨테이너 호스트를 신속하게 배포하는 절차는 Windows 컨테이너 설명서에 포함되어 있습니다. 이것이 Windows 컨테이너를 신속하게 시작하는 가장 쉬운 방법입니다. 컨테이너 호스트가 아직 없는 경우 [컨테이너 호스트 배포 빠른 시작](./container_setup.md)을 참조하세요.

다음 항목은 각 연습에 필요합니다.

**Windows Server 컨테이너:**

- 온-프레미스 또는 Azure에서 Windows Server 2016 Core를 실행하는 Windows 컨테이너 호스트입니다.

**Hyper-V 컨테이너:**

- 중첩된 가상화를 사용하는 Windows 컨테이너 호스트입니다.
- Windows Server 2016 미디어 - [다운로드](https://aka.ms/tp4/serveriso)

> Microsoft Azure는 Hyper-V 컨테이너를 지원하지 않습니다. Hyper-V 연습을 완료하려면 온-프레미스 컨테이너 호스트가 필요합니다.

## Windows Server 컨테이너

Windows Server 컨테이너는 실행 중인 응용 프로그램 및 호스팅 프로세스에 대한 격리된 휴대용, 리소스 제어된 운영 환경을 제공합니다. Windows Server 컨테이너는 프로세스와 네임스페이스 격리를 통해 컨테이너와 호스트 간, 호스트에서 실행하는 컨테이너 간의 격리를 제공합니다.

### 컨테이너 만들기

TP4 시 Windows Server 2016 또는 Windows Server 2016 Core에서 실행되는 Windows Server 컨테이너는 Windows Server 2016 Core OS 이미지가 필요합니다.

`powershell`을 입력하여 PowerShell 세션을 시작합니다.

```powershell
C:\> powershell
Windows PowerShell
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\>
```

Windows Server Core OS 이미지가 설치되었는지 확인하려면 `Get-ContainerImage` 명령을 사용합니다. 유효한 여러 OS 이미지를 볼 수 있습니다.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

Windows Server 컨테이너를 만들려면 `New-Container` 명령을 사용합니다. 아래 예제에서는 `WindowsServerCore` OS 이미지에서 `TP4Demo`라는 컨테이너를 만들고 `가상 스위치`라는 VM 스위치에 컨테이너를 연결합니다. 출력, 컨테이너를 나타내는 개체는 변수 `$con`에 저장됩니다. 이 변수는 후속 명령에 사용됩니다.

```powershell
PS C:\> New-Container -Name TP4Demo -ContainerImageName WindowsServerCore -SwitchName "Virtual Switch"

Name    State Uptime   ParentImageName
----    ----- ------   ---------------
TP4Demo Off   00:00:00 WindowsServerCore
```

`Start-Container` 명령을 사용하여 컨테이너를 시작합니다.

```powershell
PS C:\> Start-Container -Name TP4Demo
```

`Enter-PSSession` 명령을 사용하여 컨테이너에 연결합니다. PowerShell 세션이 컨테이너로 만들어진 경우 PowerShell 프롬프트는 컨테이너 이름에 맞게 변경합니다.

```powershell
PS C:\> Enter-PSSession -ContainerName TP4Demo -RunAsAdministrator

[TP4Demo]: PS C:\Windows\system32>
```

### IIS 이미지 만들기

이제 컨테이너를 수정할 수 있으므로 이러한 수정 사항은 새 컨테이너 이미지를 만들도록 캡처됩니다. 이 예제에서는 IIS가 설치되어 있습니다.

컨테이너에 IIS 역할을 설치하려면 `Install-WindowsFeature` 명령을 사용합니다.

```powershell
[TP4Demo]: PS C:\> Install-WindowsFeature web-server

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Common HTTP Features, Default Document, D...
```

IIS 설치가 완료되면 `exit`을 입력하여 컨테이너를 종료합니다. PowerShell 세션이 컨테이너 호스트의 세션으로 반환됩니다.

```powershell
[TP4Demo]: PS C:\> exit
PS C:\>
```

마지막으로 `Stop-Container` 명령을 사용하여 컨테이너를 중지합니다.

```powershell
PS C:\> Stop-Container -Name TP4Demo
```

이 컨테이너의 상태는 이제 새 컨테이너 이미지로 캡처될 수 있습니다. `New-ContainerImage` 명령을 사용합니다.

이 예제에서는 `데모`의 게시자와 버전 `1.0`으로 `WindowsServerCoreIIS`라는 새 컨테이너 이미지를 만듭니다.

```powershell
PS C:\> New-ContainerImage -ContainerName TP4Demo -Name WindowsServerCoreIIS -Publisher Demo -Version 1.0

Name                 Publisher Version IsOSImage
----                 --------- ------- ---------
WindowsServerCoreIIS CN=Demo   1.0.0.0 False
```

### IIS 컨테이너 만들기

이번에는 `WindowsServerCoreIIS` 컨테이너 이미지에서 새 컨테이너를 만듭니다.

```powershell
PS C:\> New-Container -Name IIS -ContainerImageName WindowsServerCoreIIS -SwitchName "Virtual Switch"

Name State Uptime   ParentImageName
---- ----- ------   ---------------
IIS  Off   00:00:00 WindowsServerCoreIIS
```
컨테이너를 시작합니다.

```powershell
PS C:\> Start-Container -Name IIS
```

### 네트워킹 구성

Windows 컨테이너 빠른 시작에 대한 기본 네트워크 구성은 NAT(Network Address Translation)로 구성된 가상 스위치에 연결하는 컨테이너를 갖는 것입니다. 이 때문에 컨테이너 내부, 컨테이너 호스트의 포트에서 실행 중인 응용 프로그램에 연결하기 위해 컨테이너의 포트에 매핑되어야 합니다.

이 연습에서 웹 사이트는 컨테이너 내부에서 실행 중인 IIS에서 호스팅됩니다. 포트 80에서 웹 사이트에 액세스하려면 컨테이너 호스트 IP 주소의 포트 80을 컨테이너 IP 주소의 포트 80에 매핑합니다.

다음을 실행하여 컨테이너의 IP 주소를 반환합니다.

```powershell
PS C:\> Invoke-Command -ContainerName IIS {ipconfig}

Windows IP Configuration


Ethernet adapter vEthernet (Virtual Switch-7570F6B1-E1CA-41F1-B47D-F3CA73121654-0):

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::ed23:c1c6:310a:5c10%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

NAT 포트 매핑을 만들려면 `Add-NetNatStaticMapping` 명령을 사용합니다. 다음 예제에서는 기존 포트 매핑 규칙을 확인하고 존재하지 않는 경우 만듭니다. `-InternalIPAddress`는 컨테이너의 IP 주소와 일치해야 합니다.

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```

포트 매핑을 만들면 구성된 포트에 대한 인바운드 방화벽 규칙을 구성해야 합니다. 포트 80에 이렇게 하려면 다음 스크립트를 실행합니다. 80 이외의 외부 포트에 대한 NAT 규칙을 만든 경우 방화벽 규칙은 일치하도록 만들어져야 합니다.

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

Azure에서 작업하고 네트워크 보안 그룹을 아직 만들지 않은 경우 지금 새로 만들어야 합니다. 네트워크 보안 그룹에 대한 자세한 내용은 [네트워크 보안 그룹이란](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-nsg/) 문서를 참조하세요.

### 응용 프로그램 만들기

이제 IIS 이미지에서 컨테이너가 만들어졌고 네트워킹이 구성되었으므로 브라우저를 열고 컨테이너 호스트의 IP 주소를 찾아봅니다. IIS 시작 화면이 표시됩니다.

![](media/iis1.png)

IIS 인스턴스가 실행 중으로 확인되면 이제 'Hello World' 응용 프로그램을 만들고 이를 IIS 인스턴스에 호스팅할 수 있습니다. 이렇게 하려면 컨테이너로 PowerShell 세션을 만듭니다.

```powershell
PS C:\> Enter-PSSession -ContainerName IIS -RunAsAdministrator
[IIS]: PS C:\Windows\system32>
```

다음 명령을 실행하여 IIS 시작 화면을 제거합니다.

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
다음 명령을 실행하여 기본 IIS 사이트를 새 정적 사이트로 바꿉니다.

```powershell
[IIS]: PS C:\> "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

컨테이너 호스트의 IP 주소를 다시 검색하면 'Hello World' 응용 프로그램이 표시됩니다. 업데이트된 응용 프로그램을 확인하기 위해 모든 기존 브라우저 연결을 닫거나 브라우저 캐시의 선택을 취소해야 할 수 있습니다.

![](media/HWWINServer.png)

원격 컨테이너 세션을 종료합니다.

```powershell
[IIS]: PS C:\> exit
PS C:\>
```

### 컨테이너 제거

컨테이너를 제거하기 전에 중지해야 합니다.

```powershell
PS C:\> Stop-Container -Name IIS
```

컨테이너가 중지되었을 때 `Remove-Container` 명령으로 제거할 수 있습니다.

```powershell
PS C:\> Remove-Container -Name IIS -Force
```

마지막으로 `Remove-ContainerImage` 명령을 사용하여 컨테이너 이미지를 제거할 수 있습니다.

```powershell
PS C:\> Remove-ContainerImage -Name WindowsServerCoreIIS -Force
```

## Hyper-V 컨테이너

Hyper-V 컨테이너는 Windows Server 컨테이너에 대한 추가 수준의 격리를 제공합니다. 각 Hyper-V 컨테이너는 고도로 최적화된 가상 컴퓨터 내에서 만들어집니다. Windows Server 컨테이너가 컨테이너 호스트의 커널 및 해당 호스트에서 실행되는 다른 모든 Windows Server 컨테이너를 공유하는 곳에서 Hyper-V 컨테이너가 다른 컨테이너에서 완전히 격리됩니다. Hyper-V 컨테이너는 Windows Server 컨테이너와 동일하게 만들어지고 관리됩니다. Hyper-V 컨테이너에 대한 자세한 내용은 [Hyper-V 컨테이너 관리](../management/hyperv_container.md)를 참조하세요.

> Microsoft Azure는 Hyper-V 컨테이너를 지원하지 않습니다. Hyper-V 컨테이너 연습을 완료하려면 온-프레미스 컨테이너 호스트가 필요합니다.

### 컨테이너 만들기

TP4 시 Hyper-V 컨테이너는 Nano Server Core OS 이미지를 사용해야 합니다. Nano Server OS 이미지가 설치되었는지 확인하려면 `Get-ContainerImage` 명령을 사용합니다.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

Hyper-V 컨테이너를 만들려면 HyperV의 런타임을 지정하는 `New-Container` 명령을 사용합니다.

```powershell
PS C:\> New-Container -Name HYPV -ContainerImageName NanoServer -SwitchName "Virtual Switch" -RuntimeType HyperV

Name State Uptime   ParentImageName
---- ----- ------   ---------------
HYPV Off   00:00:00 NanoServer
```

컨테이너가 만들어지면 **시작하지 마십시오**.

### 공유 폴더 만들기

공유 폴더는 컨테이너 호스트에서 컨테이너로 디렉터리를 노출합니다. 공유 폴더가 만들어지면 공유 폴더에 있는 모든 파일을 컨테이너에 사용할 수 있습니다. 공유 폴더는 컨테이너로 Nano Server IIS 패키지를 복사하도록 이 예제에 사용됩니다. 그런 다음 해당 패키지는 IIS를 설치하는 데 사용됩니다. 공유 폴더에 대한 자세한 내용은 [컨테이너 데이터 관리](../management/manage_data.md)를 참조하세요.

컨테이너 호스트에 `c:\share\en-us`라는 디렉터리를 만듭니다.

```powershell
S C:\> New-Item -Type Directory c:\share\en-us

    Directory: C:\share

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

`Add-ContainerSharedFolder` 명령을 사용하여 새 컨테이너에 새 공유 폴더를 만듭니다.

> 컨테이너는 공유 폴더를 만들 때 중지 상태에 있어야 합니다.

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName HYPV -SourcePath c:\share -DestinationPath c:\iisinstall

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
HYPV          c:\share   c:\iisinstall   ReadWrite
```

공유 폴더가 만들어지면 컨테이너를 시작합니다.

```powershell
PS C:\> Start-Container -Name HYPV
```
`Enter-PSSession` 명령을 사용하여 컨테이너로 PowerShell 원격 세션을 만듭니다.

```powershell
PS C:\> Enter-PSSession -ContainerName HYPV -RunAsAdministrator
[HYPV]: PS C:\windows\system32\config\systemprofile\Documents>cd /
```
그러나 원격 세션에서 작업하는 경우 공유 폴더 `c:\iisinstall\en-us`가 생성되었지만 비어 있습니다.

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

### IIS 이미지 만들기

컨테이너는 Nano Server OS 이미지를 실행하기 때문에 IIS를 설치하려면 Nano Server IIS 패키지가 필요합니다. 이는 `NanoServer\Packages` 디렉터리 아래의 Windows Sever 2016 TP4 설치 미디어에서 찾을 수 있습니다.

`NanoServer\Packages`에서 `Microsoft-NanoServer-IIS-Package.cab`을 컨테이너 호스트의 `c:\share`로 복사합니다.

`NanoServer\Packages\en-us\Microsoft-NanoServer-IIS-Package.cab`을 컨테이너 호스트의 `c:\share\en-us`로 복사합니다.

c:\share 폴더에 unattend.xml이라는 파일을 만들고 이 텍스트를 unattend.xml 파일로 복사합니다.

```powershell
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <servicing>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" />
            <source location="c:\iisinstall\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="en-US" />
            <source location="c:\iisinstall\en-us\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
    </servicing>
</unattend>
```

완료되면 컨테이너 호스트의 `c:\share` 디렉터리는 다음과 같이 구성되어야 합니다.

```
c:\share
|-- en-us
|    |-- Microsoft-NanoServer-IIS-Package.cab
|
|-- Microsoft-NanoServer-IIS-Package.cab
|-- unattend.xml
```

컨테이너의 원격 세션으로 돌아가서 이제 IIS 패키지 및 unattended.xml 파일은 c:\iisinstall 디렉터리에 표시됩니다.

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:32 PM                en-us
-a----       10/29/2015  11:51 PM        1922047 Microsoft-NanoServer-IIS-Package.cab
-a----       11/18/2015   5:31 PM            789 unattend.xml
```

다음 명령을 실행하여 IIS를 설치합니다.

```powershell
[HYPV]: PS C:\> dism /online /apply-unattend:c:\iisinstall\unattend.xml

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0


[                           1.0%                           ]

[=====                      10.1%                          ]

[=====                      10.3%                          ]

[===============            26.2%                          ]
```

IIS 설치가 완료되면 다음 명령을 사용하여 수동으로 IIS를 시작합니다.

```powershell
[HYPV]: PS C:\> Net start w3svc
The World Wide Web Publishing Service service is starting.
The World Wide Web Publishing Service service was started successfully.
```

컨테이너 세션을 종료합니다.

```powershell
[HYPV]: PS C:\> exit
```

컨테이너를 중지합니다.

```powershell
PS C:\> Stop-Container -Name HYPV
```

이 컨테이너의 상태는 이제 새 컨테이너 이미지로 캡처될 수 있습니다.

이 예제에서는 `데모`의 게시자와 버전 `1.0`으로 `NanoServerIIS`라는 새 컨테이너 이미지를 만듭니다.

```powershell
PS C:\> New-ContainerImage -ContainerName HYPV -Name NanoServerIIS -Publisher Demo -Version 1.0

Name          Publisher Version IsOSImage
----          --------- ------- ---------
NanoServerIIS CN=Demo   1.0.0.0 False
```

### IIS 컨테이너 만들기

`New-Container` 명령을 사용하여 IIS 이미지에서 새 Hyper-V 컨테이너를 만듭니다.

```powershell
PS C:\> New-Container -Name IISApp -ContainerImageName NanoServerIIS -SwitchName "Virtual Switch" -RuntimeType HyperV

Name   State Uptime   ParentImageName
----   ----- ------   ---------------
IISApp Off   00:00:00 NanoServerIIS
```

컨테이너를 시작합니다.

```powershell
PS C:\> Start-Container -Name IISApp
```

### 네트워킹 구성

Windows 컨테이너 빠른 시작에 대한 기본 네트워크 구성은 NAT(Network Address Translation)로 구성된 가상 스위치에 연결하는 컨테이너를 갖는 것입니다. 이 때문에 컨테이너 내부, 컨테이너 호스트의 포트에서 실행 중인 응용 프로그램에 연결하기 위해 컨테이너의 포트에 매핑되어야 합니다.

이 연습에서 웹 사이트는 컨테이너 내부에서 실행 중인 IIS에서 호스팅됩니다. 포트 80에서 웹 사이트에 액세스하려면 컨테이너 호스트 IP 주소의 포트 80을 컨테이너 IP 주소의 포트 80에 매핑합니다.

다음을 실행하여 컨테이너의 IP 주소를 반환합니다.

```powershell
PS C:\> Invoke-Command -ContainerName IISApp {ipconfig}

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::c574:5a5e:d5f5:18a0%4
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

NAT 포트 매핑을 만들려면 `Add-NetNatStaticMapping` 명령을 사용합니다. 다음 예제에서는 기존 포트 매핑 규칙을 확인하고 존재하지 않는 경우 만듭니다. `-InternalIPAddress`는 컨테이너의 IP 주소와 일치해야 합니다.

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```
컨테이너 호스트에서 포트 80을 열어야 합니다. 80 이외의 외부 포트에 대한 NAT 규칙을 만든 경우 방화벽 규칙은 일치하도록 만들어져야 합니다.

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

### 응용 프로그램 만들기

이제 IIS 이미지에서 컨테이너가 만들어졌고 네트워킹이 구성되었으므로 브라우저를 열고 컨테이너 호스트의 IP 주소를 찾아봅니다. IIS 시작 화면이 표시됩니다.

![](media/iis1.png)

IIS 인스턴스가 실행 중으로 확인되면 이제 'Hello World' 응용 프로그램을 만들고 이를 IIS 인스턴스에 호스팅할 수 있습니다. 이렇게 하려면 컨테이너로 PowerShell 세션을 만듭니다.

```powershell
PS C:\> Enter-PSSession -ContainerName IISApp -RunAsAdministrator
[IISApp]: PS C:\windows\system32\config\systemprofile\Documents>
```

다음 명령을 실행하여 IIS 시작 화면을 제거합니다.

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
다음 명령을 실행하여 기본 IIS 사이트를 새 정적 사이트로 바꿉니다.

```powershell
[IISApp]: PS C:\> "Hello World From a Hyper-V Container" > C:\inetpub\wwwroot\index.html
```

컨테이너 호스트의 IP 주소를 다시 검색하면 'Hello World' 응용 프로그램이 표시됩니다. 업데이트된 응용 프로그램을 확인하기 위해 모든 기존 브라우저 연결을 닫거나 브라우저 캐시의 선택을 취소해야 할 수 있습니다.

![](media/HWWINServer.png)

원격 컨테이너 세션을 종료합니다.

```powershell
exit
```




