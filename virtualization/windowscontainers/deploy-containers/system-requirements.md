---
title: Windows의 컨테이너 요구 사항
description: Windows의 컨테이너 요구 사항입니다.
keywords: 메타데이터, 컨테이너
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 74f501e5efab3a93e60c9d4797464cea283cdc0b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910503"
---
# <a name="windows-container-requirements"></a>Windows의 컨테이너 요구 사항

이 가이드는 Windows 컨테이너 호스트에 대 한 요구 사항을 나열 합니다.

## <a name="operating-system-requirements"></a>운영 체제 요구 사항

- Windows 컨테이너 기능은 Windows Server (반기 채널), Windows Server 2019, Windows Server 2016 및 Windows 10 Professional 및 Enterprise Edition (버전 1607 이상)에서 사용할 수 있습니다.
- Hyper-v 격리를 실행 하려면 hyper-v 역할을 설치 해야 합니다.
- Windows Server 컨테이너 호스트에서는 Windows가 c:\에 설치되어야 합니다. Hyper-v 격리 된 컨테이너만 배포 되는 경우에는이 제한이 적용 되지 않습니다.

## <a name="virtualized-container-hosts"></a>가상화 된 컨테이너 호스트

Hyper-v 가상 머신에서 Windows 컨테이너 호스트를 실행 하 고 Hyper-v 격리를 호스트 하는 경우에는 중첩 된 가상화를 사용 하도록 설정 해야 합니다. 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- Windows Server (반기 채널), Windows Server 2019, Windows Server 2016 또는 호스트 시스템의 Windows 10 가상 컴퓨터에서 및 Windows Server (전체 또는 서버 코어)가 있습니다.
- Intel VT-x가 포함된 프로세서.(이 기능은 현재 Intel 프로세서에 대해서만 사용할 수 있습니다)
- 컨테이너 호스트 VM에는 두 개 이상의 가상 프로세서도 필요 합니다.

### <a name="memory-requirements"></a>메모리 요구 사항

컨테이너에 대해 사용 가능한 메모리에 대한 제한은 [리소스 컨트롤](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)을 통해 또는 컨테이너 호스트를 오버로드하여 구성할 수 있습니다.  컨테이너를 시작 하 고 기본 명령 (ipconfig, dir 등)을 실행 하는 데 필요한 최소 메모리는 다음과 같습니다.

>[!NOTE]
>이러한 값은 컨테이너에서 실행 되는 응용 프로그램의 요구 사항 또는 컨테이너 간의 리소스 공유를 고려 하지 않습니다.  예를 들어 512MB의 사용 가능한 메모리가 있는 호스트는 Server Core 컨테이너가 리소스를 공유하므로 Hyper-V에서 여러 개의 Server Core 컨테이너를 실행할 수 있습니다.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Base image  | Windows Server 컨테이너 | Hyper-V 격리    |
| ----------- | ------------------------ | -------------------- |
| Nano 서버 | 40 M B                     | 130 MB + 1gb 페이지 파일 |
| Server Core | 50MB                     | 325 MB + 1gb 페이지 파일 |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server(반기 채널)

| Base image  | Windows Server 컨테이너 | Hyper-V 격리    |
| ----------- | ------------------------ | -------------------- |
| Nano 서버 | 30mb                     | 110 MB + 1gb 페이지 파일 |
| Server Core | 45 M B                     | 360 MB + 1gb 페이지 파일 |

## <a name="see-also"></a>참고 항목

[온-프레미스 시나리오에서 Windows 컨테이너 및 Docker에 대 한 지원 정책](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)