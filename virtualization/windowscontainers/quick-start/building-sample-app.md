---
title: Containerize .NET Core 앱
description: 컨테이너를 사용 하 여 샘플 .NET core 앱을 빌드하는 방법 배우기
keywords: Docker, 컨테이너
author: cwilhit
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: f5a51fd1211868195126f06d917c0bef6e496c3d
ms.sourcegitcommit: f3b6b470dd9cde8e8cac7b13e7e7d8bf2a39aa34
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/10/2019
ms.locfileid: "10077474"
---
# <a name="containerize-a-net-core-app"></a>Containerize .NET Core 앱


이 퀵 스타트에서는 간단한 .NET 핵심 응용 프로그램을 containerize 하는 방법에 대해 설명 합니다. 너는 할 거야:

> [!div class="checklist"]
> * GitHub에서 샘플 앱 원본 복제
> * 앱 원본을 사용 하 여 컨테이너 이미지를 만드는 dockerfile을 만듭니다.
> * 로컬 Docker 환경에서 containerized .NET core 앱 테스트

## <a name="before-you-begin"></a>시작하기 전에

이 퀵 스타트는 개발 환경이 이미 컨테이너 사용을 위해 구성 되었다고 가정 합니다. 컨테이너에 대해 구성 된 환경이 없는 경우 시작 하는 방법에 대 한 자세한 내용은 [Windows 10 빠른 시작](./quick-start-windows-10.md) 을 참조 하세요.

컴퓨터에 Git 소스 제어 시스템이 설치 되어 있어야 합니다. 사용할 수 있는 내용: [Git](https://git-scm.com/download)

## <a name="getting-started"></a>시작하기

모든 컨테이너 샘플 소스 코드는 이라는 `windows-container-samples`폴더의 [가상화 문서](https://github.com/MicrosoftDocs/Virtualization-Documentation) git 리포지토리 아래에 보관 됩니다. 이 git 리포지토리를 curent 작업 디렉터리로 복제 합니다.

```Powershell
git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
```

아래 `<directory where clone occured>\Virtualization-Documentation\windows-container-samples\asp-net-getting-started` 에 있는 샘플 디렉터리로 이동 하 여 Dockerfile을 만듭니다. [Dockerfile](https://docs.docker.com/engine/reference/builder/) 은 컨테이너를 구성 해야 하는 방법에 대 한 컨테이너 엔진에 지시 하는 지침 목록 (예: 메이크파일)입니다.

```Powershell
#Create the dockerfile for our project
New-Item -name dockerfile -type file
```

## <a name="write-the-dockerfile"></a>Dockerfile 작성

방금 만든 dockerfile을 열고 (원하는 텍스트 편집기를 사용 하 여) 다음 콘텐츠를 추가 합니다.

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

_여러 단계 빌드_를 수행 하는 dockerfile을 작성 했습니다. Dockerfile이 실행 되는 경우 .NET core 2.1 SDK를 사용 하 `build-env`여 샘플 앱을 빌드한 다음 출력 된 이진 파일을 .net core 2.1 runtime이 포함 된 다른 컨테이너에 복사 하 여 표시 되는 크기를 최소화 하도록 하는 임시 컨테이너입니다. 최종 컨테이너.

## <a name="run-the-app"></a>앱 실행

Dockerfile이 기록 된 경우 dockerfile에서 Docker를 가리켜 이미지를 작성 하도록 지시할 수 있습니다. 

>[!IMPORTANT]
>아래 실행 된 명령을 dockerfile이 있는 디렉터리에서 실행 해야 합니다.

```Powershell
docker build -t my-asp-app .
```

컨테이너를 실행 하려면 아래 명령을 실행 합니다.

```Powershell
docker run -d -p 5000:80 --name myapp my-asp-app
```

다음 명령을 dissect 하겠습니다.

* `-d` tun에서 ' 분리 됨 ' 컨테이너를 실행 하도록 지시 하 여 컨테이너 안의 콘솔에 콘솔이 연결 되지 않았음을 의미 합니다. 컨테이너는 백그라운드에서 실행 됩니다. 
* `-p 5000:80` 호스트의 포트 5000을 컨테이너의 포트 80에 매핑하여 Docker에 알립니다. 각 컨테이너는 고유한 IP 주소를 얻습니다. ASP .NET은 기본적으로 포트 80를 수신 합니다. 포트 매핑을 사용 하면 매핑된 포트에서 호스트의 IP 주소로 이동 하 고 Docker는 모든 트래픽을 컨테이너 내부의 대상 포트로 전달 합니다.
* `--name myapp` 런타임에 Docker에 의해 할당 된 contaienr ID를 조회 하는 대신이 컨테이너에 편리한 이름을 제공 하도록 Docker에 지시 합니다.
* `my-asp-app` 는 Docker를 실행할 이미지입니다. 이 컨테이너 이미지는 `docker build` 프로세스의 culmination으로 생성 됩니다.

웹 브라우저 웹 브라우저를 열고 containerized 응용 프로그램 `https://localhost:5000` 에서 마지막으로 이동 합니다.

>![](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>다음 단계

ASP.NET 웹 앱을 containerized 했습니다. 더 많은 앱 샘플 및 관련 dockerfiles를 보려면 아래 단추를 클릭 하세요.

> [!div class="nextstepaction"]
> [더 많은 컨테이너 샘플 확인](../samples.md)
