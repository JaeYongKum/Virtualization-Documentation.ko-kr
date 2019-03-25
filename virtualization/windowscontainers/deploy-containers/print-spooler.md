---
title: Windows 컨테이너에서 인쇄 스풀러
description: Windows 컨테이너에서 인쇄 스풀러 서비스에 대 한 현재 작업 동작에 설명
keywords: docker, 컨테이너, 프린터, 스풀러
author: cwilhit
ms.openlocfilehash: 45176e651ee2ef9b6daea9919004601734084083
ms.sourcegitcommit: 04c372c87c832f73a1aa120b0ff6c2c2b9c8c1b1
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/25/2019
ms.locfileid: "9257988"
---
# <a name="print-spooler-in-windows-containers"></a>Windows 컨테이너에서 인쇄 스풀러

인쇄 서비스에 종속 된 응용 프로그램 수 될 컨테이너 화 된 성공적으로 Windows 컨테이너를 사용 합니다. 호스트에 프린터 드라이버 설치에 종속 된 응용 프로그램을 컨테이너 화 된 수 없습니다. 컨테이너 호스트에는 상태를 누설 합니다 컨테이너 내에서 드라이버 설치는 지원 하지 않습니다. 성공적으로 프린터 서비스 기능을 사용 하기 위해 충족 해야 하는 특별 한 요구 사항이 있습니다. 이 가이드는 배포 구성 하는 방법을 설명 합니다.

> [!IMPORTANT]
> 기능 폼 제한 되는 인쇄에 대 한 액세스를 얻는 서비스 성공적으로 컨테이너에서 작동 하는 동안 일부 인쇄 관련 작업을 작동할 수 있습니다. 이 경우 아래 대 한 피드백을 여세요.

## <a name="setup"></a>Setup

* Windows 10 Pro/Enterprise 또는 Windows Server 2019 호스트 되어야 2018 년 10 월 업데이트 또는 최신 합니다.
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) 이미지를 대상으로 지정 된 기본 이미지 되어야 합니다. 다른 Windows 컨테이너 기본 이미지 (예: Nano 서버 및 Windows Server Core) 인쇄 서버 역할을 수행 하지 않습니다.

### <a name="hyper-v-isolation"></a>Hyper-V 격리

Hyper-v 격리를 사용 하 여 컨테이너를 실행 하는 것이 좋습니다. 이 모드에서 실행 하는 인쇄 서비스에 대 한 액세스를 사용 하 여 실행을 원하는 만큼 많은 컨테이너 있을 수 있습니다. 호스트에서 스풀러 서비스를 수정할 필요가 없습니다.

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

현재 동작이 프로세스 격리 된 컨테이너 공유 커널 특성으로 인해 호스트와 모든 컨테이너 자식에서 프린터 스풀러 서비스 **인스턴스만** 을 실행 하는 사용자를 제한 합니다. 호스트 실행 프린터 스풀러에 있는 경우에 게스트에서 프린터 서비스를 시작 하는 하기 전에 호스트에서 서비스를 중지 해야 합니다.

> [!TIP]
> 컨테이너를 시작 하 고 동시에 컨테이너와 호스트에서 스풀러 서비스에 대 한 쿼리 하는 경우 둘 다 '실행'으로 상태를 보고 합니다. 하지만 deceived-컨테이너 목록은 사용 가능한 프린터를 쿼리할 수 없습니다 되지는지 않습니다. 호스트의 스풀러 서비스가 실행 되지 해야 합니다. 

호스트 프린터 서비스가 실행 되 고 있는지를 확인 하려면 다음 PowerShell에서 쿼리를 사용 합니다.

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