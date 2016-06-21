---
author: enderb-ms
redirect_url: https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples
---


# .NET 3.5 Server Core 컨테이너 이미지 만들기

이 가이드는 .NET 3.5 Framework를 포함하는 Windows Server Core 컨테이너를 만드는 방법을 자세히 설명합니다. 이 연습을 시작하기 전에 Windows Server 2016 .iso 파일 또는 Windows Server 2016 미디어에 액세스할 수 있어야 합니다.

## 미디어 준비

NET 3.5 사용 컨테이너 이미지를 만들기 전에 .NET 3.5 패키지를 컨테이너 내에서 사용할 수 있도록 스테이징해야 합니다. 이 예제에서는 `Microsoft-windows-netfx3-ondemand-package.cab` 파일이 Windows Server 2016 미디어에서 컨테이너 호스트로 복사 됩니다.

컨테이너 호스트에 `dotnet3.5\source`라는 디렉터리를 만듭니다.

```powershell
New-Item -ItemType Directory c:\dotnet3.5\source
```

`Microsoft-windows-netfx3-ondemand-package.cab` 파일을 이 디렉터리에 복사합니다. 이 파일은 Windows Server 2016 미디어의 sources\sxs 폴더에서 찾을 수 있습니다.

```powershell
$file = "d:\sources\sxs\Microsoft-windows-netfx3-ondemand-package.cab"
Copy-Item -Path $file -Destination c:\dotnet3.5\source
``` 
    
컨테이너 호스트가 Hyper-V 가상 컴퓨터에서 실행 중이고 빠른 시작 스크립트로 배포된 경우에는 다음을 실행할 수 있습니다. 컨테이너 호스트가 아니라 Hyper-V 호스트에서 실행한다는 점에 유의합니다. 

```powershell
$vm = "<Container Host VM Name>"
$iso = "$((Get-VMHost).VirtualHardDiskPath)".TrimEnd("\") + "\WindowsServerTP4.iso"
Mount-DiskImage -ImagePath $iso
$ISOImage = Get-DiskImage -ImagePath $iso | Get-Volume
$ISODrive = "$([string]$iSOImage.DriveLetter):"
Get-VM -Name $vm | Enable-VMIntegrationService -Name "Guest Service Interface"
Copy-VMFile -Name $vm -SourcePath "$iSODrive\sources\sxs\microsoft-windows-netfx3-ondemand-package.cab" -DestinationPath "c:\dotnet3.5\source\microsoft-windows-netfx3-ondemand-package.cab" -FileSource Host -CreateFullPath
Dismount-DiskImage -ImagePath $iso
```

이제 .NET 3.5 Framework를 포함하는 컨테이너 이미지를 만들 수 있습니다. 이 작업은 PowerShell 또는 Docker를 사용하여 완료할 수 있습니다. 두 가지 경우에 대한 예제는 다음과 같습니다.

## 이미지 만들기 - PowerShell

PowerShell을 사용하여 새로운 이미지를 만들려면, 컨테이너가 만들어지고 원하는 내용으로 수정된 후에 새로운 이미지로 캡처됩니다.

Windows Server Core 기본 이미지에서 새 컨테이너를 만듭니다.

```powershell
New-Container -Name dotnet35 -ContainerImageName windowsservercore -SwitchName "Virtual Switch"
```

새 컨테이너로 공유 폴더를 만듭니다. 이것은 새 컨테이너의 내부에서 액세스할 수 있는 .NET 3.5 cab 파일을 만드는 데 사용됩니다.  다음 명령을 실행하는 경우에는 컨테이너를 중지해야 합니다.

```powershell
Add-ContainerSharedFolder -ContainerName dotnet35 -SourcePath C:\dotnet3.5\source -DestinationPath c:\sxs
```

컨테이너를 시작하고 다음 명령을 실행하여 .NET 3.5를 설치합니다.

```powershell
Start-Container dotnet35
Invoke-Command -ContainerName dotnet35 -ScriptBlock {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs} -RunAsAdministrator
```

설치가 완료되면 컨테이너를 중지합니다.

```powershell
Stop-Container dotnet35
```

이 컨테이너로부터 이미지를 만들려면 컨테이너 호스트에서 다음을 실행합니다.

```powershell
New-ContainerImage -ContainerName dotnet35 -Name dotnet35 -Publisher Demo -Version 1.0
```

`Get-ContainerImages`를 실행하여 새 이미지를 확인합니다. 이 이미지를 사용하여 .NET 3.5 Framework가 사전 설치된 컨테이너를 실행할 수 있습니다.

```powershell
Get-ContainerImages
```

## 이미지 만들기 - Docker
 
Docker를 사용하여 새 이미지를 만들려면, 새 이미지를 만드는 방법에 대한 지침과 함께 dockerfile이 만들어집니다. 그 후 dockerfile이 실행되고 새 컨테이너 이미지가 만들어집니다. 다음 명령은 컨테이너 호스트 VM에서 실행됩니다.

dockerfile 파일을 만들고 메모장에서 엽니다.

```powershell
New-Item C:\dotnet3.5\dockerfile -Force
Notepad C:\dotnet3.5\dockerfile
```

이 텍스트를 dockerfile에 복사하고 저장합니다.

```powershell
FROM windowsservercore
ADD source /sxs
RUN powershell -Command "& {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs}"
```

`docker build`를 실행하면 dockerfile이 사용되고 새 컨테이너 이미지가 만들어집니다.

```powershell
Docker build -t dotnet35 C:\dotnet3.5\
```

`docker images`를 실행하여 새 이미지를 확인합니다. 이 이미지를 사용하여 .NET 3.5 Framework가 사전 설치된 컨테이너를 실행할 수 있습니다.

```powershell
docker images
```


<!--HONumber=May16_HO3-->


