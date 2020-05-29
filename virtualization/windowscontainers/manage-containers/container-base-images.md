---
title: Windows 컨테이너 기본 이미지
description: Windows 컨테이너 기본 이미지 및 사용 시기에 대한 개요입니다.
keywords: Docker, 컨테이너, 해시
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 9884cc0ae2d2f398d2dc2fb1997a70493a6de6c0
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2020
ms.locfileid: "76764175"
---
# <a name="container-base-images"></a>컨테이너 기본 이미지

Windows는 사용자가 빌드할 수 있는 4가지 컨테이너 기본 이미지를 제공합니다. 각 기본 이미지는 서로 다른 Windows OS용으로 제공되며, 차지하는 디스크 공간도 다르고 제공하는 Windows API 세트의 양도 다릅니다.

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
                        <p>기존의 .NET Framework 애플리케이션을 지원합니다.</p>
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
                        <p>.NET Core 애플리케이션을 위해 빌드되었습니다.</p>
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
                        <p>전체 Windows API 세트를 제공합니다.</p>
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
                        <p>IoT 애플리케이션을 위해 특별히 개발되었습니다.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>이미지 검색

모든 Windows 컨테이너 기본 이미지는 [Docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images)를 통해 검색할 수 있습니다. Windows 컨테이너 기본 이미지 자체는 MCR(Microsoft Container Registry)([mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/))에 제공됩니다. 이러한 이유로 Windows 컨테이너 기본 이미지에 대한 pull 명령은 다음과 같은 형식입니다.

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR는 자체 카탈로그 환경이 없으며 Docker Hub 같은 기존 카탈로그를 지원하는 용도로 사용됩니다. 전 세계에 Azure 인프라가 있고 Azure CDN과 긴밀하게 통합되어 있기 때문에 MCR은 일관적이고 신속한 이미지 끌어오기 환경을 제공합니다. Azure에서 워크로드를 실행하는 Azure 고객은 네트워크 성능이 향상될 뿐 아니라 MCR(Microsoft 컨테이너 이미지의 소스)과의 긴밀한 통합, Azure Marketplace, 컨테이너를 배포 패키지 형식으로 제공하는 서비스(그 수가 점점 늘어나고 있음)의 이점을 활용할 수 있습니다.

## <a name="choosing-a-base-image"></a>기본 이미지 선택

빌드할 적당한 기본 이미지를 선택하려면 어떻게 해야 할까요? 대부분의 사용자에게는 `Windows Server Core` 및 `Nanoserver`가 가장 적당한 이미지입니다.

### <a name="guidelines"></a>지침

 원하는 대상 이미지를 자유롭게 선택할 수 있으며, 다음은 적당한 이미지 선택을 도와주기 위한 지침입니다.

- **애플리케이션에 전체 .NET Framework가 필요한가요?** 이 질문에 대한 대답이 예라면 `Windows Server Core`를 대상으로 삼아야 합니다.
- **.NET Core를 기반으로 Windows 앱을 빌드하나요?** 이 질문에 대한 대답이 예라면 `Nanoserver`를 대상으로 삼아야 합니다.
- **IoT 애플리케이션을 빌드하나요?** 이 질문에 대한 대답이 예라면 `IoT Core`를 대상으로 삼아야 합니다.
- **앱에 필요한 종속성이 Windows Server Core 컨테이너 이미지에는 없나요?** 이 질문에 대한 대답이 예라면 `Windows`를 대상으로 삼아야 합니다. 이 이미지는 다른 기본 이미지보다 훨씬 크지만, 여러 핵심 Windows 라이브러리(예: GDI 라이브러리)를 제공합니다.
- **Windows 참가자인가요?** 그렇다면 참가자 버전의 이미지를 사용하는 방안을 고려해야 합니다. 아래의 "Windows 참가자를 위한 기본 이미지"를 참조하세요.

> [!TIP]
> 많은 Windows 사용자들은 .NET에 종속된 애플리케이션을 컨테이너화하려고 합니다. 여기에 설명된 4가지 기본 이미지 외에도, Microsoft는 [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) 이미지나 [ASP .NET](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) 이미지처럼 인기 있는 Microsoft 프레임워크로 미리 구성된 여러 Windows 컨테이너 이미지를 게시합니다.

### <a name="base-images-for-windows-insiders"></a>Windows 참가자를 위한 기본 이미지

Microsoft에서는 각 컨테이너 기본 이미지의 "참가자" 버전을 제공합니다. 이러한 참가자 컨테이너 이미지는 컨테이너 이미지에서 가장 강력한 최신 기능 개발을 수행합니다. Windows 참가자 버전(Windows 참가자 또는 Windows Server Insider)인 호스트를 실행하는 경우 이러한 이미지를 사용하는 것이 좋습니다. 참가자 이미지는 Docker Hub에 제공됩니다.

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

자세한 내용은 [Windows 참가자 프로그램에서 컨테이너 사용](../deploy-containers/insider-overview.md)을 참조하세요.

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core와 Nanoserver의 차이점

`Windows Server Core` 및 `Nanoserver`는 대상으로 지정되는 가장 일반적인 기본 이미지입니다. 두 이미지의 주요 차이점은 Nanoserver의 API 화면이 훨씬 작다는 점입니다. PowerShell, WMI 및 Windows 서비스 스택은 Nanoserver 이미지에 없습니다.

Nanoserver는 .NET 코어 또는 다른 최신 오픈 소스 프레임워크에 종속된 앱을 실행하기에 부족하지 않은 API 화면을 제공하는 용도로 빌드되었습니다. API 화면이 더 작은 대신, Nanoserver 이미지는 나머지 Windows 기반 이미지보다 디스크 공간이 훨씬 더 작습니다. 적합하다고 판단될 경우 언제든지 Nano 서버 위에 계층을 추가할 수 있다는 점을 기억하세요. 관련 예제는 [NET Core Nano 서버 Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1909/amd64/Dockerfile)을 참조하세요.
