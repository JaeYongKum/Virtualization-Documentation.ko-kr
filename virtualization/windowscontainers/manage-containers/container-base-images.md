---
title: Windows 컨테이너 기본 이미지
description: Windows 컨테이너 기본 이미지와 사용 시점에 대 한 개요입니다.
keywords: Docker, 컨테이너, 해시
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: f5dcaf4958828b1bcf31a96e5fb70eda0508eb96
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161750"
---
# <a name="container-base-images"></a>컨테이너 기본 이미지

Windows는 사용자가 빌드할 수 있는 네 개의 컨테이너 기본 이미지를 제공 합니다. 각 기본 이미지는 Windows OS의 다른 버전 이며 디스크에 다른 공간이 있으므로 다른 크기의 Windows API 집합을 전달 합니다.

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>기존 .NET framework 응용 프로그램을 지원 합니다.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano 서버</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>.NET 핵심 응용 프로그램을 기반으로 합니다.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>전체 Windows API 집합을 제공 합니다.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IoT Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>용도-IoT 애플리케이션을 위한 빌드입니다.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>이미지 검색

모든 Windows 컨테이너 기본 이미지는 [Docker 허브](https://hub.docker.com/_/microsoft-windows-base-os-images)를 통해 검색할 수 있습니다. Windows 컨테이너 기본 이미지 자체는 [mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), 즉 Mcr (Microsoft 컨테이너 레지스트리)에서 제공 됩니다. Windows 컨테이너 기본 이미지에 대 한 pull 명령이 다음과 같이 표시 되는 이유입니다.

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR에는 고유한 카탈로그 환경이 없으며, Docker 허브와 같은 기존 카탈로그를 지원 하기 위한 것입니다. Azure의 전역 발자국으로 Azure CDN과 함께 사용 하는 경우 MCR은 일관성 있고 빠른 이미지 끌어오기 환경을 제공 합니다. Azure 사용자는 Azure에서 작업 부하를 실행 하 고, 네트워크에서 성능이 개선 되는 것을 비롯 하 여 MCR (Microsoft 컨테이너 이미지용 원본), Azure Marketplace, 제공 하는 Azure의 서비스 수 확장에 긴밀 하 게 통합 컨테이너를 배포 패키지 형식으로 지정 합니다.

## <a name="choosing-a-base-image"></a>기본 이미지 선택

어떤 방법으로 빌드에 적합 한 기본 이미지를 선택 하나요? 대부분의 사용자 `Windows Server Core` `Nanoserver` 에 게 가장 적절 한 이미지를 사용 하는 것입니다.

### <a name="guidelines"></a>지침

 원하는 이미지를 자유롭게 대상으로 지정할 수 있는 반면, 다음과 같은 몇 가지 지침을 참조 하 여 선택할 수 있습니다.

- **응용 프로그램에 전체 .NET framework가 필요 한가요?** 이 질문에 대 한 답변이 예 인 경우 목표 `Windows Server Core`를 세워야 합니다.
- **.NET Core를 기반으로 하는 Windows 앱을 빌드 하 고 있나요?** 이 질문에 대 한 답변이 예 인 경우 목표 `Nanoserver`를 세워야 합니다.
- **IoT 응용 프로그램을 작성 하 고 있나요?** 이 질문에 대 한 답변이 예 인 경우 목표 `IoT Core`를 세워야 합니다.
- **Windows Server Core 컨테이너 이미지에 앱에 필요한 종속성이 누락 되어 있습니까?** 이 질문에 대 한 대답이 예 인 경우 목표 `Windows`를 시도해 야 합니다. 이 이미지는 다른 기본 이미지 보다 훨씬 크기는 하지만, 많은 핵심 Windows 라이브러리 (예: GDI 라이브러리)를 전달 합니다.

> [!TIP]
> 대부분의 Windows 사용자는 .NET에 종속성이 있는 응용 프로그램을 containerize 합니다. 여기에 설명 된 4 개의 기본 이미지 외에도 Microsoft는 [.net framework](https://hub.docker.com/_/microsoft-dotnet-framework) 이미지 및 [ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) 이미지와 같은 인기 microsoft 프레임 워크로 미리 구성 된 여러 Windows 컨테이너 이미지를 게시 합니다.

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core vs Nanoserver

`Windows Server Core` `Nanoserver` 가장 일반적으로 대상으로 하는 기본 이미지입니다. 이러한 이미지 간의 주요 차이점은 Nanoserver에 매우 적은 API 표면이 있다는 것입니다. PowerShell, WMI 및 Windows 서비스 스택이 Nanoserver 이미지에서 존재 하지 않습니다.

Nanoserver는 .NET core 또는 다른 최신 오픈 소스 프레임 워크에 종속성이 있는 앱을 실행 하는 데 충분 한 API 표면을 제공 하도록 작성 되었습니다. 더 작은 APi 표면에 대 한 단점은, Nanoserver 이미지의 디스크 공간이 그 나머지 Windows 기본 이미지 보다 훨씬 더 작습니다. 적합하다고 판단될 경우 언제든지 Nano 서버 위에 계층을 추가할 수 있다는 점을 기억하세요. 관련 예제는 [NET Core Nano 서버 Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)을 참조하세요.
