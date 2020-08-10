---
title: Windows의 컨테이너 요구 사항
description: Windows의 컨테이너 요구 사항입니다.
keywords: 메타데이터, 컨테이너
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 14147ac71b5c10b3d633b2ccf205fe66489e417d
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985077"
---
# <a name="windows-container-requirements"></a>Windows의 컨테이너 요구 사항

이 가이드에는 Windows 컨테이너 호스트에 대한 요구 사항이 나열되어 있습니다.

## <a name="operating-system-requirements"></a>운영 체제 요구 사항

- Windows 컨테이너 기능은 Windows Server(반기 채널), Windows Server 2019, Windows Server 2016, Windows 10 Professional 및 Enterprise Edition(버전 1607 이상)에서 사용할 수 있습니다.
- Hyper-V 격리를 실행하려면 Hyper-V 역할을 설치해야 합니다.
- Windows Server 컨테이너 호스트에서는 Windows가 c:\에 설치되어야 합니다. Hyper-V 격리 컨테이너만 배포할 경우 이 제한이 적용되지 않습니다.

## <a name="virtualized-container-hosts"></a>가상화된 컨테이너 호스트

Windows 컨테이너 호스트가 Hyper-V 가상 머신에서 실행되고 또한 Hyper-V 격리를 호스팅할 경우 중첩된 가상화를 사용해야 합니다. 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- 호스트 시스템의 Windows Server(반기 채널), Windows Server 2019, Windows Server 2016 또는 Windows 10과 가상 머신의 Windows Server(Full 또는 Server Core).
- Intel VT-x가 포함된 프로세서.(이 기능은 현재 Intel 프로세서에 대해서만 사용할 수 있습니다)
- 그리고 컨테이너 호스트 VM에는 적어도 2개의 가상 프로세서가 필요합니다.

### <a name="memory-requirements"></a>메모리 요구 사항

컨테이너에 대해 사용 가능한 메모리에 대한 제한은 [리소스 컨트롤](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)을 통해 또는 컨테이너 호스트를 오버로드하여 구성할 수 있습니다.  컨테이너를 시작하고 기본 명령(ipconfig, dir 등)을 실행하는 데 필요한 최소 메모리 용량은 다음과 같습니다.

>[!NOTE]
>이러한 값은 컨테이너 간 리소스 공유 또는 컨테이너에서 실행되는 애플리케이션의 요구 사항을 고려하지 않습니다.  예를 들어 512MB의 사용 가능한 메모리가 있는 호스트는 Server Core 컨테이너가 리소스를 공유하므로 Hyper-V에서 여러 개의 Server Core 컨테이너를 실행할 수 있습니다.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Base image  | Windows Server 컨테이너 | Hyper-V 격리    |
| ----------- | ------------------------ | -------------------- |
| Nano 서버 | 40MB                     | 130MB + 1GB 페이지 파일 |
| Server Core | 50MB                     | 325MB + 1GB 페이지 파일 |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server(반기 채널)

| Base image  | Windows Server 컨테이너 | Hyper-V 격리    |
| ----------- | ------------------------ | -------------------- |
| Nano 서버 | 30MB                     | 110MB + 1GB 페이지 파일 |
| Server Core | 45MB                     | 360MB + 1GB 페이지 파일 |

## <a name="see-also"></a>참고 항목

[온-프레미스 시나리오에서 Windows 컨테이너 및 Docker에 대한 지원 정책](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)