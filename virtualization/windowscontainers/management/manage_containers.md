



# Windows Server 컨테이너 관리

**이 예비 콘텐츠는 변경될 수 있습니다.**

컨테이너 수명 주기에는 컨테이너 시작, 중지 및 제거 등과 같은 작업이 포함됩니다. 이러한 작업을 수행할 때는 컨테이너 이미지 목록을 검색하고, 컨테이너 네트워킹을 관리하며 컨테이너 리소스를 제한해야 할 수도 있습니다. 이 문서에서는 PowerShell을 사용한 기본 컨테이너 관리 작업을 자세히 설명합니다.

Docker를 통한 Windows 컨테이너 관리에 대한 설명은 [컨테이너 작업](https://docs.docker.com/userguide/usingdocker/)을 참조하세요.

## PowerShell

### 컨테이너 만들기

새 컨테이너를 만들 때는 컨테이너의 기반 역할을 할 컨테이너 이미지의 이름이 필요합니다. `Get-ContainerImage` 명령으로 찾을 수 있습니다.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version         IsOSImage
----              ---------    -------         ---------
NanoServer        CN=Microsoft 10.0.10584.1000 True
WindowsServerCore CN=Microsoft 10.0.10584.1000 True
```

`New-Container` 명령을 사용하여 새 컨테이너를 만듭니다. `-ContainerComputerName` 매개 변수를 사용하여 컨테이너에 NetBIOS 이름을 지정할 수도 있습니다.

```powershell
PS C:\> New-Container -ContainerImageName WindowsServerCore -Name demo -ContainerComputerName demo

Name State Uptime   ParentImageName
---- ----- ------   ---------------
demo  Off   00:00:00 WindowsServerCore
```

컨테이너를 만든 후에는 네트워크 어댑터를 컨테이너에 추가합니다.

```powershell
PS C:\> Add-ContainerNetworkAdapter -ContainerName demo
```

컨테이너 네트워크 어댑터를 가상 스위치에 연결하려면 스위치 이름이 필요합니다. `Get-VMSwitch`를 사용하여 가상 스위치 목록을 반환합니다.

```powershell
PS C:\> Get-VMSwitch

Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
DHCP External   Microsoft Hyper-V Network Adapter
NAT  NAT
```

`Connect-ContainerNetworkAdapter`를 사용하여 네트워크 어댑터를 가상 스위치에 연결합니다. **참고** - SwitchName 매개 변수를 사용하여 컨테이너를 만든 경우에도 이렇게 할 수 있습니다.

```powershell
PS C:\> Connect-ContainerNetworkAdapter -ContainerName demo -SwitchName NAT
```

### 컨테이너 시작

컨테이너를 시작하기 위해 해당 컨테이너를 나타내는 PowerShell 개체를 열거합니다. 그렇게 하려면 `Get-Container` 출력을 PowerShell 변수에 배치하면 됩니다.

```powershell
PS C:\> $container = Get-Container -Name demo
```

그런 다음 이 데이터를 `Start-Container` 명령과 함께 사용하여 컨테이너를 시작합니다.

```powershell
PS C:\> Start-Container $container
```

다음 스크립트는 호스트의 모든 컨테이너를 시작합니다.

```powershell
PS C:\> Get-Container | Start-Container
```

### 컨테이너에 연결

PowerShell direct를 직접 사용하여 컨테이너에 연결할 수 있습니다. 소프트웨어 설치, 프로세스 시작 또는 컨테이너 문제 해결 등과 같은 작업을 수동으로 수행해야 할 경우 이렇게 하는 것이 유용할 수 있습니다. PowerShell direct를 사용하는 중에는 네트워크 구성에 관계없이 컨테이너에서 PowerShell 세션을 만들 수 있습니다. PowerShell Direct에 대한 자세한 내용은 [PowerShell Direct 블로그](http://blogs.technet.com/b/virtualization/archive/2015/05/14/powershell-direct-running-powershell-inside-a-virtual-machine-from-the-hyper-v-host.aspx)를 참조하세요.

컨테이너와의 대화형 세션을 만들려면 `Enter-PSSession` 명령을 사용합니다.

 ```powershell
PS C:\> Enter-PSSession -ContainerName demo -RunAsAdministrator
 ```

원격 PowerShell 세션을 만든 후에는 셸 프롬프트가 컨테이너 이름에 맞게 변경됩니다.

```powershell
[demo]: PS C:\>
```

영구 PowerShell 세션을 만들지 않고도 컨테이너에 대해 명령을 실행할 수 있습니다. 이를 위해 `Invoke-Command`를 사용합니다.

다음 샘플에서는 컨테이너 안에 이름이 'Application'인 폴더를 만듭니다.

```powershell

PS C:\> Invoke-Command -ContainerName demo -ScriptBlock {New-Item -ItemType Directory -Path c:\application }

Directory: C:\
Mode                LastWriteTime         Length Name                                                 PSComputerName
----                -------------         ------ ----                                                 --------------
d-----       10/28/2015   3:31 PM                application                                          demo
```

### 컨테이너 중지

컨테이너를 중지하려면 해당 컨테이너를 나타내는 PowerShell 개체가 필요합니다. 그렇게 하려면 `Get-Container` 출력을 PowerShell 변수에 배치하면 됩니다.

```powershell
PS C:\> $container = Get-Container -Name demo
```

그런 다음 이 데이터를 `Stop-Container` 명령과 함께 사용하여 컨테이너를 중지합니다.

```powershell
PS C:\> Stop-Container $container
```

다음 스크립트는 호스트의 모든 컨테이너를 중지합니다.

```powershell
PS C:\> Get-Container | Stop-Container
```

### 컨테이너 제거

컨테이너가 더 이상 필요하지 않으면 제거할 수 있습니다. 컨테이너를 제거하려면 중지 상태여야 하며 해당 컨테이너를 나타내는 PowerShell 개체를 만들어야 합니다.

```powershell
PS C:\> $container = Get-Container -Name demo
```

컨테이너를 제거하려면 `Remove-Container` 명령을 사용합니다.

```powershell
PS C:\> Remove-Container $container -Force
```

다음은 호스트의 모든 컨테이너를 제거합니다.

```powershell
PS C:\> Get-Container | Remove-Container -Force
```

## Docker

### 컨테이너 만들기

`docker run`을 사용하여 Docker가 있는 컨테이너를 만듭니다.

```powershell
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Docker run 명령에 대한 자세한 내용은 [Docker run 참조}( https://docs.docker.com/engine/reference/run/)에서 확인하세요.

### 컨테이너 중지

`docker stop` 명령을 사용하여 Docker가 있는 컨테이너를 중지합니다.

```powershell
PS C:\> docker stop tender_panini

tender_panini
```

이 예제에서는 Docker를 통해 실행 중인 모든 컨테이너를 중지합니다.

```powershell
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### 컨테이너 제거

Docker가 있는 컨테이너를 제거하려면 `docker rm` 명령을 사용합니다.

```powershell
PS C:\> docker rm prickly_pike

prickly_pike
```

Docker가 있는 모든 컨테이너를 제거합니다.

```powershell
PS C:\> docker rm $(docker ps -a -q)

dc3e282c064d
2230b0433370
```

Docker rm 명령에 대한 자세한 내용은 [Docker rm 참조](https://docs.docker.com/engine/reference/commandline/rm/)를 참조하세요.






<!--HONumber=Feb16_HO4-->


