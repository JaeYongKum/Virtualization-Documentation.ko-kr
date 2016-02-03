# 컨테이너 이미지

**이 예비 콘텐츠는 변경될 수 있습니다.**

컨테이너 이미지는 컨테이너를 배포하는 데 사용됩니다. 이러한 이미지는 운영 체제, 응용 프로그램 및 모든 응용 프로그램 종속성을 포함할 수 있습니다. 예를 들어 Nano Server, IIS 및 IIS를 실행 중인 응용 프로그램으로 사전 구성한 컨테이너 이미지를 개발할 수 있습니다. 그런 다음 이 컨테이너 이미지를 나중에 사용할 수 있게 컨테이너 레지스트리에 저장하고, 아무 Windows 컨테이너(온-프레미스, 클라우드 또는 컨테이너 서비스)에나 배포하며 새 컨테이너 이미지를 위한 기반으로 사용할 수도 있습니다.

컨테이너 이미지에는 다음과 같은 두 가지 유형이 있습니다.

- 기본 OS 이미지 – Microsoft에서 제공하고 핵심 OS 구성 요소를 포함합니다.
- 컨테이너 이미지 – 기본 OS 이미지에서 생성된 컨테이너 이미지입니다.

## PowerShell

### 이미지 나열

`get-containerImage`를 실행하면 컨테이너 호스트의 이미지 목록이 반환됩니다. 컨테이너 이미지 형식은 `IsOSImage` 속성으로 구분합니다.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### 기본 OS 이미지 설치

컨테이너 OS 이미지는 ContainerProvider PowerShell 모듈을 사용하여 확인 및 설치할 수 있습니다. 이 모듈을 사용하기 전에 먼저 모듈을 설치해야 합니다. 다음 명령을 사용하여 모듈을 설치할 수 있습니다.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

PowerShell OneGet 패키지 관리자로부터 이미지 목록을 반환합니다.
```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Nano Server 기본 OS 이미지를 다운로드하여 설치하려면 다음을 실행합니다. `–version` 매개 변수는 선택 사항입니다. 기본 OS 이미지 버전을 지정하지 않으면 최신 버전이 설치됩니다.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

마찬가지로 이 명령은 Windows Server Core 기본 OS 이미지를 다운로드하여 설치합니다. `–version` 매개 변수는 선택 사항입니다. 기본 OS 이미지 버전을 지정하지 않으면 최신 버전이 설치됩니다.

>**문제** Save-ContainerImage 및 Install-ContainerImage cmdlet이 PowerShell 원격 세션의 WindowsServerCore 컨테이너 이미지에서 작동하지 않습니다. **해결 방법:** 원격 데스크톱을 사용하여 컴퓨터에 로그온하고 Save-ContainerImage cmdlet을 직접 저장합니다.

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

`Get-ContainerImage` 명령을 사용하여 이미지가 설치되었는지 확인합니다.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```
컨테이너 이미지 관리에 대한 자세한 내용은 [Windows 컨테이너 이미지](../management/manage_images.md)를 참조하세요.

### 새 이미지 만들기

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### 이미지 제거

중지 상태에서라도 컨테이너에 해당 이미지에 대한 종속성이 있으면 컨테이너 이미지를 제거할 수 없습니다.

PowerShell로 단일 이미지를 제거합니다.

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### 이미지 종속성

새 이미지를 만들면 생성의 근거가 된 이미지에 종속됩니다. 이 종속성은 `get-containerimage` 명령을 사용하여 확인할 수 있습니다. 부모 이미지가 목록에 없으면 해당 이미지가 기본 OS 이미지임을 나타냅니다.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

## Docker

### 이미지 나열

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### 새 이미지 만들기

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### 이미지 제거

중지 상태에서라도 컨테이너에 해당 이미지에 대한 종속성이 있으면 컨테이너 이미지를 제거할 수 없습니다.

Docke가 있는 이미지를 제거할 때는 이미지 이름 또는 ID로 이미지를 참조할 수 있습니다.

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Docker 허브

Docker 허브 레지스트리는 컨테이너 호스트에 다운로드할 수 있는 미리 빌드된 이미지를 포함합니다. 이 이미지를 다운로드한 후에는 Windows 컨테이너 응용 프로그램을 위한 기반으로 사용할 수 있습니다.

Docker 허브에서 사용 가능한 이미지 목록을 보려면 `docker search` 명령을 사용합니다. 참고 – Docker 허브에서 Windows Server Core에 종속되는 이미지를 가져오려면 먼저 Windows Serve Core 기본 OS 이미지를 설치해야 합니다.

```powershell
C:\> docker search *

NAME                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/aspnet        ASP.NET 5 framework installed in a Windows...   1         [OK]       [OK]
microsoft/django        Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35      .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/golang        Go Programming Language installed in a Win...   1                    [OK]
microsoft/httpd         Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis           Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/mongodb       MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/mysql         MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/nginx         Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/node          Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/php           PHP running on Internet Information Servic...   1                    [OK]
microsoft/python        Python installed in a Windows Server Core ...   1                    [OK]
microsoft/rails         Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/redis         Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/ruby          Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sqlite        SQLite installed in a Windows Server Core ...   1                    [OK]
```

Docker 허브에서 이미지를 다운로드하려면 `docker pull`을 사용합니다.

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

이제 `docker images`를 실행하면 이미지가 표시됩니다.

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

### 이미지 종속성

Docker를 통한 이미지 종속성을 확인하려면 `docker history` 명령을 사용하면 됩니다.

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```



<!--HONumber=Jan16_HO1-->
