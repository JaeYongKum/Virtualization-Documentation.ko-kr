---
title: 샘플 앱 빌드
description: 컨테이너를 활용하는 동안 샘플 앱을 빌드하는 방법을 알아보세요.
keywords: Docker, 컨테이너
author: cwilhit
ms.date: 07/25/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 9cfb5cb062259e906ce499423619ec7a5b814ac9
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620841"
---
# <a name="build-a-sample-app"></a>샘플 앱 빌드

이 연습에서는 샘플 ASP.net 앱을 가져온 후 컨테이너에서 실행되도록 변환하는 방법을 안내합니다. Windows 10에서 컨테이너를 작동하는 방법을 알아보려면 [Windows 10 빠른 시작](./quick-start-windows-10.md)을 참조하세요.

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다. 이 자습서의 초점은 컨테이너에 있으므로 코드 작성 없이 오직 컨테이너에만 집중하겠습니다. 자습서를 처음부터 빌드하고 싶으면 [ASP.NET Core 설명서](https://docs.microsoft.com/aspnet/core/tutorials/first-mvc-app-xplat/)를 참조하세요.

컴퓨터에 Git 소스 컨트롤이 설치되지 않은 경우 [Git](https://git-scm.com/download)에서 다운로드할 수 있습니다.

## <a name="getting-started"></a>시작

이 샘플 프로젝트는 [VSCode](https://code.visualstudio.com/)를 사용하여 설정되었습니다. Powershell도 사용할 것입니다. github에서 데모 코드를 가져옵시다. git를 사용하여 리포지토리를 복제해도 되고 [SampleASPContainerApp](https://github.com/cwilhit/SampleASPContainerApp)에서 직접 프로젝트를 다운로드해도 됩니다.

```Powershell
git clone https://github.com/cwilhit/SampleASPContainerApp.git
```

이제 프로젝트 디렉터리로 이동하여 Dockerfile을 만듭시다. [Dockerfile](https://docs.docker.com/engine/reference/builder/)은 메이크파일과 비슷한 파일로, 컨테이너 이미지를 빌드하는 방법을 설명하는 명령 목록입니다.

```Powershell
#Create the dockerfile for our proj
New-Item C:/Your/Proj/Location/Dockerfile -type file
```

## <a name="writing-our-dockerfile"></a>Dockerfile 쓰기

프로젝트 루트 폴더에 만든 이 Dockerfile을 열고(선호하는 텍스트 편집기를 사용하여) 몇 가지 논리를 추가해 봅시다. 그런 다음 한 줄씩 살펴보면서 상황을 설명하겠습니다.

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

첫 번째 선 그룹은 컨테이너를 빌드하는 데 사용할 기반이 되는 기본 이미지를 선언합니다. 로컬 시스템에 이 이미지가 없으면 docker가 자동으로 이미지를 가져오려고 시도합니다. Aspnetcore 빌드는 기본적으로 프로젝트를 컴파일하기 위한 종속성이 함께 패키징됩니다. 그런 다음 컨테이너의 작업 디렉터리를 '/app'으로 변경합니다. 그러면 dockerfile의 모든 확인 명령이 여기서 실행됩니다.

_참고_: 우리 목적은 프로젝트를 빌드하는 것이고 우리가 만드는 첫 번째 컨테이너는 이 목적에만 사용되는 임시 컨테이너로, 작업이 끝나면 삭제할 것입니다.

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app
```

다음으로 임시 컨테이너의 '/app' 디렉터리에 .csproj 파일을 복사합니다. 프로젝트에 필요한 패키지 참조 목록이.csproj 파일에 포함 되어 있으므로이 야기 합니다.

이 파일을 복사하면 dotnet이 이 파일을 읽은 후 프로젝트에 필요한 모든 종속성과 도구를 가져옵니다.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

이러한 종속성을 모두 가져온 후 임시 컨테이너로 복사합니다. 그런 다음 릴리스 구성을 통해 응용 프로그램을 게시하라고 dotnet에 알리고 출력 경로를 지정합니다.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

프로젝트가 성공적으로 컴파일될 것입니다. 이제 최종 컨테이너를 빌드해야 합니다. 우리 응용 프로그램은 ASP.NET이므로 라이브러리를 원본으로 사용하여 이미지를 지정합니다. 그런 다음 임시 컨테이너의 출력 디렉터리에 들어 있는 모든 파일을 최종 컨테이너에 복사합니다. 실행할 때 컴파일한 새 .dll을 사용하여 실행되도록 컨테이너를 구성합니다.

_참고_: 이 최종 컨테이너에 사용되는 기본 이미지는 위의 ```FROM``` 명령과 비슷하지만 ASP.NET 앱을 _빌드_할 수 있는 라이브러리가 없고 실행할 수만 있다는 차이가 있습니다.

```Dockerfile
FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

소위 말하는 _다단계 빌드_라는 것을 성공적으로 수행했습니다. 최종 결과의 크기를 최소화하기 위해 임시 컨테이너를 사용하여 이미지를 빌드한 다음 게시된 dll을 다른 컨테이너로 이동했습니다. 우리는 이 컨테이너의 종속성을 실행에 필요한 최소한의 수준으로 유지해야 합니다. 만약 첫 번째 이미지를 계속 사용했다면 필수적이지 않은 다른 계층이 포함되었을 것이며(ASP.NET 앱을 빌드하기 위해), 이렇게 되면 이미지 크기가 더 커졌을 것입니다.

## <a name="running-the-app"></a>앱 실행

dockerfile이 작성되었으니, 앱을 빌드한 다음 컨테이너를 실행하라고 docker에 알려주기만 하면 됩니다. 게시할 포트를 지정하고 컨테이너에 "myapp"이라는 태그를 지정합니다. powershell에서 다음 명령을 실행합니다.

_참고_: 사용자의 PowerShell 콘솔의 현재 작업 디렉터리는 위에서 만든 dockerfile이 있는 디렉터리여야 합니다.

```Powershell
docker build -t myasp .
docker run -d -p 5000:80 --name myapp myasp
```

앱이 실행되는 것을 보려면 앱이 실행되고 있는 주소를 방문해야 합니다. 이 명령을 실행하여 IP 주소를 가져옵니다.

```Powershell
 docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" myapp
```

이 명령을 실행하면 실행 중인 컨테이너의 IP 주소를 얻을 수 있습니다. 그러면 다음과 같은 출력이 표시됩니다.

```Powershell
 172.19.172.12
```

원하는 웹 브라우저에 이 IP 주소를 입력하면 컨테이너에서 성공적으로 실행되고 있다는 응용 프로그램의 메시지가 표시됩니다!

<center style="margin: 25px">![](media/SampleAppScreenshot.png)</center>

탐색 모음에서 "MvcMovie"를 클릭하면 동영상 항목을 입력, 편집 및 삭제할 수 있는 웹 페이지로 이동됩니다.

## <a name="next-steps"></a>다음 단계

성공적으로 ASP.NET 웹앱을 가져오고, Docker를 사용하여 구성 및 빌드하고, 실행 중인 컨테이너에 배포했습니다. 하지만 수행 가능한 추가 단계가 더 남아 있습니다! 웹앱을 더 많은 구성 요소(웹 API를 실행하는 컨테이너, 프런트 엔드를 실행하는 컨테이너, SQL 서버를 실행하는 컨테이너)로 분할할 수 있습니다.

컨테이너의 중단을가지고 했으므로 높 하 고 컨테이너 화 멋진 소프트웨어 빌드!

> [!div class="nextstepaction"]
> [더 많은 컨테이너 샘플 확인](../samples.md)
