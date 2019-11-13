---
title: Containerize .NET Core 앱
description: 컨테이너를 사용 하 여 샘플 .NET core 앱을 빌드하는 방법 배우기
keywords: Docker, 컨테이너
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288071"
---
# <a name="containerize-a-net-core-app"></a>Containerize .NET Core 앱

이 항목에서는 [시작](set-up-environment.md)에 설명 된 대로 환경을 설정 하 고 windows 컨테이너로 배포 하기 위해 기존 샘플 .net 앱을 패키지 하는 방법을 설명 하 고 [첫 번째 windows 컨테이너 실행](run-your-first-container.md)에서 설명한 대로 첫 번째 컨테이너를 실행 합니다.

또한 컴퓨터에 Git 소스 제어 시스템을 설치 해야 합니다. 설치 하려면 [Git](https://git-scm.com/download)을 방문 하세요.

## <a name="clone-the-sample-code-from-github"></a>GitHub에서 샘플 코드 복제

모든 컨테이너 샘플 소스 코드는 호출 `windows-container-samples`되는 폴더에서 [가상화-문서](https://github.com/MicrosoftDocs/Virtualization-Documentation) git 리포지토리 (리포지토리로 라고도 함)에 보관 됩니다.

1. PowerShell 세션을 열고이 저장소를 저장할 폴더에 대 한 디렉터리를 변경 합니다. 다른 명령 프롬프트 창 유형도 작동 하지만이 예제 명령에는 PowerShell이 사용 됩니다.
2. 리포지토리를 현재 작업 디렉터리로 복제 합니다.

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 다음 명령을 사용 하 여 아래 `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` 에 있는 샘플 디렉터리로 이동 하 여 Dockerfile을 만듭니다.

   [Dockerfile](https://docs.docker.com/engine/reference/builder/) 은 메이크파일의와 비슷하지만 컨테이너 엔진에 컨테이너 이미지를 작성 하는 방법을 알려 주는 지침 목록입니다.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Dockerfile 작성

원하는 텍스트 편집기를 사용 하 여 방금 만든 Dockerfile을 열고 다음 콘텐츠를 추가 합니다.

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

줄 단위로 나누어 각 명령의 기능에 대해 설명 하겠습니다.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

첫 번째 선 그룹은 컨테이너를 빌드하는 데 사용할 기반이 되는 기본 이미지를 선언합니다. 로컬 시스템에 이 이미지가 없으면 docker가 자동으로 이미지를 가져오려고 시도합니다. 제공 `mcr.microsoft.com/dotnet/core/sdk:2.1` 되는 패키지는 .net CORE 2.1 SDK가 설치 되어 있으므로 버전 2.1를 대상으로 하는 ASP .net 핵심 프로젝트를 작성 하는 작업까지 제공 됩니다. 다음 명령은 컨테이너의 작업 디렉터리를 변경 하 여이 컨텍스트에서 `/app`실행 되는 모든 명령이이 컨텍스트 아래에서 수행 됩니다.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

다음으로이 명령은 .csproj 파일을 `build-env` 컨테이너의 `/app` 디렉터리로 복사 합니다. 이 파일을 복사한 후 .NET에서 읽어 온 후 프로젝트에서 필요한 모든 종속성과 도구를 가져와 가져옵니다.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

.NET에서 `build-env` 컨테이너에 모든 종속성을 가져온 후 다음 명령은 모든 프로젝트 소스 파일을 컨테이너에 복사 합니다. 그런 다음 .NET에서 릴리스 구성으로 응용 프로그램을 게시 하 고에서 출력 경로를 지정 하도록 합니다.

컴파일은 성공적 이어야 합니다. 이제 최종 이미지를 작성 해야 합니다. 

> [!TIP]
> 이 퀵 스타트는 소스에서 .NET 코어 프로젝트를 빌드합니다. 컨테이너 이미지를 빌드할 때 컨테이너 이미지에 프로덕션 페이로드 및 해당 종속성 _만_ 포함 하는 것이 좋습니다. .Net core 런타임이 필요 하기 때문에 마지막 이미지에는 .NET core SDK가 포함 되어 있지 않으므로, 앱을 빌드하기 위해 호출 `build-env` 되는 SDK와 함께 패키지 된 임시 컨테이너를 사용 하도록 dockerfile이 작성 되지 않도록 합니다.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

이 응용 프로그램은 ASP.NET 이므로이 런타임이 포함 된 이미지를 지정 합니다. 그런 다음 임시 컨테이너의 출력 디렉터리에 들어 있는 모든 파일을 최종 컨테이너에 복사합니다. 컨테이너가 시작 될 때 진입점으로 새 앱을 실행 하도록 컨테이너를 구성 합니다.

_여러 단계 빌드_를 수행 하는 dockerfile을 작성 했습니다. Dockerfile이 실행 되 면 임시 컨테이너를 사용 하 고 `build-env`, .net CORE 2.1 SDK를 통해 샘플 앱을 빌드한 다음, .net core 2.1 런타임에서만 포함 된 다른 컨테이너에 출력 된 이진 파일을 복사 하 여 최종 컨테이너의 크기를 최소화 하도록 합니다.

## <a name="build-and-run-the-app"></a>앱 빌드 및 실행

Dockerfile이 기록 된 경우 Dockerfile에서 Docker를 가리켜 이미지를 작성 한 다음 실행 하도록 지시할 수 있습니다.

1. 명령 프롬프트 창에서 dockerfile이 있는 디렉터리로 이동한 다음 [docker 빌드](https://docs.docker.com/engine/reference/commandline/build/) 명령을 실행 하 여 dockerfile에서 컨테이너를 빌드합니다.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 새로 빌드된 컨테이너를 실행 하려면 [docker 실행](https://docs.docker.com/engine/reference/commandline/run/) 명령을 실행 합니다.

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   다음 명령을 dissect 하겠습니다.

   * `-d` tun에서 ' 분리 됨 ' 컨테이너를 실행 하도록 지시 하 여 컨테이너 안의 콘솔에 콘솔이 연결 되지 않았음을 의미 합니다. 컨테이너는 백그라운드에서 실행 됩니다. 
   * `-p 5000:80` 호스트의 포트 5000을 컨테이너의 포트 80에 매핑하여 Docker에 알립니다. 각 컨테이너는 고유한 IP 주소를 얻습니다. ASP .NET은 기본적으로 포트 80를 수신 합니다. 포트 매핑을 사용 하면 매핑된 포트에서 호스트의 IP 주소로 이동 하 고 Docker는 모든 트래픽을 컨테이너 내부의 대상 포트로 전달 합니다.
   * `--name myapp` 런타임에 Docker에 의해 할당 된 contaienr ID를 조회 하는 대신이 컨테이너에 편리한 이름을 제공 하도록 Docker에 지시 합니다.
   * `my-asp-app` 는 Docker를 실행할 이미지입니다. 이 컨테이너 이미지는 `docker build` 프로세스의 culmination으로 생성 됩니다.

3. 이 스크린샷은 다음 그림과 같이 웹 브라우저 웹 브라우저 `http://localhost:5000` 를 열고 containerized 응용 프로그램이 표시 되도록 탐색 합니다.

   >![컨테이너의 localhost에서 실행 되는 ASP.NET 핵심 웹 페이지](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>다음 단계

1. 다음 단계는 Azure 컨테이너 레지스트리를 사용 하 여 개인 레지스트리에 containerized ASP.NET 웹 앱을 게시 하는 것입니다. 이렇게 하면 조직에 배포할 수 있습니다.

   > [!div class="nextstepaction"]
   > [개인 컨테이너 레지스트리 만들기](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   [컨테이너 이미지를 레지스트리에 푸시하](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)는 섹션으로 이동 하는 경우 컨테이너 레지스트리와 함께 방금 패키징된`my-asp-app`ASP.NET 앱의 이름을 지정 합니다 (예: `contoso-container-registry`).

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   더 많은 앱 샘플 및 관련 dockerfiles를 보려면 [추가 컨테이너 샘플](../samples.md)을 참조 하세요.

2. 컨테이너 레지스트리에 앱을 게시 한 후 다음 단계는 Azure Kubernetes Service를 사용 하 여 만드는 Kubernetes 클러스터에 앱을 배포 하는 것입니다.

   > [!div class="nextstepaction"]
   > [Kubernetes 클러스터 만들기](https://docs.microsoft.com/azure/aks/windows-container-cli)
