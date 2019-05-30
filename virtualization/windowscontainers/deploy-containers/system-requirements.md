---
title: Windows의 컨테이너 요구 사항
description: Windows의 컨테이너 요구 사항입니다.
keywords: 메타데이터, 컨테이너
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: d3df0631a8a61db16ad207f49163a7304c5db717
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681053"
---
# <a name="windows-container-requirements"></a>Windows의 컨테이너 요구 사항

이 가이드에는 Windows 컨테이너 호스트에 대 한 요구 사항이 나열 되어 있습니다.

## <a name="os-requirements"></a>OS 요구 사항

- Windows 컨테이너 기능은 windows Server 2016 (Core 및 데스크톱 환경), Windows 10 Professional 및 Enterprise (기념일 Edition) 이상 에서만 사용할 수 있습니다.
<<<<<<< HEAD
- Hyper-v 격리를 실행 하기 전에 Hyper-v 역할을 설치 해야 합니다.
- Windows Server 컨테이너 호스트에서는 Windows가 c:\에 설치되어야 합니다. Hyper-v 격리 컨테이너만 배포 하는 경우에는이 제한이 적용 되지 않습니다.
=======
- Hyper-v 격리를 사용 하 여 컨테이너를 실행 하기 전에 Hyper-v 역할을 설치 해야 합니다.
- Windows Server 컨테이너 호스트에서는 Windows가 c:\에 설치되어야 합니다. Hyper-V 컨테이너만 배포할 경우 이 제한이 적용되지 않습니다.
>>>>>>> 원본/마스터

## <a name="virtualized-container-hosts"></a>가상화 된 컨테이너 호스트

<<<<<<< 헤드 Windows 컨테이너 호스트가 Hyper-v 가상 머신에서 실행 되며 Hyper-v 격리를 호스트 하는 경우에는 중첩 가상화를 사용 해야 합니다. 중첩 가상화에는 다음과 같은 요구 사항이 있습니다. = = = = = = = Hyper-v 가상 머신에서 Windows 컨테이너 호스트를 실행 하는 경우에도 Hyper-v 격리를 사용 하 여 컨테이너를 호스팅 하는 경우에는 중첩 가상화를 사용 해야 합니다. 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.
>>>>>>> 원본/마스터

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- 호스트 시스템의 windows server 2019, Windows Server 버전 1803, Windows Server 버전 1709, windows Server 2016 또는 windows 10에서 가상 컴퓨터의 Windows Server (Full, Core)를 표시 합니다.
- Intel VT-x가 포함된 프로세서.(이 기능은 현재 Intel 프로세서에 대해서만 사용할 수 있습니다)
<<<<<<< HEAD
- 또한 컨테이너 호스트 VM에는 두 개 이상의 가상 프로세서가 필요 합니다.

## <a name="supported-base-images"></a>지원 되는 기본 이미지

<a name="windows-containers-are-offered-with-four-container-base-images-windows-server-core-nano-server-windows-and-iot-core-not-all-configurations-support-both-os-images-this-table-details-the-supported-configurations"></a>Windows 컨테이너는 Windows Server Core, Nano 서버, Windows, IoT Core 등 네 가지 컨테이너 기본 이미지와 함께 제공 됩니다. 일부 구성은 두 개의 OS 이미지를 모두 지원하지 않습니다. 이 테이블은 지원되는 구성을 자세히 설명합니다.
=======
- 또한 컨테이너 호스트 VM에는 적어도 2개의 가상 프로세서가 필요합니다.

## <a name="supported-base-images"></a>지원되는 기본 이미지

Windows 컨테이너는 Windows Server Core, Nano 서버, Windows, IoT Core 등 네 가지 컨테이너 기본 이미지와 함께 제공 됩니다. 일부 구성은 두 개의 OS 이미지를 모두 지원하지 않습니다. 이 테이블은 지원되는 구성을 자세히 설명합니다.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>호스트 운영 체제</center></th>
<th><center>Windows Server 컨테이너</center></th>
<th><center>Hyper-V 격리</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016/2019 (Standard 또는 Datacenter)</center></td>
<td><center>Server Core, Nano 서버, Windows</center></td>
<td><center>Server Core, Nano 서버, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano 서버</center></td>
<td><center>Server Core, Nano 서버, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro/Enterprise</center></td>
<td><center>창을<a href="#warn-2">**</a></center></td>
<td><center>Server Core, Nano 서버, Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Core</center></td>
<td><center>IoT Core</center></td>
<td><center>사용할 수 없음</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">* Windows Server부터 버전 1709 Nano 서버를 더 이상 컨테이너 호스트로 사용할 수 없습니다.</span>

