---
title: Windows Server 컨테이너 업데이트
description: Windows에서 빌드를 실행하고 다양한 버전 간에 컨테이너를 실행할 수 있는 방법
keywords: 메타데이터, 컨테이너, 버전
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 84413f27bfce66e7d259c05795a280ed34b582ab
ms.sourcegitcommit: 6f216408434a437da87a72d582500a4ca6c2679c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/21/2020
ms.locfileid: "80112696"
---
# <a name="update-windows-server-containers"></a>Windows Server 컨테이너 업데이트

매월 Windows Server 서비스의 일부로 업데이트된 Windows Server 기본 OS 컨테이너 이미지를 정기적으로 게시합니다. 이러한 업데이트를 사용하면 업데이트된 컨테이너 이미지 빌드를 자동화하거나 최신 버전을 가져와서 수동으로 업데이트할 수 있습니다. Windows Server 컨테이너에는 Windows Server와 같은 서비스 스택이 포함되지 않습니다. Windows Server에서와 같이 컨테이너 내에서 업데이트를 가져올 수 없습니다. 따라서 매월 업데이트를 사용하여 Windows Server 기본 OS 컨테이너 이미지를 다시 빌드하고 업데이트된 컨테이너 이미지를 게시합니다.

.NET 또는 IIS와 같은 다른 컨테이너 이미지는 업데이트된 기본 OS 컨테이너 이미지를 기반으로 하여 다시 빌드되고 매월 게시됩니다.

## <a name="how-to-get-windows-server-container-updates"></a>Windows Server 컨테이너 업데이트를 가져오는 방법

Windows 서비스 주기에 맞춰 Windows Server 기본 OS 컨테이너 이미지를 업데이트합니다. 업데이트된 컨테이너 이미지는 매월 두 번째 화요일에 게시되며, 릴리스 월을 기반으로 하는 접두사 번호가 붙는 "B" 릴리스라고도 합니다. 예를 들어 2월 업데이트는 "2B"이고, 3월 업데이트는 "3B"라고 합니다. 이 월간 업데이트 이벤트는 새 보안 수정이 포함된 유일한 정기 릴리스입니다.

컨테이너 호스트 또는 "호스트"라고 하는 이러한 컨테이너를 호스팅하는 서버는 "B" 릴리스 이외의 추가 업데이트 이벤트 중에 서비스를 제공할 수 있습니다. Windows 업데이트 서비스 주기에 대한 자세한 내용은 [Windows 업데이트 서비스 주기](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376) 블로그 게시물을 참조하세요.

새 Windows Server 기본 OS 컨테이너 이미지는 매월 둘째 주 화요일 오전 10:00 PST 직후에 MCR(Microsoft Container Registry)에서 게시되며, 추천 태그는 최신 B 릴리스를 대상으로 합니다. 예를 들면 다음과 같습니다.

