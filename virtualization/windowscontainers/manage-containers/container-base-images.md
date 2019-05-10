---
title: Windows 컨테이너 기본 이미지 기록
description: SHA256 계층 해시과 함께 릴리스된 Windows 컨테이너 이미지 목록
keywords: Docker, 컨테이너, 해시
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 1b8bb433781e00bb8a435f14751d180ec52dec30
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621053"
---
# <a name="windows-container-base-image-history"></a>Windows 컨테이너 기본 이미지 기록

모든 Windows 컨테이너는 Microsoft가 제공하는 기본 OS를 기반으로 구축됩니다. 컨테이너가 구축된 기반 Windows 버전이 확실하지 않을 경우 `docker inspect <tag>`를 실행하고 최상위 행 또는 두 번째 행을 아래 차트와 일치하는지 확인합니다.

예를 들어 `docker inspect microsoft/windowsservercore:10.0.14393.447`을 실행하면 다음이 표시됩니다.

```
...
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67",
        "sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15"
    ]
}
```

Microsoft에서 제공하는 이미지에는 두 계층이 있습니다. 맨 위 계층은 상수이며 기존 Windows Server 릴리스를 나타내고, 두 번째 계층은 포함된 최신 누적 업데이트에 따라 달라집니다.

각 버전에서 변경된 내용을 찾으려면 [Windows 10 및 Windows Server 2016 업데이트 기록](https://support.microsoft.com/help/12387/windows-10-update-history)에서 기술 자료를 찾아보세요.


## <a name="tools-to-simplify-this-process"></a>이 프로세스를 간소화하기 위한 도구

Stefan Scherer는 전체 컨테이너를 다운로드하지 않고 버전을 파악하고 이미지 매니페스트를 읽을 수 있는 도구를 만들었습니다. 자세한 내용은 Stefan Scherer의 [블로그](https://stefanscherer.github.io/winspector/) 및 [GitHub](https://github.com/StefanScherer/winspector) 리포지토리를 확인하세요.


## <a name="image-versions"></a>이미지 버전

<table>
    <tr>
        <th>Windows 버전</th>
        <th>microsoft/windowsservercore</th>
        <th>microsoft/nanoserver</th>
    </tr>
    <tr>
        <td>10.0.14393.206</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063</td>
    </tr>
    <tr>
        <td>10.0.14393.321</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:cc6b0a07c696c3679af48ab4968de1b42d35e568f3d1d72df21f0acb52592e0b</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:2c195a33d84d936c7b8542a8d9890a2a550e7558e6ac73131b130e5730b9a3a5</td>
    </tr>
    <tr>
        <td>10.0.14393.447</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:c8606bedb07a714a6724b8f88ce85b71eaf5a1c80b4c226e069aa3ccbbe69154</td>
    </tr>
    <tr>
        <td>10.0.14393.576</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:de57d9086f9a337bb084b78f5f37be4c8f1796f56a1cd3ec8d8d1c9c77eb693c</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:0d812bf7a7032db75770c3d5b92c0ac9390ca4a9efa0d90ba2f55ccb16515381</td>
    </tr>
    <tr>
        <td>10.0.14393.693</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:c28d44287ce521eac86e0296c7677f5d8ca1e86d1e45e7618ec900da08c95df3</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:dd33c5d8d8b3c230886132c328a7801547f13de1dac9a629e2739164a285b3ab</td>
    </tr>
</table>

