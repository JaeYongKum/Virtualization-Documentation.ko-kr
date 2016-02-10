# 컨테이너 네트워킹

**이 예비 콘텐츠는 변경될 수 있습니다.**

Windows 컨테이너는 네트워킹에 관해서 가상 컴퓨터와 유사하게 작동합니다. 각 컨테이너에는 가상 네트워크 어댑터가 있고 이것은 가상 스위치에 연결되며 이를 통해 인바운드 및 아웃바운드 트래픽이 전달됩니다. 두 가지 유형의 네트워크 구성을 사용할 수 있습니다.

- **Network Address Translation 모드** – 각 컨테이너는 내부 가상 스위치에 연결되어 내부 IP 주소를 받습니다. NAT 구성은 이러한 내부 주소를 컨테이너 호스트의 외부 주소로 변환합니다.

- **투명 모드** – 각 컨테이너는 외부 가상 스위치에 연결되며 DHCP 서버로부터 IP 주소를 받습니다.

이 문서는 각각의 모드에 대한 이점과 구성을 자세히 설명합니다.

## NAT 네트워킹 모드

**Network Address Translation** – 이 구성은 WinNat 및 NAT 유형의 내부 네트워크 스위치를 포함합니다. 이 구성에서 컨테이너 호스트는 네트워크 상에 도달할 수 있는 ‘외부’ IP를 갖습니다. 모든 컨테이너에는 네트워크에서 액세스할 수 없는 ‘내부’ 주소가 할당됩니다. 이 구성에서 컨테이너에 액세스할 수 있도록 하기 위해, 호스트의 외부 포트가 컨테이너 포트의 내부 포트로 매핑됩니다. 이러한 매핑은 NAT 포트 매핑 테이블에 저장됩니다. 컨테이너는 호스트의 외부 포트 및 IP 주소를 통해 액세스가 가능하며, 호스트는 컨테이너의 포트 및 내부 IP 주소에 트래픽을 전달합니다. NAT의 이점은 컨테이너 호스트가 외부적으로 사용 가능한 IP 주소를 하나만 사용하면서, 수백 개의 컨테이너로 규모가 조정될 수 있다는 점입니다. 또한 NAT는 여러 컨테이너가 동일한 통신 포트를 필요로 하는 응용 프로그램을 호스트할 수 있도록 합니다.

### 호스트 구성

네트워크 주소 변환을 위해 컨테이너 호스트를 구성하려면 다음 단계를 수행합니다.

