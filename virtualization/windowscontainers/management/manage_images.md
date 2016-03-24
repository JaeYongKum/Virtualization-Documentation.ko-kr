---
author: neilpeterson
---

# 컨테이너 이미지

**이 예비 콘텐츠는 변경될 수 있습니다.**

컨테이너 이미지는 컨테이너를 배포하는 데 사용됩니다. 이러한 이미지는 운영 체제, 응용 프로그램 및 모든 응용 프로그램 종속성을 포함할 수 있습니다. 예를 들어 Nano Server, IIS 및 IIS를 실행 중인 응용 프로그램으로 사전 구성한 컨테이너 이미지를 개발할 수 있습니다. 그런 다음 이 컨테이너 이미지를 나중에 사용할 수 있게 컨테이너 레지스트리에 저장하고, 아무 Windows 컨테이너(온-프레미스, 클라우드 또는 컨테이너 서비스)에나 배포하며 새 컨테이너 이미지를 위한 기반으로 사용할 수도 있습니다.

컨테이너 이미지에는 다음과 같은 두 가지 유형이 있습니다.

- **기본 OS 이미지** - Microsoft에서 제공하고 핵심 OS 구성 요소를 포함합니다.
- **컨테이너 이미지** - 기본 OS 이미지에서 파생된 사용자 지정 컨테이너 이미지입니다.

## 기본 OS 이미지

### 이미지 설치

ContainerProvider PowerShell 모듈을 사용하여 컨테이너 OS 이미지를 찾아 PowerShell 관리와 Docker 관리 둘 다를 위해 설치할 수 있습니다. 이 모듈을 사용하기 전에 먼저 모듈을 설치해야 합니다. 다음 명령을 사용하여 모듈을 설치할 수 있습니다.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

설치한 후 `Find-ContainerImage`를 사용하여 기본 OS 이미지 목록을 반환할 수 있습니다.

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Nano Server 기본 OS 이미지를 다운로드하여 설치하려면 다음을 실행합니다. `-version` 매개 변수는 선택 사항입니다. 기본 OS 이미지 버전을 지정하지 않으면 최신 버전이 설치됩니다.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

마찬가지로 이 명령은 Windows Server Core 기본 OS 이미지를 다운로드하여 설치합니다. `-version` 매개 변수는 선택 사항입니다. 기본 OS 이미지 버전을 지정하지 않으면 최신 버전이 설치됩니다.

> **문제** Save-ContainerImage 및 Install-ContainerImage cmdlet이 PowerShell 원격 세션의 WindowsServerCore 컨테이너 이미지에서 작동하지 않을 수 있습니다. **해결 방법:** 원격 데스크톱을 사용하여 컴퓨터에 로그온하고 Save-ContainerImage cmdlet을 직접 저장합니다.

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

> **Install-ContainerImage**는 PowerShell 또는 Docker에서 관리하는 컨테이너에 사용할 기본 OS 이미지를 설치합니다. 기본 OS 이미지가 다운로드되었지만, `docker images`를 실행할 때 표시되지 않으면 서비스 제어판 애플릿을 사용하거나, 'sc docker stop' 명령을 사용한 다음 'sc docker start' 명령을 사용하여 Docker 서비스를 다시 시작합니다.

### 오프라인 설치

인터넷 연결 없이 기본 OS 이미지를 설치할 수도 있습니다. 이렇게 하려면 인터넷에 연결된 컴퓨터에 이미지를 다운로드한 다음 이미지를 대상 시스템에 복사하고 `Install-ContainerOSImages` 명령을 사용하여 이미지를 가져옵니다.

기본 OS 이미지를 다운로드하기 전에 다음 명령을 실행하여 컨테이너 이미지 공급자로 시스템을 준비합니다.

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

이미지를 다운로드하려면 `Save-ContainerImage` 명령을 사용합니다.

```powershell
PS C:\> Save-ContainerImage -Name NanoServer -Destination c:\container-image\NanoServer.wim
```

이제 다운로드한 컨테이너 이미지를 다른 컨테이너 호스트에 복사하고 `Install-ContainerOSImage` 명령을 사용하여 설치할 수 있습니다.

```powershell
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### 이미지 태그 지정

컨테이너 이미지를 이름으로 참조할 경우 Docker 엔진은 이미지의 최신 버전을 검색합니다. 최신 버전을 확인할 수 없는 경우 다음과 같은 오류가 발생합니다.

```powershell
PS C:\> docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Windows Server Core 또는 Nano Server 기본 OS 이미지를 설치한 후 이러한 이미지에 'latest' 버전이라는 태그가 지정되어야 합니다. 이렇게 하려면 `docker tag` 명령을 사용합니다.

