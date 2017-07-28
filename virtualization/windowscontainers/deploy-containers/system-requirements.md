---
title: "Windows의 컨테이너 요구 사항"
description: "Windows의 컨테이너 요구 사항입니다."
keywords: "메타데이터, 컨테이너"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: d4594bd5efe3e1852f6a47b6474bf9ea56196c14
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/21/2017
---
# Windows의 컨테이너 요구 사항

이 가이드는 Windows 컨테이너 호스트에 대한 요구 사항을 나열합니다.

## OS 요구 사항

- Windows 컨테이너 기능은 Windows Server 2016(코어 및 데스크톱 경험 포함), Nano Server, Windows 10 Professional 및 Enterprise(Anniversary Edition)에서만 사용할 수 있습니다.
- Hyper-V 컨테이너를 실행하려면 Hyper-V 역할을 설치해야 합니다.
- Windows Server 컨테이너 호스트에서는 Windows가 c:\에 설치되어야 합니다. Hyper-V 컨테이너만 배포할 경우 이 제한이 적용되지 않습니다.

## 가상화된 컨테이너 호스트

Windows 컨테이너 호스트가 Hyper-V 가상 컴퓨터에서 실행되고 Hyper-V 호스트 컨테이너를 호스팅할 경우 중첩된 가상화를 사용해야 합니다. 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- Windows Server 2016 또는 호스트 시스템의 Windows 10, 가상 컴퓨터의 Windows Server(Full, Core) 또는 Nano Server.
- Intel VT-x가 포함된 프로세서.(이 기능은 현재 Intel 프로세서에 대해서만 사용할 수 있습니다)
- 또한 컨테이너 호스트 VM에는 적어도 2개의 가상 프로세서가 필요합니다.

## 지원되는 기본 이미지

Windows 컨테이너는 두 컨테이너 기본 이미지(Windows Server Core 및 Nano Server)와 함께 제공됩니다. 일부 구성은 두 개의 OS 이미지를 모두 지원하지 않습니다. 이 테이블은 지원되는 구성을 자세히 설명합니다.

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
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro/Enterprise</center></td>
<td><center>사용할 수 없음</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
</tbody>
</table>

## 컨테이너 호스트 버전과 컨테이너 이미지 버전 일치
### Windows Server 컨테이너
Windows Server 컨테이너와 기본 호스트는 단일 커널을 공유하기 때문에 컨테이너의 기본 이미지는 호스트의 기본 이미지와 일치해야 합니다.  버전이 다른 경우 컨테이너는 시작할 수 있지만 일부 기능은 사용하지 못할 수 있습니다. 그러므로 일치하지 않는 버전은 지원되지 않습니다.  Windows 운영 체제에는 주 버전, 부 버전, 빌드 및 수정의 네 가지 수준의 버전이 있습니다(예: 10.0.14393.0). 빌드 번호는 OS의 새 버전이 게시되는 경우에만 변경됩니다. 수정 번호는 Windows 업데이트가 적용되면 업데이트됩니다. Windows Server 컨테이너는 빌드 번호가 다른 경우 시작이 차단됩니다(예: 10.0.14300.1030(Technical Preview 5) 및 10.0.14393(Windows Server 2016 RTM)). 빌드 번호가 일치하지만 수정 번호가 다른 경우 시작이 차단되지 않습니다(예: 10.0.14393(Windows Server 2016 RTM) 및 10.0.14393.206(Windows Server 2016 GA)). 기술적으로는 차단되지 않지만 일부 상황에서 제대로 작동하지 않을 수 있는 구성이므로 제품 환경에 대해 지원되지 않습니다. 

Windows 호스트에서 설치한 버전을 확인하기 위해 HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion을 쿼리할 수 있습니다.  기본 이미지에서 사용하는 버전을 확인하기 위해 Docker 허브에서 태그를 검토하거나 이미지 설명에 제공된 이미지 해시 테이블을 검토할 수 있습니다.  [Windows 10 업데이트 기록](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) 페이지에는 각 빌드 및 수정이 릴리스된 날짜가 나와 있습니다.

이 예에서 14393은 주 빌드 번호이고 321은 수정 번호입니다.
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### 컨테이너에 대한 Hyper-V 격리
Hyper-V 격리를 사용하여 또는 사용하지 않고 Windows 컨테이너를 실행할 수 있습니다.  Hyper-V 격리는 최적화된 VM을 사용하여 컨테이너 주위에 안전한 경계를 만듭니다.  컨테이너와 호스트 간에 커널을 공유하는 표준 Windows 컨테이너와 달리, 격리된 각 Hyper-V 컨테이너가 고유의 Windows 커널 인스턴스를 갖습니다.  따라서 컨테이너 호스트 및 이미지의 OS 버전이 달라도 됩니다(아래 호환성 매트릭스 참조).  

Hyper-V 격리를 사용하여 컨테이너를 실행하려면 간단하게 docker run 명령에 "--isolation=hyper-v" 태그를 추가하기만 하면 됩니다.

### 호환성 매트릭스
2016 GA(10.0.14393.206) 이후의 Windows Server 빌드는 수정 번호와 상관없이 지원되는 구성에서 Windows Server Core 또는 Nano 서버의 Windows Server 2016 GA 이미지를 실행할 수 있습니다.    

Windows 업데이트에서 제공하는 모든 기능, 안정성 및 보안 보증을 받으려면 모든 시스템에서 최신 버전을 유지해야 합니다.  
