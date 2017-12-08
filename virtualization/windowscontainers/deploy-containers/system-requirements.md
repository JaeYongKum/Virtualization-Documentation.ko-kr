---
title: "Windows의 컨테이너 요구 사항"
description: "Windows의 컨테이너 요구 사항입니다."
keywords: "메타데이터, 컨테이너"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 6ae690ff6592198bc16cbaf60489d3ed5aceeeb0
ms.sourcegitcommit: 64f5f8d838f72ea8e0e66a72eeb4ab78d982b715
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/22/2017
---
# <a name="windows-container-requirements"></a>Windows의 컨테이너 요구 사항

이 가이드는 Windows 컨테이너 호스트에 대한 요구 사항을 나열합니다.

## <a name="os-requirements"></a>OS 요구 사항

- Windows 컨테이너 기능은 Windows Server 빌드 1709, Windows Server 2016(코어 및 데스크톱 경험 포함), Windows 10 Professional 및 Enterprise(Anniversary Edition)에서만 사용할 수 있습니다.
- Hyper-V 컨테이너를 실행하려면 Hyper-V 역할을 설치해야 합니다.
- Windows Server 컨테이너 호스트에서는 Windows가 c:\에 설치되어야 합니다. Hyper-V 컨테이너만 배포할 경우 이 제한이 적용되지 않습니다.

## <a name="virtualized-container-hosts"></a>가상화된 컨테이너 호스트

Windows 컨테이너 호스트가 Hyper-V 가상 컴퓨터에서 실행되고 Hyper-V 호스트 컨테이너를 호스팅할 경우 중첩된 가상화를 사용해야 합니다. 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- Windows Server 빌드 1709, Windows Server 2016 또는 호스트 시스템의 Windows 10, 가상 컴퓨터의 Windows Server(Full, Core).
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
<td><center>Windows Server 2016(Standard 또는 Datacenter)</center></td>
<td><center>Server Core/Nano 서버</center></td>
<td><center>Server Core/Nano 서버</center></td>
</tr>
<tr valign="top">
<td><center>Nano 서버*</center></td>
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
* Windows Server 버전 1709부터 Nano 서버는 더 이상 컨테이너 호스트로 사용할 수 없습니다.

### <a name="memory-requirments"></a>메모리 요구 사항
컨테이너에 대해 사용 가능한 메모리에 대한 제한은 [리소스 컨트롤](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)을 통해 또는 컨테이너 호스트를 오버로드하여 구성할 수 있습니다.  컨테이너를 시작하고 기본 명령(ipconfig, dir 등)을 실행하는 데 필요한 최소 메모리 용량은 다음과 같습니다.  이러한 값은 컨테이너 간 리소스 공유 또는 컨테이너에서 실행되는 응용 프로그램의 요구 사항을 고려하지 않습니다.

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

이러한 것들이 가장 큰 차이점이며 그 외에도 다른 차이점이 더 있습니다. 여기서는 다루지 않았지만 제외된 다른 구성 요소가 더 있습니다. 적합하다고 판단될 경우 언제든지 Nano 서버 위에 계층을 추가할 수 있다는 점을 기억하세요. 관련 예제는 [NET Core Nano 서버 Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile)을 참조하세요.

## <a name="matching-container-host-version-with-container-image-versions"></a>컨테이너 호스트 버전과 컨테이너 이미지 버전 일치
### <a name="windows-server-containers"></a>Windows Server 컨테이너
Windows Server 컨테이너와 기본 호스트는 단일 커널을 공유하기 때문에 컨테이너의 기본 이미지는 호스트의 기본 이미지와 일치해야 합니다.  버전이 다른 경우 컨테이너는 시작할 수 있지만 일부 기능은 사용하지 못할 수 있습니다. 그러므로 일치하지 않는 버전은 지원되지 않습니다.  Windows 운영 체제에는 주 버전, 부 버전, 빌드 및 수정의 네 가지 수준의 버전이 있습니다(예: 10.0.14393.0). 빌드 번호는 OS의 새 버전이 게시되는 경우에만 변경됩니다. 수정 번호는 Windows 업데이트가 적용되면 업데이트됩니다. Windows Server 컨테이너는 빌드 번호가 다른 경우 시작이 차단됩니다(예: 10.0.14300.1030(Technical Preview 5) 및 10.0.14393(Windows Server 2016 RTM)). 빌드 번호가 일치하지만 수정 번호가 다른 경우 시작이 차단되지 않습니다(예: 10.0.14393(Windows Server 2016 RTM) 및 10.0.14393.206(Windows Server 2016 GA)). 기술적으로는 차단되지 않지만 일부 상황에서 제대로 작동하지 않을 수 있는 구성이므로 제품 환경에 대해 지원되지 않습니다. 

Windows 호스트에서 설치한 버전을 확인하기 위해 HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion을 쿼리할 수 있습니다.  기본 이미지에서 사용하는 버전을 확인하기 위해 Docker 허브에서 태그를 검토하거나 이미지 설명에 제공된 이미지 해시 테이블을 검토할 수 있습니다.  [Windows 10 업데이트 기록](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) 페이지에는 각 빌드 및 수정이 릴리스된 날짜가 나와 있습니다.

이 예에서 14393은 주 빌드 번호이고 321은 수정 번호입니다.
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### <a name="hyper-v-isolation-for-containers"></a>컨테이너에 대한 Hyper-V 격리
Hyper-V 격리를 사용하여 또는 사용하지 않고 Windows 컨테이너를 실행할 수 있습니다.  Hyper-V 격리는 최적화된 VM을 사용하여 컨테이너 주위에 안전한 경계를 만듭니다.  컨테이너와 호스트 간에 커널을 공유하는 표준 Windows 컨테이너와 달리, 격리된 각 Hyper-V 컨테이너가 고유의 Windows 커널 인스턴스를 갖습니다.  따라서 컨테이너 호스트 및 이미지의 OS 버전이 달라도 됩니다(아래 호환성 매트릭스 참조).  

Hyper-V 격리를 사용하여 컨테이너를 실행하려면 간단하게 docker run 명령에 "--isolation=hyper-v" 태그를 추가하기만 하면 됩니다.

### <a name="compatibility-matrix"></a>호환성 매트릭스
2016 GA(10.0.14393.206) 이후의 Windows Server 빌드는 수정 번호와 상관없이 지원되는 구성에서 Windows Server Core 또는 Nano 서버의 Windows Server 2016 GA 이미지를 실행할 수 있습니다.    

Windows 업데이트에서 제공하는 모든 기능, 안정성 및 보안 보증을 받으려면 모든 시스템에서 최신 버전을 유지해야 합니다.  
