---
title: Windows 컨테이너의 인쇄 스풀러
description: Windows 컨테이너의 인쇄 스풀러 서비스에 대 한 현재 작업 동작에 대해 설명 합니다.
keywords: docker, 컨테이너, 프린터, 스풀러
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999100"
---
# <a name="print-spooler-in-windows-containers"></a>Windows 컨테이너의 인쇄 스풀러

인쇄 서비스에 종속 된 응용 프로그램은 Windows 컨테이너에서 성공적으로 containerized 수 있습니다. 프린터 서비스 기능을 성공적으로 사용 하도록 설정 하기 위해 충족 해야 하는 특별 한 요구 사항이 있습니다. 이 가이드에서는 배포를 올바르게 구성 하는 방법에 대해 설명 합니다.

> [!IMPORTANT]
> 컨테이너에서 인쇄 서비스에 대 한 액세스를 성공적으로 진행 하는 동안에는 양식이 제한적으로 작동 합니다. 일부 인쇄 관련 작업이 작동 하지 않을 수 있습니다. 예를 들어 호스트에 프린터 드라이버를 설치 하는 데 종속성이 있는 앱 **은 컨테이너 내에서 드라이버 설치가 지원**되지 않기 때문에 containerized 될 수 없습니다. 컨테이너에서 지원 하려는 지원 되지 않는 인쇄 기능을 발견 한 경우 아래에서 피드백을 열어 주십시오.

## <a name="setup"></a>Setup

* 호스트는 Windows Server 2019 또는 Windows 10 Pro/Enterprise 10 월 2018 업데이트 이상 이어야 합니다.
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) 이미지는 대상 지정 된 기본 이미지 여야 합니다. 다른 Windows 컨테이너 기본 이미지 (Nano 서버 및 Windows Server Core 등)는 인쇄 서버 역할을 수행 하지 않습니다.

### <a name="hyper-v-isolation"></a>Hyper-V 격리

Hyper-v 격리를 사용 하 여 컨테이너를 실행 하는 것이 좋습니다. 이 모드에서 실행 되는 경우 인쇄 서비스에 대 한 액세스 권한으로 실행 하는 데 필요한 만큼의 컨테이너를 만들 수 있습니다. 호스트에서 스풀러 서비스를 수정할 필요는 없습니다.

다음 PowerShell 쿼리를 사용 하 여 기능을 확인할 수 있습니다.

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>프로세스 격리

프로세스 격리 컨테이너의 공유 커널 특성으로 인해, 현재 동작은 사용자가 호스트 및 모든 컨테이너 하위 항목에서 하나의 프린터 스풀러 서비스 **인스턴스만** 실행 하도록 제한 합니다. 호스트에 프린터 스풀러가 실행 되는 경우 attemping에서 게스트의 프린터 서비스를 시작 하기 전에 호스트에서 서비스를 중지 해야 합니다.

> [!TIP]
> 컨테이너와 호스트에서 모두 동시에 스풀러 서비스에 대 한 쿼리를 실행 하는 경우 둘 다 ' 실행 중 '으로 보고 됩니다. 그러나 deceived 되지 않음--컨테이너는 사용 가능한 프린터 목록을 쿼리할 수 없게 됩니다. 호스트의 스풀러 서비스를 실행 하지 않아야 합니다. 

호스트가 프린터 서비스를 실행 중인지 확인 하려면 아래 PowerShell의 쿼리를 사용 합니다.

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

호스트에서 스풀러 서비스를 중지 하려면 아래 PowerShell에서 다음 명령을 사용 합니다.

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

컨테이너를 시작 하 고 프린터에 대 한 액세스를 확인 합니다.

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```