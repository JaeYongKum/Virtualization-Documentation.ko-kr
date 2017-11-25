# <a name="build-and-run-an-application-with-or-without-net-core-20-or-powershell-core-6"></a>.NET Core 2.0 또는 PowerShell Core 6를 사용하여 또는 사용하지 않고 응용 프로그램 빌드 및 실행

.NET Core와 PowerShell이 기본 Nano 서버 컨테이너 위에 추가 기능 계층형 컨테이너로 지원되기는 하지만 이 릴리스의 Nano 서버 기본 OS 컨테이너 이미지에서는 .NET Core와 PowerShell이 제거되었습니다.  

컨테이너에서 Node.js, Python, Ruby 등의 원시 코드 또는 개방형 프레임워크를 실행하는 경우 기본 Nano 서버 컨테이너면 충분합니다.  이번 릴리스는 Windows Server 2016 릴리스에 비해 [사용 공간이 축소](https://docs.microsoft.com/en-us/windows-server/get-started/nano-in-semi-annual-channel)되었으며 그로 인해 특정 원시 코드가 실행되지 않을 수 있습니다. 성능 저하 문제가 발견되면 [포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)을 통해 알려주세요. 

Dockerfile에서 컨테이너를 빌드하려면 docker build를 사용하고, 실행하려면 docker run을 사용합니다.  다음 명령은 Nano 서버 컨테이너 기본 OS 이미지를 다운로드하고(몇 분 정도 걸릴 수 있음) 호스트 콘솔에서 “Hello World!”라는 메시지를 인쇄합니다.

```
docker run microsoft/nanoserver-insider cmd /c echo Hello World!
```

[Windows의 Dockerfile](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile)과 FROM, RUN, COPY, ADD, CMD 등의 Dockerfile 구문을 사용하여 좀 더 복잡한 응용 프로그램을 빌드할 수 있습니다. 특정 명령은 이 기본 이미지에서 즉시 실행할 수 없지만, 이제는 응용 프로그램 작동에 필요한 것만 포함되어 있는 컨테이너 이미지를 만들 수 있습니다.

기본 Nano 서버 컨테이너 OS 이미지에서 .NET Core와 PowerShell을 사용할 수 없게 되면서, 압축된 zip 형식의 콘텐츠가 포함된 컨테이너를 어떻게 빌드할 것인지에 대한 과제가 생겼습니다. Docker 17.05에 제공되는 [다단계 빌드](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) 기능을 사용하면 다른 컨테이너에서 PowerShell을 활용하여 콘텐츠 압축을 풀고 Nano 컨테이너에 복사할 수 있습니다. 이 방법을 사용하여 .NET Core 컨테이너와 PowerShell 컨테이너를 만들 수 있습니다. 

다음 명령을 사용하여 PowerShell 컨테이너 이미지를 가져올 수 있습니다.

```
docker pull microsoft/nanoserver-insider-powershell
```

다음 명령을 사용하여 .NET Core 컨테이너 이미지를 가져올 수 있습니다.

```
docker pull microsoft/nanoserver-insider-dotnet
```

아래는 다단계 빌드를 사용하여 이러한 컨테이너 이미지를 만드는 방법에 대한 몇 가지 예입니다.

## <a name="deploy-apps-based-on-net-core-20"></a>.NET Core 2.0을 기반으로 앱 배포
참가자 릴리스에서 .NET Core 2.0 컨테이너 이미지를 활용하여 .NET Core 앱을 실행할 수 있으며, .NET Core 응용 프로그램은 다른 곳에 빌드되고 개발자는 응용 프로그램을 컨테이너에서 실행하려 합니다.  .NET Core 컨테이너 이미지를 사용하여 .NET Core 응용 프로그램을 실행하는 방법에 대한 자세한 내용은 [.NET Core GitHub](https://github.com/dotnet/dotnet-docker-nightly)에서 확인할 수 있습니다.  컨테이너 내부에서 응용 프로그램을 개발하는 경우 .NET Core SDK를 대신 사용해야 합니다.  고급 사용자의 경우 .NET Core 2.0 버전, Dockerfile 및 [dotnet-docker-nightly](https://github.com/dotnet/dotnet-docker-nightly/tree/master/2.0)에 지정된 URL을 사용하여 고유의 .NET Core 2.0 컨테이너를 빌드할 수 있습니다. 이렇게 하려면 Windows Server Core 컨테이너를 사용하여 기능 다운로드 및 압축 해제를 수행할 수 있습니다.  Dockerfile 샘플은 [.NET Core 런타임 Dockerfile](https://github.com/dotnet/dotnet-docker-nightly/blob/master/2.0/runtime/nanoserver-insider/amd64/Dockerfile)과 같습니다.


이 Dockerfile을 사용하면 다음 명령을 사용하여 .NET Core 2.0 컨테이너를 빌드할 수 있습니다.

```
docker build -t nanoserverdnc -f Dockerfile-dotnetRuntime .
```

## <a name="run-powershell-core-6-in-a-container"></a>컨테이너에서 PowerShell Core 6 실행
동일한 [다단계 빌드](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) 방법을 사용하여 [이 PowerShell Dockerfile](https://github.com/PowerShell/PowerShell/blob/master/docker/release/nanoserver-insider/Dockerfile)로 PowerShell Core 6 컨테이너를 빌드할 수 있습니다.


그런 다음 docker build를 실행하여 PowerShell 컨테이너 이미지를 만듭니다.

``` 
docker build -t nanoserverPowerShell6 -f Dockerfile-PowerShell6 .
```

[PowerShell GitHub](https://github.com/PowerShell/PowerShell/tree/master/docker/release)에서 더 많은 정보를 찾을 수 있습니다.  PowerShell zip에는 PowerShell Core 6를 빌드하는 데 필요한 .NET Core 2.0 하위 집합이 포함되어 있습니다.  PowerShell 모듈이 .NET Core 2.0을 사용하는 경우 기본 Nano 컨테이너 대신 Nano .NET Core 컨테이너 위에, 다시 말해서 Dockerfile에서 FROM microsoft/nanoserver-insider-dotnet을 사용하여 Dockerfile.PowerShell 컨테이너를 빌드하는 것이 안전합니다. 

## <a name="next-steps"></a>다음 단계
- Docker 허브에 제공되는 Nano 서버 기반의 새 컨테이너 이미지(즉 기본 Nano 서버 이미지) 중 하나, Nano with .NET Core 2.0 및 Nano with PowerShell Core 6 사용
- 이 가이드의 Dockerfile 샘플 콘텐츠를 사용하여 새 Nano 서버 컨테이너 기본 OS 이미지를 기반으로 고유의 컨테이너 이미지 만들기 
