
# <a name="using-insider-container-images"></a>참가자 컨테이너 이미지 사용

이 연습에서는 Windows Insider Preview 프로그램의 최신 Windows Server 참가자 빌드에 Windows 컨테이너 기능을 배포하고 사용하는 방법을 안내합니다. 이 연습을 진행하는 과정에서 컨테이너 역할을 설치하고 기본 OS 이미지의 미리 보기 버전을 배포할 것입니다. 컨테이너에 대해 좀 더 숙지해야 하는 경우 [컨테이너 정보](../about/index.md)에서 이 정보를 찾을 수 있습니다.

이 빠른 시작은 Windows Server Insider Preview 프로그램의 Windows Server 컨테이너와 관련이 있습니다. 이 빠른 시작을 시작하기 전에 프로그램에 대해 충분히 숙지하시기 바랍니다.

## <a name="prerequisites"></a>필수 조건:

- [Windows 참가자 프로그램](https://insider.windows.com/GettingStarted)에 가입하고 사용 약관 검토.
- Windows 참가자 프로그램의 최신 Windows Server 및/또는 Windows 참가자 프로그램의 최신 Windows 10 빌드를 실행하는 컴퓨터 시스템(실제 또는 가상) 하나.

> [!IMPORTANT]
> 아래에 설명 된 기본 이미지를 사용 하 여 Windows Insider Preview 프로그램의 Windows 10 빌드 또는 Windows Server Insider Preview 프로그램의 Windows Server의 빌드를 사용 해야 합니다. 두 빌드 중 하나를 사용하지 않을 경우 이러한 기본 이미지를 사용하면 컨테이너가 시작되지 않는 오류가 발생합니다.

## <a name="install-docker-enterprise-edition-ee"></a>Docker Enterprise Edition(EE) 설치

Windows 컨테이너를 사용하려면 Docker EE가 필요합니다. Docker EE는 Docker 엔진 및 Docker 클라이언트로 구성됩니다.

Docker EE를 설치하기 위해 OneGet provider PowerShell module(OneGet 공급자 PowerShell 모듈)을 사용합니다. 공급자는 컴퓨터에서 컨테이너 기능을 사용하도록 설정하고 Docker EE를 설치합니다. 그러면 컴퓨터를 다시 부팅해야 합니다. 관리자 권한으로 PowerShell 세션을 열고 다음 명령을 실행합니다.

> [!NOTE]
> Windows Server 참가자 빌드를 사용 하 여 Docker EE를 설치 하려면 비 참가자 빌드에 사용 되는 것과 다른 OneGet 공급자가 필요 합니다. Docker EE와 DockerMsftProvider OneGet 공급자가 이미 설치된 경우 계속하기 전에 이를 제거합니다.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Windows 참가자 빌드와 함께 사용할 OneGet PowerShell 모듈을 설치합니다.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

OneGet을 사용하여 최신 버전의 Docker EE 미리 보기를 설치합니다.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

설치가 완료되면 컴퓨터를 다시 부팅합니다.

```powershell
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>기본 컨테이너 이미지 설치

Windows 컨테이너를 사용하려면 기본 이미지를 설치해야 합니다. Windows 참가자 프로그램에 가입하면 기본 이미지에 대해 최신 빌드를 테스트할 수도 있습니다. 참가자 기본 이미지를 사용하면 Windows Server 기반의 기본 이미지 4개가 제공됩니다. 아래 표에서 각 이미지의 용도를 확인하세요.

| 기본 OS 이미지                       | 용도                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | 프로덕션 및 개발 |
| mcr.microsoft.com/windows/nanoserver              | 프로덕션 및 개발 |
| mcr.microsoft.com/windows/servercore/insider | 개발 전용           |
| mcr.microsoft.com/windows/nanoserver/insider        | 개발 전용           |

Nano 서버 참가자 기본 이미지를 가져오려면 다음을 실행합니다.

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Windows Server Core 참가자 기본 이미지를 가져오려면 다음을 실행합니다.

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Windows 컨테이너 OS 이미지 [EULA](../EULA.md ) 및 Windows 참가자 프로그램 [사용 약관을](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)읽어 보십시오.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [빌드 및 샘플 응용 프로그램 실행](./Nano-RS3-.NET-Core-and-PS.md)
