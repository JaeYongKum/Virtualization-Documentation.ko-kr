# 컨테이너 호스트 배포 - Nano Server

**이 예비 콘텐츠는 변경될 수 있습니다.**

Windows 컨테이너 호스트 배포 단계는 운영 체제와 호스트 시스템 유형(물리적 또는 가상)에 따라 다릅니다. 이 문서의 단계는 물리적 또는 가상 시스템에서 Nano Server에 Windows 컨테이너 호스트를 배포하는 데 사용됩니다. Windows 컨테이너 호스트를 Windows Server에 설치하려면 [컨테이너 호스트 배포 - Windows Server](./deployment.md)를 참조하세요.

시스템 요구 사항에 대한 자세한 내용은 [Windows 컨테이너 호스트 시스템 요구 사항](./system_requirements.md)을 참조하세요.

PowerShell 스크립트를 사용하여 Windows 컨테이너 호스트의 배포를 자동화할 수 있습니다.
- [새 Hyper-V 가상 컴퓨터에 컨테이너 호스트 배포](../quick_start/container_setup.md)
- [기존 시스템에 컨테이너 호스트 배포](../quick_start/inplace_setup.md)


# Nano Server 호스트

이 표에 나열된 단계를 사용하여 컨테이너 호스트를 Nano Server에 배포할 수 있습니다. Windows Server와 Hyper-V 컨테이너 모두에 필요한 구성이 포함되어 있습니다.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>배포 작업</strong></td>
<td width="70%"><strong>세부 정보</strong></td>
</tr>
<tr>
<td>[컨테이너에 사용할 Nano Server 준비](#nano)</td>
<td>컨테이너 및 Hyper-V 기능을 통해 Nano Server를 준비합니다.</td>
</tr>
<tr>
<td>[가상 스위치 만들기](#vswitch)</td>
<td>컨테이너는 네트워크 연결을 위해 가상 스위치에 연결합니다.</td>
</tr>
<tr>
<td>[NAT 구성](#nat)</td>
<td>가상 스위치가 NAT(Network Address Translation)로 구성된 경우 NAT 자체를 구성해야 합니다.</td>
</tr>
<tr>
<td>[컨테이너 OS 이미지 설치](#img)</td>
<td>OS 이미지는 컨테이너 배포를 위한 기초를 제공합니다.</td>
</tr>
<tr>
<td>[Docker 설치](#docker)</td>
<td>선택 사항이지만 Docker로 Windows 컨테이너를 만들어 관리하려면 필요합니다. </td>
</tr>
</table>

Hyper-V 컨테이너를 사용할 경우 이 단계를 수행해야 합니다. * 표시가 있는 단계는 컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우에만 필요합니다.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>배포 작업</strong></td>
<td width="70%"><strong>세부 정보</strong></td>
</tr>
<tr>
<td>[Hyper-V 역할 사용](#hypv) </td>
<td>Hyper-V는 Hyper-V 컨테이너를 배포할 때만 필요합니다.</td>
</tr>
<tr>
<td>[중첩된 가상화 사용 *](#nest)</td>
<td>컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우 중첩된 가상화를 사용해야 합니다.</td>
</tr>
<tr>
<td>[가상 프로세서 구성 *](#proc)</td>
<td>컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우 둘 이상의 가상 프로세서를 구성해야 합니다.</td>
</tr>
<tr>
<td>[동적 메모리 사용 안 함 *](#dyn)</td>
<td>컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우 동적 메모리를 사용하지 않도록 설정해야 합니다.</td>
</tr>
<tr>
<td>[MAC 주소 스푸핑 구성 *](#mac)</td>
<td>컨테이너 호스트가 가상화된 경우 MAC 스푸핑을 사용해야 합니다.</td>
</tr>
</table>

## 배포 단계

### <a name=nano></a> Nano Server 준비

Nano 서버 배포에는 준비된 가상 하드 드라이브 만들기가 필요합니다. 여기에는 Nano Server 운영 체제와 추가적인 기능 패키지가 포함됩니다. 이 가이드에서는 Windows 컨테이너에 사용할 수 있는 Nano Server 가상 하드 드라이브를 준비하는 방법을 신속히 설명하겠습니다. Nano Server에 대한 자세한 내용과 다양한 Nano Server 배포 옵션을 확인하려면 [Nano Server 설명서](https://technet.microsoft.com/en-us/library/mt126167.aspx)를 참조하세요.

이름이 `nano`인 폴더를 만듭니다.

```powershell
PS C:\> New-Item -ItemType Directory c:\nano
```

Windows Server 미디어의 Nano Server 폴더에서 `NanoServerImageGenerator.psm1` 및 `Convert-WindowsImage.ps1` 파일을 찾습니다. 이 두 파일을 `c:\nano`에 복사합니다.

```powershell
#Set path to Windows Server 2016 Media
PS C:\> $WindowsMedia = "C:\Users\Administrator\Desktop\TP4 Release Media"

PS C:\> Copy-Item $WindowsMedia\NanoServer\Convert-WindowsImage.ps1 c:\nano

PS C:\> Copy-Item $WindowsMedia\NanoServer\NanoServerImageGenerator.psm1 c:\nano
```
다음을 실행하여 Nano Server 가상 하드 드라이브를 만듭니다. `–Containers` 매개 변수는 컨테이너 패키지가 설치되었음을 나타내고, `–Compute` 매개 변수는 Hyper-V 패키지를 관리합니다. Hyper-V는 Hyper-V 컨테이너를 사용하는 경우에만 필요합니다.

```powershell
PS C:\> Import-Module C:\nano\NanoServerImageGenerator.psm1

PS C:\> New-NanoServerImage -MediaPath $WindowsMedia -BasePath c:\nano -TargetPath C:\nano\NanoContainer.vhdx -MaxSize 10GB -GuestDrivers -ReverseForwarders -Compute -Containers
```
완료되면 `NanoContainer.vhdx` 파일에서 가상 컴퓨터를 만듭니다. 이 가상 컴퓨터는 Nano Server OS와 선택적 패키지를 실행합니다.

### <a name=vswitch></a>가상 스위치 만들기

각 컨테이너는 네트워크를 통해 통신을 하기 위해 가상 스위치에 연결되어야 합니다. 가상 스위치는 `New-VMSwitch` 명령으로 만들어집니다. 컨테이너는 형식이 `외부` 또는 `NAT`인 가상 스위치를 지원합니다. Windows 컨테이너 네트워킹에 대한 자세한 내용은 [컨테이너 네트워킹](../management/container_networking.md)을 참조하세요.

이 예제에서는 이름이 “Virtual Switch”이고 NAT 유형이며 NAT 서브넷이 172.16.0.0/12인 가상 스위치를 만듭니다.

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```

### <a name=nat></a>NAT 구성

스위치 형식이 NAT인 경우 가상 스위치를 만드는 것 외에도 NAT 개체를 만들어야 합니다. 이 작업은 `New-NetNat` 명령을 통해 완료됩니다. 이 예에서는 이름이 `ContainerNat`이며, 컨테이너 스위치에 할당된 NAT 서브넷과 일치하는 주소 접두사를 갖는 NAT 개체를 만듭니다.

```powershell
PS C:\> New-NetNat -Name ContainerNat -InternalIPInterfaceAddressPrefix "172.16.0.0/12"

Name                             : ContainerNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 172.16.0.0/12
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

### <a name=img></a>OS 이미지 설치

OS 이미지는 모든 Windows Server나 Hyper-V 컨테이너의 기반으로 사용합니다. 이미지는 컨테이너를 배포하는 데 사용할 수 있습니다. 그런 다음 수정하고 새 컨테이너 이미지에 캡처할 수 있습니다. Windows Server Core와 Nano Server모두를 기본 운영 체제로 하여 OS 이미지가 만들어졌습니다.

컨테이너 OS 이미지는 ContainerProvider PowerShell 모듈을 사용하여 확인 및 설치할 수 있습니다. 이 모듈을 사용하기 전에 먼저 모듈을 설치해야 합니다. 다음 명령을 사용하여 모듈을 설치할 수 있습니다.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

`Find-ContainerImage`를 사용하여 PowerShell OneGet 패키지 관리자로부터 이미지 목록을 반환합니다.

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```
**참고** - 이때 Nano Server OS 이미지는 Nano Server 컨테이너 호스트와 호환됩니다. Nano Server 기본 OS 이미지를 다운로드하여 설치하려면 다음을 실행합니다.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

`Get-ContainerImage` 명령을 사용하여 이미지가 설치되었는지 확인합니다.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
```
컨테이너 이미지 관리에 대한 자세한 내용은 [Windows 컨테이너 이미지](../management/manage_images.md)를 참조하세요.


### <a name=docker></a>Docker 설치

Docker 데몬 및 명령줄 인터페이스는 Windows와 함께 제공되지 않으며 Windows 컨테이너 기능과 함께 설치되지 않습니다. Docker는 Windows 컨테이너를 사용 하기 위한 요구 사항이 아닙니다. Docker를 설치하려면 [Docker 및 Windows](./docker_windows.md) 문서의 지침을 따릅니다.


## Hyper-V 컨테이너 호스트

### <a name=hypv></a>Hyper-V 역할 사용

Nano Server에서 Nano Server 이미지를 만들 때 이를 완료할 수 있습니다. 이러한 지침은 [컨테이너에 사용할 Nano Server 준비](#nano)를 참조하세요.

### <a name=nest></a>중첩된 가상화 구성

컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터에서 실행되며 Hyper-V 호스트 컨테이너를 호스팅할 경우, 중첩된 가상화를 사용해야 합니다 다음 PowerShell 명령을 사용하여 이를 수행할 수 있습니다.

**참고** - 이 명령을 실행할 때는 가상 컴퓨터가 꺼져 있어야 합니다.

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>가상 프로세서 구성

컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터에서 실행되며 Hyper-V 호스트 컨테이너를 호스팅할 경우, 가상 컴퓨터에 둘 이상의 프로세서가 필요합니다. 가상 컴퓨터의 설정을 통해 또는 다음 명령을 사용하여 이를 구성할 수 있습니다.

**참고** - 이 명령을 실행할 때는 가상 컴퓨터가 꺼져 있어야 합니다.

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=dyn></a>동적 메모리 사용 안 함

컨테이너 호스트 자체가 Hyper-V 가상 컴퓨터인 경우 컨테이너 호스트 가상 컴퓨터에서 동적 메모리를 사용하지 않도록 설정해야 합니다. 가상 컴퓨터의 설정을 통해 또는 다음 명령을 사용하여 이를 구성할 수 있습니다.

**참고** - 이 명령을 실행할 때는 가상 컴퓨터가 꺼져 있어야 합니다.

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>MAC 주소 스푸핑 구성

마지막으로 컨테이너 호스트가 Hyper-V 가상 컴퓨터 안에서 실행 중인 경우 MAC 스푸핑을 사용하도록 설정해야 합니다. 이렇게 하면 각 컨테이너가 IP 주소를 받을 수 있습니다. MAC 주소 스푸핑을 사용하려면 Hyper-V 호스트에서 다음 명령을 실행합니다. VMName 속성은 컨테이너 호스트의 이름이 됩니다.

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```




<!--HONumber=Feb16_HO1-->
