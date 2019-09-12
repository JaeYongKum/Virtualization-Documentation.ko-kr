---
title: Windows의 컨테이너 요구 사항
description: Windows의 컨테이너 요구 사항입니다.
keywords: 메타데이터, 컨테이너
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: df5d8e17d0d512f7f53fcacf6c2c2a2652f3e7c0
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107867"
---
# <a name="windows-container-requirements"></a>Windows의 컨테이너 요구 사항

이 가이드에는 Windows 컨테이너 호스트에 대 한 요구 사항이 나열 되어 있습니다.

## <a name="os-requirements"></a>OS 요구 사항

- Windows 컨테이너 기능은 windows Server 2016 (Core 및 데스크톱 환경), Windows 10 Professional 및 Enterprise (기념일 Edition) 이상 에서만 사용할 수 있습니다.
- Hyper-v 격리를 실행 하기 전에 Hyper-v 역할을 설치 해야 합니다.
- Windows Server 컨테이너 호스트에서는 Windows가 c:\에 설치되어야 합니다. Hyper-v 격리 컨테이너만 배포 하는 경우에는이 제한이 적용 되지 않습니다.

## <a name="virtualized-container-hosts"></a>가상화 된 컨테이너 호스트

Hyper-v 가상 머신에서 Windows 컨테이너 호스트를 실행 하 고 Hyper-v 격리를 호스트 하는 경우에는 중첩 가상화를 사용 해야 합니다. 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- 호스트 시스템의 windows server 2019, Windows Server 버전 1803, Windows Server 버전 1709, windows Server 2016 또는 windows 10에서 가상 컴퓨터의 Windows Server (Full, Core)를 표시 합니다.
- Intel VT-x가 포함된 프로세서.(이 기능은 현재 Intel 프로세서에 대해서만 사용할 수 있습니다)
- 또한 컨테이너 호스트 VM에는 두 개 이상의 가상 프로세서가 필요 합니다.

### <a name="memory-requirements"></a>메모리 요구 사항

컨테이너에 대해 사용 가능한 메모리에 대한 제한은 [리소스 컨트롤](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)을 통해 또는 컨테이너 호스트를 오버로드하여 구성할 수 있습니다.  컨테이너를 시작 하 고 기본 명령 (ipconfig, dir 등)을 실행 하는 데 필요한 최소 메모리 양이 아래에 나열 되어 있습니다.

>[!NOTE]
>이러한 값은 컨테이너에서 실행 되는 응용 프로그램의 컨테이너 또는 요구 사항 간 리소스 공유를 고려 하지 않습니다.  예를 들어 512MB의 사용 가능한 메모리가 있는 호스트는 Server Core 컨테이너가 리소스를 공유하므로 Hyper-V에서 여러 개의 Server Core 컨테이너를 실행할 수 있습니다.

#### <a name="windows-server-2016"></a>WindowsServer 2016

| 기본 이미지  | Windows Server 컨테이너 | Hyper-V 격리    |
| ----------- | ------------------------ | -------------------- |
| Nano 서버 | 40 MB                     | 130 MB + 1 GB 페이지 파일 |
| Server Core | 50 MB                     | 325 MB + 1 GB 페이지 파일 |

#### <a name="windows-server-version-1709"></a>Windows Server 버전 1709

| 기본 이미지  | Windows Server 컨테이너 | Hyper-V 격리    |
| ----------- | ------------------------ | -------------------- |
| Nano 서버 | 30mb                     | 110 MB + 1 GB 페이지 파일 |
| Server Core | 45 MB                     | 360 MB + 1 GB 페이지 파일 |
