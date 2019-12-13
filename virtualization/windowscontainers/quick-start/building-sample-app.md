---
title: .NET Core 앱 컨테이너 화
description: 컨테이너를 사용 하 여 샘플 .NET core 앱을 빌드하는 방법 알아보기
keywords: Docker, 컨테이너
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910143"
---
# <a name="containerize-a-net-core-app"></a>.NET Core 앱 컨테이너 화

이 항목에서는 [시작: 컨테이너에 대 한 Windows 준비](set-up-environment.md)에 설명 된 대로 환경을 설정한 후 [첫 번째 windows 컨테이너 실행](run-your-first-container.md)에 설명 된 대로 첫 번째 컨테이너를 실행 하는 방법에 대 한 기존 샘플 .net 앱을 Windows 컨테이너로 패키징하는 방법에 대해 설명 합니다.

또한 컴퓨터에 Git 소스 제어 시스템이 설치 되어 있어야 합니다. 설치 하려면 [Git](https://git-scm.com/download)를 방문 합니다.

## <a name="clone-the-sample-code-from-github"></a>GitHub에서 샘플 코드를 복제 합니다.

모든 컨테이너 샘플 소스 코드는 `windows-container-samples`라는 폴더에 있는 [가상화 문서](https://github.com/MicrosoftDocs/Virtualization-Documentation) git 리포지토리 (리포지토리로 비공식적으로 알려진)에 유지 됩니다.

1. PowerShell 세션을 열고이 리포지토리를 저장 하려는 폴더로 디렉터리를 변경 합니다. 다른 명령 프롬프트 창 유형도 작동 하지만 예제 명령은 PowerShell을 사용 합니다.
2. 현재 작업 디렉터리에 리포지토리를 복제 합니다.

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 다음 명령을 사용 하 여 `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` 아래에 있는 샘플 디렉터리로 이동 하 고 Dockerfile을 만듭니다.

   [Dockerfile](https://docs.docker.com/engine/reference/builder/) 은 메이크파일의와 같습니다. 컨테이너 엔진에 컨테이너 이미지를 빌드하는 방법을 알려 주는 명령 목록입니다.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Dockerfile 쓰기

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

줄 단위로 분할 하 고 각 지침의 용도를 설명 해 보겠습니다.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

첫 번째 선 그룹은 컨테이너를 빌드하는 데 사용할 기반이 되는 기본 이미지를 선언합니다. 로컬 시스템에 이 이미지가 없으면 docker가 자동으로 이미지를 가져오려고 시도합니다. `mcr.microsoft.com/dotnet/core/sdk:2.1`는 .NET core 2.1 SDK가 설치 된 상태로 제공 되므로 버전 2.1을 대상으로 하는 ASP .NET core 프로젝트를 빌드하는 작업의 작업을 진행 합니다. 다음 명령은 `/app`되도록 컨테이너의 작업 디렉터리를 변경 하므로이 컨텍스트 다음에 오는 모든 명령이이 컨텍스트에서 실행 됩니다.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

그런 다음이 지침은 .csproj 파일을 `build-env` 컨테이너의 `/app` 디렉터리로 복사 합니다. 이 파일을 복사한 후 .NET은이 파일에서 읽은 다음 프로젝트에 필요한 모든 종속성과 도구를 가져옵니다.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

.NET이 모든 종속성을 `build-env` 컨테이너로 끌어온 다음 명령은 모든 프로젝트 소스 파일을 컨테이너에 복사 합니다. 그런 다음 .NET에 릴리스 구성을 사용 하 여 응용 프로그램을 게시 하 고에서 출력 경로를 지정 합니다.

컴파일이 성공 합니다. 이제 최종 이미지를 빌드해야 합니다. 

> [!TIP]
> 이 빠른 시작은 원본에서 .NET core 프로젝트를 빌드합니다. 컨테이너 이미지를 빌드할 때 컨테이너 이미지에 프로덕션 페이로드 및 해당 종속성 _만_ 포함 하는 것이 좋습니다. .Net core 런타임이 필요 하기 때문에 .NET core SDK를 최종 이미지에 포함 하지 않으려는 경우, 앱을 빌드하는 `build-env` SDK와 함께 패키지 된 임시 컨테이너를 사용 하도록 dockerfile을 작성 합니다.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

응용 프로그램이 ASP.NET 이므로이 런타임이 포함 된 이미지를 지정 합니다. 그런 다음 임시 컨테이너의 출력 디렉터리에 들어 있는 모든 파일을 최종 컨테이너에 복사합니다. 컨테이너가 시작 될 때 진입점으로 새 앱을 실행 하도록 컨테이너를 구성 합니다.

_여러 단계 빌드_를 수행 하기 위해 dockerfile을 작성 했습니다. Dockerfile이 실행 되 면 .NET core 2.1 SDK를 사용 하 여 `build-env`임시 컨테이너를 사용 하 여 샘플 앱을 빌드한 다음, 최종 컨테이너의 크기를 최소화 하도록 .NET core 2.1 런타임만 포함 된 다른 컨테이너에 출력 된 이진 파일을 복사 합니다.

## <a name="build-and-run-the-app"></a>앱 빌드 및 실행

Dockerfile을 작성 한 후에는 Dockerfile에서 Docker를 가리키고 이미지를 빌드하고 실행 하도록 지시할 수 있습니다.

1. 명령 프롬프트 창에서 dockerfile이 있는 디렉터리로 이동한 후 [docker build](https://docs.docker.com/engine/reference/commandline/build/) 명령을 실행 하 여 dockerfile에서 컨테이너를 빌드합니다.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 새로 빌드된 컨테이너를 실행 하려면 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 명령을 실행 합니다.

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   다음 명령을 알아본 보겠습니다.

   * `-d`는 Docker 실행가 컨테이너 ' 분리 됨 '을 실행 하도록 지시 합니다. 즉, 콘솔은 콘솔에 연결 되지 않습니다. 컨테이너는 백그라운드에서 실행 됩니다. 
   * `-p 5000:80`는 호스트의 포트 5000를 컨테이너의 포트 80에 매핑하도록 Docker에 지시 합니다. 각 컨테이너는 고유의 IP 주소를 가져옵니다. ASP .NET은 기본적으로 포트 80에서 수신 대기 합니다. 포트 매핑을 사용 하면 매핑된 포트에서 호스트의 IP 주소로 이동 하 고 Docker는 컨테이너 내의 대상 포트로 모든 트래픽을 전달 합니다.
   * `--name myapp`는 docker에서 런타임에 할당 된 contaienr ID를 조회 하지 않고이 컨테이너에 쿼리 하는 데 편리한 이름을 지정 합니다.
   * `my-asp-app`는 Docker를 실행 하려는 이미지입니다. `docker build` 프로세스의 다룬로 생성 된 컨테이너 이미지입니다.

3. 다음 스크린샷에 표시 된 것 처럼 웹 브라우저 웹 브라우저를 열고 `http://localhost:5000`로 이동 하 여 컨테이너 화 된 응용 프로그램을 확인 합니다.

   >![ASP.NET Core 웹 페이지, 컨테이너의 localhost에서 실행](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>다음 단계

1. 다음 단계는 Azure Container Registry를 사용 하 여 개인 레지스트리에 컨테이너 화 된 ASP.NET 웹 앱을 게시 하는 것입니다. 이렇게 하면 조직에 배포할 수 있습니다.

   > [!div class="nextstepaction"]
   > [개인 컨테이너 레지스트리 만들기](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   [컨테이너 이미지를 레지스트리에 푸시하](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)는 섹션으로 이동 하는 경우 컨테이너 레지스트리와 함께 패키지 (`my-asp-app`) 할 ASP.NET 앱의 이름을 지정 합니다 (예: `contoso-container-registry`).

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   더 많은 앱 샘플 및 관련 dockerfiles를 보려면 [추가 컨테이너 샘플](../samples.md)을 참조 하세요.

2. 컨테이너 레지스트리에 앱을 게시 한 후 다음 단계는 Azure Kubernetes Service를 사용 하 여 만든 Kubernetes 클러스터에 앱을 배포 하는 것입니다.

   > [!div class="nextstepaction"]
   > [Kubernetes 클러스터 만들기](https://docs.microsoft.com/azure/aks/windows-container-cli)
