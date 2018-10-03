---
title: Windows의 컨테이너 요구 사항
description: Windows의 컨테이너 요구 사항입니다.
keywords: 메타데이터, 컨테이너
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 8ff9373bd943c360079679a7e41256c24aa21aa8
ms.sourcegitcommit: d69ed13d505e96f514f456cdae0f93dab4fd3746
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/03/2018
ms.locfileid: "4340871"
---
# <a name="windows-container-requirements"></a>Windows의 컨테이너 요구 사항

이 가이드는 Windows 컨테이너 호스트에 대한 요구 사항을 나열합니다.

## <a name="os-requirements"></a>OS 요구 사항

- Windows 컨테이너 기능은 Windows Server 2016에서 사용할 수 있습니다 (코어 및 데스크톱 경험 포함), Windows 10 Professional 및 Enterprise (Anniversary Edition) 이상.
- Hyper-V 컨테이너를 실행하려면 Hyper-V 역할을 설치해야 합니다.
- Windows Server 컨테이너 호스트에서는 Windows가 c:\에 설치되어야 합니다. Hyper-V 컨테이너만 배포할 경우 이 제한이 적용되지 않습니다.

## <a name="virtualized-container-hosts"></a>가상화된 컨테이너 호스트

Windows 컨테이너 호스트가 Hyper-V 가상 컴퓨터에서 실행되고 Hyper-V 호스트 컨테이너를 호스팅할 경우 중첩된 가상화를 사용해야 합니다. 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- Windows Server 버전 1803, Windows Server (Full, Core) 및 Windows Server 버전 1709, Windows Server 2016 또는 Windows 10 호스트 시스템에 가상 컴퓨터에서 Windows Server 2019 합니다.
- Intel VT-x가 포함된 프로세서.(이 기능은 현재 Intel 프로세서에 대해서만 사용할 수 있습니다)
- 또한 컨테이너 호스트 VM에는 적어도 2개의 가상 프로세서가 필요합니다.

## <a name="supported-base-images"></a>지원되는 기본 이미지

Windows 컨테이너는 두 컨테이너 기본 이미지(Windows Server Core 및 Nano 서버)와 함께 제공됩니다. 일부 구성은 두 개의 OS 이미지를 모두 지원하지 않습니다. 이 테이블은 지원되는 구성을 자세히 설명합니다.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>호스트 운영 체제</center></th>
<th><center>Windows Server 컨테이너</center></th>
<th><center>Hyper-V 컨테이너</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 / 2019 (Standard 또는 Datacenter)</center></td>
<td><center>Server Core/Nano 서버</center></td>
<td><center>Server Core/Nano 서버</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano 서버</center></td>
<td><center>Server Core/Nano 서버</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro/Enterprise</center></td>
<td><center>사용할 수 없음</center></td>
<td><center>Server Core/Nano 서버</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">Windows Server 버전 1709부터 Nano 서버는 더 이상 컨테이너 호스트로 사용할 수 없습니다.</span>


### <a name="memory-requirements"></a>메모리 요구 사항
컨테이너에 대해 사용 가능한 메모리에 대한 제한은 [리소스 컨트롤](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)을 통해 또는 컨테이너 호스트를 오버로드하여 구성할 수 있습니다.  컨테이너를 시작하고 기본 명령(ipconfig, dir 등)을 실행하는 데 필요한 최소 메모리 용량은 다음과 같습니다.  __이러한 값은 컨테이너 간 리소스 공유 또는 컨테이너에서 실행되는 응용 프로그램의 요구 사항을 고려하지 않습니다.  예를 들어 512MB의 사용 가능한 메모리가 있는 호스트는 Server Core 컨테이너가 리소스를 공유하므로 Hyper-V에서 여러 개의 Server Core 컨테이너를 실행할 수 있습니다.__

#### <a name="windows-server-2016"></a>WindowsServer 2016
| 기본 이미지  | Windows Server 컨테이너 | Hyper-V 격리    |
| ----------- | ------------------------ | -------------------- |
| Nano 서버 | 40MB                     | 130MB + 1GB 페이지 파일 |
| Server Core | 50MB                     | 325MB + 1GB 페이지 파일 |

#### <a name="windows-server-version-1709"></a>Windows Server 버전 1709
| 기본 이미지  | Windows Server 컨테이너 | Hyper-V 격리    |
| ----------- | ------------------------ | -------------------- |
| Nano 서버 | 30MB                     | 110MB + 1GB 페이지 파일 |
| Server Core | 45MB                     | 360MB + 1GB 페이지 파일 |


### <a name="nano-server-vs-windows-server-core"></a>Nano 서버와 Windows Server Core 비교

Windows Server Core와 Nano 서버 중에 무엇을 선택해야 할까요? 개발자는 무엇이든 자유롭게 빌드할 수 있지만, 응용 프로그램이 .NET Framework와 완벽하게 호환되기를 원한다면 [Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/)를 사용해야 합니다. 반면, 응용 프로그램이 클라우드용으로 빌드되고 .NET Core를 사용하는 경우 [Nano 서버](https://hub.docker.com/r/microsoft/nanoserver/)를 사용해야 합니다. Nano 서버는 설치 공간을 최대한 줄일 목적으로 개발되었기 때문에 필수적이지 않은 여러 라이브러리가 제거되었습니다. Nano 서버 위에 빌드하려고 생각 중이라면 다음 사항을 염두에 두어야 합니다.

- 서비스 스택 제거됨
- .NET Core 미포함([.NET Core Nano 서버 이미지](https://hub.docker.com/r/microsoft/dotnet/) 사용 가능)
- PowerShell 제거됨
- WMI 제거됨
- Windows Server 버전 1709부터 응용 프로그램이 사용자 컨텍스트로 실행되므로 응용 프로그램 관리자 권한이 필요한 명령이 실패하게 됩니다. --user 플래그(예: docker run --user ContainerAdministrator)를 통해 컨테이너 관리자 계정을 지정할 수 있습니다. 하지만 향후에는 NanoServer에서 관리자 계정을 완전히 제거하려고 합니다.

이러한 것들이 가장 큰 차이점이며 그 외에도 다른 차이점이 더 있습니다. 여기서는 다루지 않았지만 제외된 다른 구성 요소가 더 있습니다. 적합하다고 판단될 경우 언제든지 Nano 서버 위에 계층을 추가할 수 있다는 점을 기억하세요. 관련 예제는 [NET Core Nano 서버 Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)을 참조하세요.

