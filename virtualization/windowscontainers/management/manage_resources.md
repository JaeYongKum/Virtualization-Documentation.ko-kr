# 컨테이너 리소스 관리

**이 예비 콘텐츠는 변경될 수 있습니다.**

Windows 컨테이너는 컨테이너가 사용할 수 있는 CPU, 디스크 IO, 네트워크 및 메모리 리소스의 양을 관리하는 기능을 포함합니다. 컨테이너 리소스 사용 제한을 통해 호스트 리소스를 효율적으로 사용할 수 있으며 과사용을 방지할 수 있습니다. 이 문서에서는 PowerShell와 Docker를 사용한 컨테이너 리소스 관리를 자세히 설명합니다.

## PowerShell을 사용하여 리소스 관리

### 메모리

컨테이너가 `New-Container` 명령의 `-MaximumMemoryBytes` 매개 변수를 사용하여 만들어진 경우 컨테이너 메모리 제한을 설정할 수 있습니다. 이 예에서는 256mb로 최대 메모리를 설정합니다.

```powershell
PS C:\> New-Container –Name TestContainer –MaximumMemoryBytes 256MB -ContainerimageName WindowsServerCore
```
`Set-ContainerMemory` cmdlet을 사용하여 기존 컨테이너의 메모리 제한을 설정할 수도 있습니다.

```powershell
PS C:\> Set-ContainerMemory -ContainerName TestContainer -MaximumBytes 256mb
```

### 네트워크 대역폭

기존 컨테이너에 네트워크 대역폭 제한을 설정할 수 있습니다. 이렇게 하려면 컨테이너가 `Get-ContainerNetworkAdapter` 명령을 사용하여 네트워크 어댑터를 갖도록 해야 합니다. 네트워크 어댑터가 없는 경우 `Add-ContainerNetworkAdapter` 명령을 사용하여 하나를 만듭니다. 마지막으로 `Set-ContainerNetworkAdapter` 명령을 사용하여 컨테이너의 최대 송신 네트워크 대역폭을 제한합니다.

아래 샘플은 최대 대역폭을 100Mbps로 설정합니다.

```powershell
PS C:\> Set-ContainerNetworkAdapter –ContainerName TestContainer –MaximumBandwidth 100000000
```

### CPU

CPU의 최대 백분율을 설정하거나 컨테이너에 대한 상대 가중치를 설정하여 컨테이너가 사용할 수는 계산의 양을 제한할 수 있습니다. 이러한 두 개의 CPU 관리 체계 혼합은 차단되지 않지만 권장되지 않습니다. 기본적으로 모든 컨테이너는 프로세서(최대 100%)를 사용하고 100의 상대 가중치를 적용합니다.

아래는 컨테이너의 상대 가중치를 1000으로 설정합니다. 컨테이너의 기본 가중치는 100이므로 기본값으로 설정된 컨테이너 우선 순위에 10배를 가집니다. 최대값은 10000입니다.

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 –RelativeWeight 10000
```

컨테이너가 CPU 시간의 백분율을 기준으로 사용할 수는 CPU의 양에 하드 한도를 설정할 수도 있습니다. 기본적으로 컨테이너는 CPU의 100%를 사용할 수 있습니다. 다음은 컨테이너가 사용할 수 있는 CPU의 최대 백분율을 30%로 설정합니다. 최대 플래그를 사용하면 RelativeWeight가 자동으로 100으로 설정됩니다.

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -Maximum 30
```

### 저장소 IO

대역폭(초당 바이트) 또는 8k 정규화된 IOPS를 기준으로 컨테이너가 사용할 수는 IO의 양을 제한할 수 있습니다. 이 두 매개 변수는 함께 설정할 수 있습니다. 첫 번째 한도에 도달할 때 제한이 발생합니다.

```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumBandwidth 1000000
```
```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumIOPS 32
```

## Docker를 사용하여 리소스 관리

Docker를 통해 컨테이너 리소스의 하위 집합을 관리하는 기능을 제공합니다. 특히 사용자가 컨테이너에서 CPU가 공유되는 방법을 지정할 수 있도록 허용합니다.

### CPU

컨테이너 간의 CPU 공유는 --cpu 공유 플래그를 통해 런타임 시 관리할 수 있습니다. 기본적으로 모든 컨테이너는 CPU 시간과 동일한 비율로 이용할 수 있습니다. 컨테이너가 사용하는 CPU의 상대 공유를 변경하려면 1-10000의 값으로 --cpu 공유 플래그를 실행합니다. 기본적으로 모든 컨테이너는 5000의 가중치를 받습니다. CPU 공유 제약 조건에 대한 자세한 내용은 [Docker Run 참조](https://docs.docker.com/engine/reference/run/#cpu-share-constraint)를 참조하세요.

```powershell 
C:\> docker run –it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## 알려진 문제

- CPU 및 IO 리소스 컨트롤은 현재 Hyper-V 컨테이너로 지원되지 않습니다.
- IO 리소스 컨트롤은 현재 컨테이너 공유 폴더로 지원되지 않습니다.

## 비디오 연습

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-4-Resource-Management/player#ccLang=ko" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>





<!--HONumber=Feb16_HO1-->
