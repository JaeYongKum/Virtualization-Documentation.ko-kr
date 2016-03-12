



# 컨테이너 공유 폴더

**이 예비 콘텐츠는 변경될 수 있습니다.**

공유 폴더를 사용하면 컨테이너 호스트와 컨테이너 간에 데이터를 공유할 수 있습니다. 공유 폴더를 만들면 공유 폴더를 컨테이너 안에서 사용할 수 있습니다. 호스트로부터 공유 폴더에 배치된 모든 데이터는 컨테이너 안에서 사용할 수 있게 됩니다. 컨테이너 안에서 공유 폴더에 배치된 모든 데이터는 호스트에서 사용할 수 있게 됩니다. 호스트의 단일 폴더를 여러 컨테이너에서 공유할 수 있으며 이러한 구성에서는 실행 중인 컨테이너 간에 데이터를 공유할 수 있습니다.

## 데이터 관리 - PowerShell

### 공유 폴더 만들기

공유 폴더를 만들려면 `Add-ContainerSharedFolder` 명령을 사용합니다. 아래 예제에서는 `c:\shared_data` 컨테이너에 디렉터리를 만들고 이 디렉터리는 호스트의 `c:\data_source` 디렉터리에 매핑됩니다.

> 컨테이너는 공유 폴더를 추가할 때 중지 상태에 있어야 합니다.

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\data_source -DestinationPath c:\shared_data

ContainerName SourcePath       DestinationPath AccessMode
------------- ----------       --------------- ----------
DEMO          c:\data_source   c:\shared_data  ReadWrite
```

### 읽기 전용 공유 폴더

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\sf1 -DestinationPath c:\sf2 -AccessMode ReadOnly

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\sf1     c:\sf2          ReadOnly
```

### 공유 폴더 나열

특정 컨테이너의 공유 폴더 목록을 보려면 `Get-ContainerSharedFolder` 명령을 사용합니다.

```powershell
PS C:\> Get-ContainerSharedFolder -ContainerName DEMO2

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\source  c:\source       ReadWrite
```

### 공유 폴더 수정

기존 공유 폴더 구성을 수정하려면 `Set-ContainerSharedFolder` 명령을 사용합니다.

```powershell
PS C:\> Set-ContainerSharedFolder -ContainerName SFRO -SourcePath c:\sf1 -DestinationPath c:\sf1
```

### 공유 폴더 제거

공유 폴더를 제거하려면 `Remove-ContainerSharedFolder` 명령을 사용합니다.

> 컨테이너는 공유 폴더를 제거할 때 중지 상태에 있어야 합니다.

```powershell
PS C:\> Remove-ContainerSharedFolder -ContainerName DEMO2 -SourcePath c:\source -DestinationPath c:\source
```
## 데이터 관리 - Docker

### 볼륨 마운트

Windows 컨테이너를 Docker로 관리할 때는 `-v` 옵션으로 볼륨을 탑재할 수 있습니다.

아래 예제에서는 원본 폴더가 c:\source, 대상 폴더가 c:\destination입니다.

```powershell
PS C:\> docker run -it -v c:\source:c:\destination 1f62aaf73140 cmd
```

Docker를 통한 컨테이너의 데이터 관리에 대한 자세한 내용은 [Docker.com의 Docker 볼륨](https://docs.docker.com/userguide/dockervolumes/)을 참조하세요.

## 비디오 연습

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-3-Shared-Folders/player#ccLang=ko" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>



<!--HONumber=Feb16_HO3-->