`docker tag`에 대한 자세한 내용은 [docker.com의 Tag, push, and pull your images(이미지 태그 지정, 푸시 및 끌어오기)](https://docs.docker.com/mac/step_six/)를 참조하세요.

```powershell
PS C:\> docker tag <image id> windowsservercore:latest
```

태그가 지정된 경우 `docker images` 출력에 같은 이미지의 두 가지 버전이 표시됩니다. 하나는 이미지 버전 태그가 있는 것이고, 다른 하나는 'latest' 태그가 있는 것입니다. 이제 이름으로 이미지를 참조할 수 있습니다.

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14289.1000     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14289.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### OS 이미지 제거

`Uninstall-ContainerOSImage` 명령을 사용하여 기본 OS 이미지를 제거할 수 있습니다. 다음 예제에서는 NanoServer 기본 OS 이미지를 제거합니다.

```powershell
Get-ContainerImage -Name NanoServer | Uninstall-ContainerOSImage
```

## 컨테이너 이미지 PowerShell

### 이미지 나열

`Get-ContainerImage`를 실행하면 컨테이너 호스트의 이미지 목록이 반환됩니다. 컨테이너 이미지 형식은 `IsOSImage` 속성으로 구분합니다.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### 새 이미지 만들기

모든 기존 컨테이너에서 새 컨테이너 이미지를 만들 수 있습니다. 이렇게 하려면 `New-ContainerImage` 명령을 사용합니다.

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

새 이미지를 만들면 생성의 근거가 된 이미지에 종속됩니다. 이 종속성은 `Get-ContainerImage` 명령을 사용하여 확인할 수 있습니다. 부모 이미지가 목록에 없으면 해당 이미지가 기본 OS 이미지임을 나타냅니다.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

### 이미지 리포지토리 이동

`New-ContainerImage` 명령을 사용하여 새 컨테이너 이미지를 만들 경우 이 이미지는 기본 위치 'C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store'에 저장됩니다. `Move-ContainerImageRepository` 명령을 사용하여 이 리포지토리를 이동할 수 있습니다. 예를 들어 다음은 'c:\container-images' 위치에 새 컨테이너 이미지 리포지토리를 만듭니다.

```powershell
Move-ContainerImageRepository -Path c:\container-images
```
> `Move-ContainerImageRepository` 명령에 사용된 경로는 명령 실행 시 아직 존재하지 않아야 합니다.

## 컨테이너 이미지 Docker

### 이미지 나열

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### 새 이미지 만들기

모든 기존 컨테이너에서 새 컨테이너 이미지를 만들 수 있습니다. 이를 위해 `docker commit` 명령을 사용합니다. 다음 예제에서는 이름이 'windowsservercoreiis'인 새 컨테이너 이미지를 만듭니다.

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

### 이미지 종속성

Docker를 통한 이미지 종속성을 확인하려면 `docker history` 명령을 사용하면 됩니다.

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker 허브

Docker 허브 레지스트리는 컨테이너 호스트에 다운로드할 수 있는 미리 빌드된 이미지를 포함합니다. 이 이미지를 다운로드한 후에는 Windows 컨테이너 응용 프로그램을 위한 기반으로 사용할 수 있습니다.

Docker 허브에서 사용 가능한 이미지 목록을 보려면 `docker search` 명령을 사용합니다. 참고 - Docker 허브에서 이러한 종속 컨테이너 이미지를 끌어오려면 먼저 Windows Serve Core 또는 Nano Server 기본 OS 이미지를 설치해야 합니다.

> "nano-"로 시작하는 이미지는 Nano Server 기본 OS 이미지에 대한 종속성이 있습니다.

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
microsoft/nano-golang   Go Programming Language installed in a Nan...   1                    [OK]
microsoft/nano-httpd    Apache httpd installed in a Nano Server ba...   1                    [OK]
microsoft/nano-iis      Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/nano-mysql    MySQL installed in a Nano Server based con...   1                    [OK]
microsoft/nano-nginx    Nginx installed in a Nano Server based con...   1                    [OK]
microsoft/nano-node     Node installed in a Nano Server based cont...   1                    [OK]
microsoft/nano-python   Python installed in a Nano Server based co...   1                    [OK]
microsoft/nano-rails    Ruby on Rails installed in a Nano Server b...   1                    [OK]
microsoft/nano-redis    Redis installed in a Nano Server based con...   1                    [OK]
microsoft/nano-ruby     Ruby installed in a Nano Server based cont...   1                    [OK]
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




<!--HONumber=Mar16_HO3-->