‘NAT’ 유형의 가상 스위치를 만들고 내부 서브넷으로 구성합니다. **New-VMSwitch** 명령에 대한 자세한 내용은 [New-VMSwitch 참조](https://technet.microsoft.com/en-us/library/hh848455.aspx)를 참조하세요.

```powershell
New-VMSwitch -Name "NAT" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```
네트워크 주소 변환 개체를 만듭니다. 이 개체는 NAT 주소 변환을 담당합니다. **New-NetNat** 명령에 대한 자세한 내용은 [New-NetNat 참조](https://technet.microsoft.com/en-us/library/dn283361.aspx)를 참조하세요.

```powershell
New-NetNat -Name NAT -InternalIPInterfaceAddressPrefix "172.16.0.0/12" 
```

### 컨테이너 구성

Windows 컨테이너를 만들 때 컨테이너에 대한 가상 스위치를 선택할 수 있습니다. 컨테이너가 NAT를 사용하도록 구성된 가상 스위치에 연결되면, 컨테이너는 변환된 주소를 받습니다.

이 예제는 NAT를 사용하는 가상 스위치에 연결된 컨테이너를 만듭니다.

```powershell
New-Container -Name DemoNAT -ContainerImageName WindowsServerCore -SwitchName "NAT"
```

컨테이너가 시작되면 컨테이너 내에서 IP 주소를 볼 수 있습니다.

```powershell
[DemoNAT]: PS C:\> ipconfig

Windows IP Configuration
Ethernet adapter vEthernet (Virtual Switch-527ED2FB-D56D-4852-AD7B-E83732A032F5-0):
   Connection-specific DNS Suffix  . : contoso.com
   Link-local IPv6 Address . . . . . : fe80::384e:a23d:3c4b:a227%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.240.0.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

Windows 컨테이너 시작 및 연결에 대한 자세한 내용은 [컨테이너 관리](./manage_containers.md)를 참조하세요.

### 포트 매핑

‘NAT 사용’ 컨테이너 내부의 응용 프로그램에 액세스하려면, 컨테이너와 컨테이너 호스트 사이에 포트 매핑을 만들어야 합니다. 매핑을 만들려면 컨테이너의 IP 주소, ‘내부’ 컨테이너 포트, ‘외부’ 호스트 포트가 필요합니다.

이 예제는 호스트의 포트 **80**과 IP 주소가 **172.16.0.2**인 컨테이너의 포트 **80** 간에 매핑을 만듭니다.

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
```

이 예제는 컨테이너 호스트의 포트 **82**와 IP 주소가 **172.16.0.3**인 컨테이너의 포트 **80** 간에 매핑을 만듭니다.

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.3 -InternalPort 80 -ExternalPort 82
```
> 각 외부 포트에 해당하는 방화벽 규칙이 필요합니다. 규칙은 `New-NetFirewallRule`을 사용하여 만들 수 있습니다. 자세한 내용은 [New-NetFirewallRule 참조](https://technet.microsoft.com/en-us/library/jj554908.aspx)를 참조하세요.

포트 매핑을 만든 후에는 컨테이너 호스트(물리적 또는 가상)의 IP 주소 및 노출된 외부 포트를 통해 컨테이너 응용 프로그램에 액세스할 수 있습니다. 예를 들어 아래 다이어그램은 컨테이너 호스트의 외부 포트 **82**를 대상으로 하는 요청을 포함하는 NAT 구성을 나타냅니다. 포트 매핑을 기반으로, 이 요청은 컨테이너 2에 호스트 중인 응용 프로그램을 반환합니다.

![](./media/nat1.png)

인터넷 브라우저의 요청에 대한 보기입니다.

![](./media/portmapping.png)

## 투명 네트워킹 모드

**투명 네트워킹** – 이 구성은 외부 네트워크 스위치를 포함합니다. 이 구성에서 각 컨테이너는 DHCP 서버로부터 IP 주소를 받으며, 이 IP 주소로 액세스할 수 있습니다. 여기서 이점은 포트 매핑 테이블이 유지 관리되지 않는다는 점입니다.

### 호스트 구성

컨테이너가 DHCP 서버로부터 IP 주소를 받을 수 있도록 컨테이너 시스템을 구성하려면, 물리적 또는 가상 네트워크 어댑터에 연결된 가상 스위치를 만듭니다.

다음 샘플은 Ethernet이라는 이름의 네트워크 어댑터를 사용하여 DHCP라는 이름의 가상 스위치를 만듭니다.

```powershell
New-VMSwitch -Name DHCP -NetAdapterName Ethernet
```

컨테이너 호스트 자체가 가상 컴퓨터인 경우에는 컨테이너 스위치와 함께 사용되는 네트워크 어댑터에 MacAddressSpoofing을 사용하도록 설정해야 합니다. 다음 예제에서는 `DemoVm`이라는 이름의 VM에서 이 작업을 완료합니다.

```powershell
Get-VMNetworkAdapter -VMName DemoVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
이제 외부 가상 스위치가 컨테이너에 연결될 수 있고 DHCP 서버로부터 IP 주소를 받을 수 있습니다. 이 구성에서 컨테이너 내부에 호스트되는 응용 프로그램은 컨테이너에 할당된 IP 주소로 액세스할 수 있습니다.

## Docker 구성

Docker 디먼을 시작할 때 네트워크 브리지를 선택할 수 있습니다. Windows에서 Docker를 실행하는 경우, 이것은 외부 또는 NAT 가상 스위치입니다. 다음 예제에서는 이름이 `Virtual Switch`인 가상 스위치를 지정하는 Docker 디먼을 시작합니다.

```powershell
Docker daemon –D –b “Virtual Switch” -H 0.0.0.0:2375
```

컨테이너 호스트를 배포했고, 스크립트를 사용하는 Docker가 Windows 컨테이너 빠른 시작에 제공된 경우에는 내부 가상 스위치가 NAT 유형으로 만들어지고 Docker 서비스가 생성되어 이 스위치를 사용하도록 사전 구성됩니다. Docker 서비스가 사용하는 가상 스위치를 변경하려면, Docker 서비스를 중지하고, 구성 파일을 수정하고, 서비스를 다시 시작해야 합니다.

서비스를 중지하려면 다음 PowerShell 명령을 사용합니다.

```powershell
Stop-Service docker
```

구성 파일은 `c:\programdata\docker\runDockerDaemon.cmd`에서 찾을 수 있습니다. 다음 줄을 편집하고 `Virtual Switch`를 Docker 서비스에 사용될 가상 스위치의 이름으로 바꿉니다.

```powershell
docker daemon -D -b “New Switch Name"
```
마지막으로 서비스를 시작합니다.

```powershell
Start-Service docker
```

## 네트워크 어댑터 관리

네트워크 구성(NAT 또는 투명)에 관계없이, 컨테이너 네트워크 어댑터 및 가상 스위치 연결을 관리하는 데 몇 가지 PowerShell 명령을 사용할 수 있습니다.

컨테이너 네트워크 어댑터 관리

- Add-ContainerNetworkAdapter – 네트워크 어댑터를 컨테이너에 추가합니다.
- Set-ContainerNetworkAdapter – 컨테이너 네트워크 어댑터를 수정합니다.
- Remove-ContainerNetworkAdapter - 컨테이너 네트워크 어댑터를 제거합니다.
- Get-ContainerNetworkAdapter - 컨테이너 네트워크 어댑터에 대한 데이터를 반환합니다.

컨테이너 네트워크 어댑터와 가상 스위치 간의 연결을 관리합니다.

- Connect-ContainerNetworkAdapter - 컨테이너를 가상 스위치에 연결합니다.
- Disconnect-ContainerNetworkAdapter – 컨테이너 연결을 가상 스위치에서 분리합니다.

각 명령에 대한 자세한 내용은 [컨테이너 PowerShell 참조](https://technet.microsoft.com/en-us/library/mt433069.aspx)를 참조하세요.




<!--HONumber=Feb16_HO1-->