> <span id="warn-2">* * Windows 10 10 월 2018 업데이트가 필요 하며,이를 통해 `--isolation=process` `docker run`컨테이너를 실행할 때 플래그를 사용 하 여 프로세스 격리를 직접 요청 합니다.</span>
>>>>>>> 원본/마스터

|호스트 운영 체제|Windows 컨테이너|Hyper-V 격리|
|---------------------|-----------------|-----------------|
|Windows Server 2016 또는 Windows Server 2019 (Standard 또는 Datacenter)|Server Core, Nano 서버, Windows|Server Core, Nano 서버, Windows|
|Nano Server|Nano Server|Server Core, Nano 서버, Windows|
|Windows 10 Pro 또는 Windows 10 Enterprise|사용할 수 없음|Server Core, Nano 서버, Windows|
|IoT Core|IoT Core|사용할 수 없음|

> [!WARNING]  
> Windows Server 버전 1709부터 Nano 서버는 더 이상 컨테이너 호스트로 사용할 수 없습니다.

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

### <a name="base-image-differences"></a>기본 이미지 차이점

어떤 방법으로 빌드에 적합 한 기본 이미지를 선택 하나요? 원하는 모든 항목을 사용 하 여 빌드할 수 있지만, 각 이미지에 대 한 일반적인 지침은 다음과 같습니다.

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): 응용 프로그램에 전체 .net framework가 필요한 경우이 이미지를 사용 하는 것이 가장 좋습니다.
- [Nano 서버](https://hub.docker.com/_/microsoft-windows-nanoserver): .net Core만 필요로 하는 응용 프로그램의 경우 Nano 서버는 훨씬 얇고 이미지를 제공 합니다.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): 응용 프로그램이 서버 Core 또는 Nano 서버 이미지 (예: GDI 라이브러리)에 없는 구성 요소 또는 .dll에 종속 되어 있는 것을 확인할 수 있습니다. 이 이미지는 Windows의 전체 종속성 집합을 전달 합니다.
- [Iot Core](https://hub.docker.com/_/microsoft-windows-iotcore):이 이미지는 [IoT 애플리케이션을](https://developer.microsoft.com/windows/iot)위해 작성 된 용도입니다. IoT Core host를 대상으로 하는 경우이 컨테이너 이미지를 사용 해야 합니다.

대부분의 사용자는 Windows Server Core 또는 Nano 서버가 가장 적절 한 이미지를 사용 하 게 됩니다. 다음은 Nano 서버 위에 빌드할 때 고려해 야 할 사항입니다.

- 서비스 스택 제거됨
- .NET Core 미포함([.NET Core Nano 서버 이미지](https://hub.docker.com/r/microsoft/dotnet/) 사용 가능)
- PowerShell 제거됨
- WMI 제거됨
- Windows Server 버전 1709부터 응용 프로그램이 사용자 컨텍스트로 실행되므로 응용 프로그램 관리자 권한이 필요한 명령이 실패하게 됩니다. 앞으로는 NanoServer에서 관리자 계정을 완전히 제거 하기 위해--user 플래그 (예: docker 실행--사용자 ContainerAdministrator)를 통해 컨테이너 관리자 계정을 지정할 수 있습니다.

이러한 것들이 가장 큰 차이점이며 그 외에도 다른 차이점이 더 있습니다. 여기서는 다루지 않았지만 제외된 다른 구성 요소가 더 있습니다. 적합하다고 판단될 경우 언제든지 Nano 서버 위에 계층을 추가할 수 있다는 점을 기억하세요. 관련 예제는 [NET Core Nano 서버 Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)을 참조하세요.