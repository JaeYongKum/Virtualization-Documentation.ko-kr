---
title: Windows 컨테이너 기본 이미지
description: Windows 컨테이너 기본 이미지 및 사용 시기에 대 한 개요입니다.
keywords: Docker, 컨테이너, 해시
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909783"
---
# <a name="container-base-images"></a>컨테이너 기본 이미지

Windows는 사용자가 빌드할 수 있는 4 개의 컨테이너 기본 이미지를 제공 합니다. 각 기본 이미지는 Windows OS의 다른 버전으로, 다른 디스크 공간을 차지 하 고 다른 용량의 Windows API 집합을 사용 합니다.

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>Windows Server Core 
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none"></h3>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>.NET Core 응용 프로그램을 위해 작성 되었습니다.</p>
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
                    </div>Windows IoT Core 
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none"></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>목적-IoT 응용 프로그램용으로 작성 되었습니다.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>이미지 검색

모든 Windows 컨테이너 기본 이미지는 [Docker 허브](https://hub.docker.com/_/microsoft-windows-base-os-images)를 통해 검색할 수 있습니다. Windows 컨테이너 기본 이미지 자체는 [mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), Mcr (microsoft Container Registry)에서 제공 됩니다. Windows 컨테이너 기본 이미지에 대 한 pull 명령이 다음과 같이 표시 됩니다.

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR에는 자체 카탈로그 환경이 없으며 Docker 허브와 같은 기존 카탈로그를 지원할 수 있습니다. Azure의 전 세계 공간에 감사를 보내고 Azure CDN와 결합 하 여 MCR은 일관적이 고 빠른 이미지 끌어오기 환경을 제공 합니다. Azure에서 해당 워크 로드를 실행 하는 azure 고객은 네트워크 성능 향상 및 MCR (Microsoft 컨테이너 이미지의 원본 Azure Marketplace)과의 긴밀 한 통합 뿐만 아니라 Azure에서 제공 하는 확장 된 서비스 수에 대 한 혜택을 제공 합니다. 배포 패키지 형식으로 컨테이너를 배포 합니다.

## <a name="choosing-a-base-image"></a>기본 이미지 선택

빌드할 올바른 기본 이미지를 선택 하는 방법 대부분의 사용자에 대해 `Windows Server Core` 및 `Nanoserver`는 사용할 가장 적합 한 이미지입니다.

### <a name="guidelines"></a>지침

 원하는 이미지를 자유롭게 대상으로 할 수 있지만, 다음은 사용자의 선택에 도움이 되는 몇 가지 지침입니다.

- **응용 프로그램에 전체 .NET framework가 필요 한가요?** 이 질문에 대 한 대답이 예 인 경우 `Windows Server Core`대상으로 지정 해야 합니다.
- **.NET Core를 기반으로 Windows 앱을 빌드하고 있나요?** 이 질문에 대 한 대답이 예 인 경우 `Nanoserver`대상으로 지정 해야 합니다.
- **IoT 응용 프로그램을 빌드하고 있나요?** 이 질문에 대 한 대답이 예 인 경우 `IoT Core`대상으로 지정 해야 합니다.
- **Windows Server Core 컨테이너 이미지에 앱에 필요한 종속성이 누락 되었나요?** 이 질문에 대 한 대답이 예 인 경우 `Windows`를 대상으로 지정 해야 합니다. 이 이미지는 다른 기본 이미지 보다 훨씬 크지만 많은 핵심 Windows 라이브러리 (예: GDI 라이브러리)를 전달 합니다.
- **Windows 참가자 인가요?** 그렇다면, 참가자 버전의 이미지를 사용 하는 것을 고려해 야 합니다. 아래의 "Windows 참가자를 위한 기본 이미지"를 참조 하십시오.

> [!TIP]
> 많은 Windows 사용자가 .NET에 대 한 종속성이 있는 응용 프로그램을 컨테이너 화 하려고 합니다. 여기에 설명 된 4 개의 기본 이미지 외에도 Microsoft는 [.net framework](https://hub.docker.com/_/microsoft-dotnet-framework) 이미지 및 [ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) 이미지와 같은 인기 있는 microsoft 프레임 워크로 미리 구성 된 여러 Windows 컨테이너 이미지를 게시 합니다.

### <a name="base-images-for-windows-insiders"></a>Windows 참가자에 대 한 기본 이미지

Microsoft에서는 각 컨테이너 기본 이미지의 "insider" 버전을 제공 합니다. 이러한 참가자 컨테이너 이미지는 컨테이너 이미지에서 최신의 가장 강력한 기능 개발을 수행 합니다. Windows 참가자 버전 (windows 참가자 또는 Windows Server Insider) 인 호스트를 실행 하는 경우 이러한 이미지를 사용 하는 것이 좋습니다. Insider images는 Docker 허브에서 사용할 수 있습니다.

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

자세한 내용은 [Windows 참가자 프로그램으로 컨테이너 사용](../deploy-containers/insider-overview.md) 을 참조 하세요.

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core vs Nanoserver

`Windows Server Core` 및 `Nanoserver`는 대상으로 하는 가장 일반적인 기본 이미지입니다. 이러한 이미지 간의 주요 차이점은 Nanoserver에 훨씬 더 작은 API 표면이 있다는 것입니다. PowerShell, WMI 및 Windows 서비스 스택은 Nanoserver 이미지에서 존재 하지 않습니다.

Nanoserver는 .NET core 또는 기타 최신 오픈 소스 프레임 워크에 대 한 종속성이 있는 앱을 실행 하기에 충분 한 API 표면을 제공 하도록 빌드 되었습니다. 더 작은 APi 화면에 대 한 단점은 Nanoserver 이미지는 나머지 Windows 기반 이미지 보다 디스크 공간이 훨씬 더 작습니다. 적합하다고 판단될 경우 언제든지 Nano 서버 위에 계층을 추가할 수 있다는 점을 기억하세요. 관련 예제는 [NET Core Nano 서버 Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)을 참조하세요.
