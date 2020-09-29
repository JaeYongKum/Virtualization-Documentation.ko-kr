---
title: .NET Core 앱 컨테이너 화
description: 컨테이너를 사용하여 샘플 .NET Core 앱을 빌드하는 방법 알아보기
keywords: Docker, 컨테이너
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: quickstart
ms.openlocfilehash: e04e4ee4c38409dfd20e4426d586839e1741651e
ms.sourcegitcommit: 160405a16d127892b6e2897efa95680f29f0496a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/22/2020
ms.locfileid: "90990656"
---
# <a name="containerize-a-net-core-app"></a>.NET Core 앱 컨테이너 화

이 문서에서는 [시작: 컨테이너에 맞게 Windows 준비](set-up-environment.md)에 설명된 대로 환경을 설정하고, [첫 번째 Windows 컨테이너 실행](run-your-first-container.md)에 설명된 대로 첫 번째 컨테이너를 실행한 후 기존 샘플 .NET 앱을 Windows 컨테이너로 배포하기 위해 패키징하는 방법을 설명합니다.

또한 컴퓨터에 Git 소스 제어 시스템이 설치되어 있어야 합니다. 설치하려면 [Git](https://git-scm.com/download)를 방문하세요.

## <a name="clone-the-sample-code-from-github"></a>GitHub에서 샘플 코드 복제

모든 컨테이너 샘플 소스 코드는 [Virtualization-Documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation) Git 리포지토리(비공식적으로 리포라고도 함)의 `windows-container-samples`라는 폴더에 보관됩니다.

1. PowerShell 세션을 열고 이 리포지토리를 저장할 폴더로 디렉터리를 변경합니다. 다른 명령 프롬프트 창 유형도 작동하지만 예제 명령은 PowerShell을 사용합니다.
2. 현재 작업 디렉터리로 리포를 복제합니다.

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 다음 명령을 사용하여 `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` 아래에 있는 샘플 디렉터리로 이동하고 Dockerfile을 만듭니다.

   [Dockerfile](https://docs.docker.com/engine/reference/builder/)은 메이크파일과 마찬가지로 컨테이너 엔진에 컨테이너 이미지를 빌드하는 방법을 알려주는 명령 목록입니다.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Dockerfile 작성

원하는 텍스트 편집기를 사용하여 방금 만든 Dockerfile을 연 후 다음 콘텐츠를 추가합니다.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

한 줄씩 살펴보면서 각 명령의 역할을 설명하겠습니다.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

첫 번째 선 그룹은 컨테이너를 빌드하는 데 사용할 기반이 되는 기본 이미지를 선언합니다. 로컬 시스템에 이 이미지가 없으면 docker가 자동으로 이미지를 가져오려고 시도합니다. `mcr.microsoft.com/dotnet/core/sdk:2.1`은 .NET Core 2.1 SDK가 설치된 상태로 제공되므로 버전 2.1을 대상으로 하는 ASP .NET Core 프로젝트를 빌드하는 작업에 의존합니다. 다음 명령은 컨테이너의 작업 디렉터리를 `/app`로 변경하므로 이 다음에 오는 모든 명령이 이 컨텍스트에서 실행됩니다.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

그런 다음 이 명령은 .csproj 파일을 `build-env` 컨테이너의 `/app` 디렉터리로 복사합니다. 이 파일을 복사하면 .NET이 이 파일을 읽은 후 프로젝트에 필요한 모든 종속성과 도구를 가져옵니다.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

.NET이 모든 종속성을 `build-env` 컨테이너로 끌어오면 다음 명령이 모든 프로젝트 소스 파일을 컨테이너에 복사합니다. 그런 다음 릴리스 구성을 통해 애플리케이션을 게시하라고 .NET에 알리고 출력 경로를 지정합니다.

컴파일이 성공합니다. 이제 최종 이미지를 빌드해야 합니다.

> [!TIP]
> 이 빠른 시작은 소스에서 .NET Core 프로젝트를 빌드합니다. 컨테이너 이미지를 빌드할 때 컨테이너 이미지에 프로덕션 페이로드 및 해당 _종속성만_ 포함하는 것이 좋습니다. .Net Core 런타임만 필요하기 때문에 .NET Core SDK는 최종 이미지에 포함하지 않아야 하므로 SDK와 함께 패키징된 `build-env`라는 임시 컨테이너를 사용하여 앱을 빌드하도록 Dockerfile이 작성됩니다.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

애플리케이션은 ASP.NET이므로 이 런타임을 포함하여 이미지를 지정합니다. 그런 다음 임시 컨테이너의 출력 디렉터리에 들어 있는 모든 파일을 최종 컨테이너에 복사합니다. 컨테이너가 시작될 때 진입점인 새 앱과 함께 실행되도록 컨테이너를 구성합니다.

_다단계 빌드_를 수행하도록 Dockerfile을 작성했습니다. Dockerfile이 실행되면 .NET Core 2.1 SDK와 함께 임시 컨테이너 `build-env`를 사용하여 샘플 앱을 빌드한 다음, 출력된 이진 파일을 .NET Core 2.1 런타임만 포함된 다른 컨테이너에 복사할 것이므로 최종 컨테이너의 크기를 최소화했습니다.

## <a name="build-and-run-the-app"></a>앱 빌드 및 실행

Dockerfile이 작성되면 Dockerfile에 Docker를 가리키고 이미지를 빌드한 후 실행하도록 지시할 수 있습니다.

1. 명령 프롬프트 창에서 Dockerfile이 있는 디렉터리로 이동한 다음, [docker build](https://docs.docker.com/engine/reference/commandline/build/) 명령을 실행하여 Dockerfile에서 컨테이너를 빌드합니다.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 새로 빌드된 컨테이너를 실행하려면 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 명령을 실행합니다.

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   다음 명령을 살펴보겠습니다.

   * `-d`는 Docker에 컨테이너를 '분리한 상태로' 실행하도록 지시합니다. 즉, 컨테이너 내 콘솔에 연결된 콘솔이 없습니다. 컨테이너가 백그라운드에서 실행됩니다.
   * `-p 5000:80`은 호스트의 포트 5000을 컨테이너의 포트 80에 매핑하도록 Docker에 지시합니다. 각 컨테이너는 고유의 IP 주소를 가져옵니다. ASP.NET은 기본적으로 포트 80에서 수신 대기합니다. 포트 매핑을 사용하면 매핑된 포트에서 호스트의 IP 주소로 이동하고 Docker는 컨테이너 내의 대상 포트로 모든 트래픽을 전달합니다.
   * `--name myapp`은 쿼리할 때 기준으로 사용하기 편리한 이름을 이 컨테이너에 지정하도록 Docker에 지시합니다(Docker에서 런타임에 할당한 컨테이너 ID를 조회할 필요가 없음).
   * `my-asp-app`은 Docker에서 실행하도록 하려는 이미지입니다. `docker build` 프로세스의 완성작으로 생성된 컨테이너 이미지입니다.

3. 다음 스크린샷에 표시된 것처럼 웹 브라우저를 열고 `http://localhost:5000`으로 이동하여 컨테이너화된 애플리케이션을 확인합니다.

   >![컨테이너의 localhost에서 실행되는 ASP.NET Core 웹 페이지](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>다음 단계

1. 다음 단계는 Azure Container Registry를 사용하여 프라이빗 레지스트리에 컨테이너화된 ASP.NET 웹앱을 게시하는 것입니다. 이렇게 하면 조직에 배포할 수 있습니다.

   > [!div class="nextstepaction"]
   > [프라이빗 컨테이너 레지스트리 만들기](/azure/container-registry/container-registry-get-started-powershell)

   [컨테이너 이미지를 레지스트리로 푸시](/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)한 섹션으로 이동하는 경우 컨테이너 레지스트리(예: `contoso-container-registry`)와 함께 방금 패키징한 ASP.NET 앱의 이름(`my-asp-app`)을 지정합니다.

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   더 많은 앱 샘플 및 관련 Dockerfile을 보려면 [추가 컨테이너 샘플](../samples.md)을 참조하세요.

2. 컨테이너 레지스트리에 앱을 게시한 후 다음 단계는 Azure Kubernetes Service를 사용하여 만든 Kubernetes 클러스터에 앱을 배포하는 것입니다.

   > [!div class="nextstepaction"]
   > [Kubernetes 클러스터 만들기](/azure/aks/windows-container-cli)