- ltsc2019[(LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc): docker pull mcr.microsoft.com/windows/servercore:ltsc2019
- 1909[(SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel): docker pull mcr.microsoft.com/windows/servercore:1909

MCR보다 Docker Hub에 더 익숙한 경우 [이 블로그 게시물](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/)에서 더 자세히 설명합니다.  

각 릴리스에 대해 각 컨테이너 이미지는 특정 컨테이너 이미지 수정 버전을 대상으로 하는 수정 번호 및 KB 문서 번호에 대한 두 개의 추가 태그를 사용하여 게시됩니다. 예:

- docker pull mcr.microsoft.com/windows/servercore:10.0.17763.1040
- docker pull mcr.microsoft.com/windows/servercore:1809-KB4546852

이러한 예제는 모두 2월 18일 보안 릴리스 업데이트를 통해 Windows Server 2019 Server Core 컨테이너 이미지를 끌어옵니다.  

Windows Server 기본 OS 컨테이너 이미지, 버전 및 해당 태그의 전체 목록은 Docker Hub에서 이 [Windows 기본 OS 컨테이너 이미지](https://hub.docker.com/_/microsoft-windows-base-os-images)를 참조하세요.

Microsoft에서 Azure Marketplace에 릴리스한 월간 서비스를 제공하는 Windows Server 이미지에도 미리 설치된 기본 OS 컨테이너 이미지가 제공됩니다. 자세한 내용은 [Windows Server Azure Marketplace 가격 책정 페이지](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice)에서 확인할 수 있습니다. 이러한 이미지는 일반적으로 "B" 릴리스 후 약 5일(작업일) 후에 업데이트됩니다.

Windows Server 이미지 및 버전의 전체 목록은 [Azure Marketplace 업데이트 기록의 Windows Server 릴리스](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history)를 참조하세요.

## <a name="host-and-container-version-compatibility"></a>호스트 및 컨테이너 버전 호환성

Windows 컨테이너에는 프로세스 격리 및 Hyper-V 격리라는 두 가지 유형의 격리 모드가 있습니다. 호스트 및 컨테이너 버전 호환성과 관련하여 Hyper-V 격리가 더 유연합니다. 자세한 내용은 [버전 호환성](version-compatibility.md) 및 [격리 모드](../manage-containers/hyperv-container.md)를 참조하세요. 이 섹션에서는 별도로 언급하지 않는 한 프로세스 격리 컨테이너에 대해 집중적으로 설명합니다.

호스트 및 컨테이너 이미지가 모두 지원되는 경우(Windows Server 버전 1809 이상) 컨테이너 호스트 또는 컨테이너 이미지를 월간 업데이트를 통해 업데이트하면 컨테이너를 정상적으로 시작하고 실행하기 위해 호스트 및 컨테이너 이미지 수정 버전이 일치할 필요가 없습니다.

그러나 2020년 2월 11일 월간 보안 업데이트 릴리스("2B"라고도 함) 및 이후 릴리스에서 Windows Server 컨테이너를 사용할 때 문제가 발생할 수 있습니다. 자세한 내용은 [이 Microsoft 지원 문서](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t)를 참조하세요. 이러한 문제는 애플리케이션의 보안을 보장하기 위해 사용자 모드와 커널 모드 간의 인터페이스가 필요한 보안 변경으로 인해 발생했습니다. 프로세스 격리 컨테이너에서 컨테이너 호스트와 커널 모드를 공유하므로 이러한 문제는 프로세스 격리 컨테이너에서만 발생합니다. 즉, 업데이트된 사용자 모드 구성 요소가 없는 컨테이너 이미지는 보안되지 않았으며 새 보안 커널 인터페이스와 호환되지 않습니다.

2020년 2월 18일에 수정 사항을 릴리스했습니다. 이 새 릴리스는 "새 기준"을 확정했습니다. 이 새 기준에서 따르는 규칙은 다음과 같습니다.

- 2B 이전의 호스트와 컨테이너의 조합은 모두 작동합니다.
- 2B 이후의 호스트와 컨테이너의 조합은 모두 작동합니다.
- 새 기준의 다른 쪽에 있는 호스트와 컨테이너의 조합은 모두 작동하지 않습니다. 예를 들어 3B 호스트와 1B 컨테이너는 작동하지 않습니다.

2020년 3월 월간 보안 업데이트 릴리스를 예로 들어 새 호환성 규칙이 작동하는 방식을 보여 드리겠습니다. 다음 표에서 2020년 3월 보안 업데이트 릴리스는 "3B", 2020년 2월 업데이트는 "2B" 및 2020년 1월 업데이트는 "1B"라고 합니다.

| 호스트 | 컨테이너 | 호환성 |
|---|---|---|
| 3B | 3B | 예 |
| 3B | 2B | 예 |
| 3B | 1B 또는 이전 | 아니요 |
| 2B | 3B | 예 |
| 2B | 2B | 예 |
| 2B | 1B 또는 이전 | 아니요 |
| 1B 또는 이전 | 3B | 아니요 |
| 1B 또는 이전 | 2B | 아니요 |
| 1B 또는 이전 | 1B 또는 이전 | 예 |

참고로, 다음 표에는 Windows Server 2016에서 최신 Windows Server 버전 1909 릴리스까지 여러 주요 OS 릴리스에 걸쳐 1B, 2B 및 3B 월간 보안 업데이트 릴리스가 포함된 기본 OS 컨테이너 이미지의 버전 번호가 나와 있습니다.

| Windows Server 버전(부동 태그) | 2020년 1월 14일 릴리스 버전 업데이트(1B)| 2020년 2월 18일 릴리스 버전 업데이트(2B) | 2020년 3월 10일 릴리스 버전 업데이트(3B) |
|---|---|---|---|
| Windows Server 2016(ltsc2016) | 10.0.14393.3443(KB4534271) | 10.0.14393.3506(KB4546850) | 10.0.14393.3568(KB4551573) |
| Windows Server 버전 1803(1803) | 10.0.17134.1246(KB4534293) | 10.0.17134.1305(KB4546851)  | 이 버전의 지원은 종료되었습니다. 자세한 내용은 [기본 이미지 서비스 수명 주기](base-image-lifecycle.md)를 참조하세요.|
| Windows Server 버전 1809(1809)| 10.0.17763.973(KB4534273) | 10.0.17763.1040(KB4546852) | 10.0.17763.1098(KB4538461) |
| Windows Server 2019(ltsc2019) | 10.0.17763.973(KB4534273) | 10.0.17763.1040(KB4546852) | 10.0.17763.1098(KB4538461) |
| Windows Server 버전 1903(1903) |10.0.18362.592(KB4528760) | 10.0.18362.658(KB4546853) | 10.0.18362.719(KB4540673) |
| Windows Server 버전 1909(1909) | 10.0.18363.592(KB4528760) | 10.0.18363.658(KB4546853) | 10.0.18363.719(KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>호스트 및 컨테이너 이미지의 불일치 문제 해결

시작하기 전에 [버전 호환성](version-compatibility.md)의 정보를 숙지해야 합니다. 이 정보는 패치 불일치로 인해 문제가 발생했는지 여부를 파악하는 데 도움이 됩니다. 일치하지 않는 패치를 원인으로 설정한 경우 이 섹션의 지침에 따라 문제를 해결할 수 있습니다.

### <a name="query-the-version-of-your-container-host"></a>컨테이너 호스트의 버전 쿼리

컨테이너 호스트에 액세스할 수 있으면 `ver` 명령을 실행하여 해당 OS 버전을 가져올 수 있습니다. 예를 들어 최신 2020년 2월 보안 업데이트 릴리스가 있는 Windows Server 2019를 실행하는 시스템에서 `ver`을 실행하는 경우 다음과 같이 표시됩니다.

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>2020년 2월 보안 업데이트 릴리스에는 Windows Server에 대한 대역 외 2B 릴리스가 없으므로 이 예제의 수정 번호는 1040이 아니라 1039로 표시됩니다. 컨테이너에 대한 대역 외 2B 릴리스만 있었으며, 수정 번호는 1040이었습니다.

컨테이너 호스트에 직접 액세스할 수 없는 경우 IT 관리자에게 문의하세요. 클라우드에서 실행하는 경우 클라우드 공급자의 웹 사이트에서 실행되는 컨테이너 호스트 OS 버전을 확인합니다. 예를 들어 AKS(Azure Kubernetes Service)를 사용하는 경우 [AKS 릴리스 정보](https://github.com/Azure/AKS/releases)에서 호스트 OS 버전을 찾을 수 있습니다.

### <a name="query-the-version-of-your-container-image"></a>컨테이너 이미지의 버전 쿼리

다음 지침에 따라 실행되는 컨테이너 버전을 확인합니다.

1. PowerShell에서 다음 cmdlet을 실행합니다.

    ```powershell
    docker images
    ```

    출력은 다음과 유사합니다.

     ```powershell
     REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
     mcr.microsoft.com/windows/servercore   ltsc2019            b456290f487c        4 weeks ago         4.84GB
     mcr.microsoft.com/windows              1809                58229ca44fa7        4 weeks ago         12GB
     mcr.microsoft.com/windows/nanoserver   1809                f519d4f3a868        4 weeks ago         251M

2. Run the `docker inspect` command for the Image ID of the container image that isn't working. This will tell you which version the container image is targeting.

   For example, let's say we `run docker inspect` for an ltsc 2019 container image:

   ```powershell
   docker inspect b456290f487c

       "Architecture": "amd64",

        "Os": "windows",

        "OsVersion": "10.0.17763.1039",

        "Size": 4841309825,

        "VirtualSize": 4841309825,
    ```

    이 예제에서는 컨테이너 OS 버전이 `10.0.17763.1039`로 표시됩니다.

    이미 컨테이너를 실행하고 있는 경우 컨테이너 자체 내에서 `ver` 명령을 실행하여 버전을 가져올 수도 있습니다. 예를 들어 최신 2020년 2월 보안 업데이트 릴리스가 있는 Windows Server 2019의 Server Core 컨테이너 이미지에서 `ver`을 실행하면 다음과 같이 표시됩니다.

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```
