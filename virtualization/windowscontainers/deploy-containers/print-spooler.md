---
title: Windows 컨테이너의 인쇄 스풀러
description: Windows 컨테이너의 인쇄 스풀러 서비스에 대해 현재 작동 중인 동작에 대해 설명합니다.
keywords: docker, 컨테이너, 프린터, 스풀러
author: cwilhit
ms.topic: conceptual
ms.openlocfilehash: 3348fc4665a9fe88d1cd299df665e564fa176a7e
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192221"
---
# <a name="print-spooler-in-windows-containers"></a>Windows 컨테이너의 인쇄 스풀러

인쇄 서비스에 종속된 애플리케이션은 Windows 컨테이너로 컨테이너화할 수 있습니다. 프린터 서비스 기능을 사용하려면 반드시 충족해야 하는 특별한 요구 사항이 있습니다. 이 가이드에서는 배포를 적절하게 구성하는 방법을 설명합니다.

> [!IMPORTANT]
> 컨테이너에서 인쇄 서비스에 액세스할 수 있지만 기능이 제한됩니다. 일부 인쇄 관련 작업이 작동하지 않을 수 있습니다. 예를 들어 호스트에 프린터 드라이버를 설치하는 것과 관련하여 종속성이 있는 앱은 **컨테이너 내에서 드라이버를 설치할 수 없으므로** 컨테이너화할 수 없습니다. 지원되지 않는 인쇄 기능 중 컨테이너에서 지원되기를 원하는 기능이 있으면 아래에서 피드백을 제공해 주세요.

## <a name="setup"></a>설치

* 호스트는 Windows Server 2019 또는 Windows 10 Pro/Enterprise 2018년 10월 업데이트 이상이어야 합니다.
* [mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) 이미지는 대상 기본 이미지여야 합니다. 다른 Windows 컨테이너 기본 이미지(예: Nano Server 및 Windows Server Core)는 인쇄 서버 역할을 수행하지 않습니다.

### <a name="hyper-v-isolation"></a>Hyper-V 격리

Hyper-V 격리를 사용하여 컨테이너를 실행하는 것이 좋습니다. 이 모드에서 실행하는 경우 인쇄 서비스에 대한 액세스 권한을 사용하여 실행할 컨테이너 수를 원하는 만큼 지정할 수 있습니다. 호스트에서 스풀러 서비스를 수정할 필요가 없습니다.

다음 PowerShell 쿼리를 사용하여 기능을 확인할 수 있습니다.

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

프로세스 격리 컨테이너의 공유 커널 특성으로 인해, 현재 동작은 호스트 및 호스트의 모든 컨테이너 자식에서 프린터 스풀러 서비스의 **인스턴스를 하나만** 실행하도록 사용자를 제한합니다. 호스트에서 프린터 스풀러를 실행하는 경우 게스트로 프린터 서비스를 시작하려면 먼저 호스트에서 서비스를 중지해야 합니다.

> [!TIP]
> 컨테이너를 시작하고 컨테이너와 호스트에서 동시에 스풀러 서비스를 쿼리하면 둘 다 '실행 중' 상태를 보고합니다. 하지만 속지 마세요. 컨테이너는 사용 가능한 프린터 목록을 쿼리할 수 없습니다. 호스트의 스풀러 서비스가 실행되면 안 됩니다.

호스트가 프린터 서비스를 실행하고 있는지 확인하려면 아래 PowerShell에서 쿼리를 사용합니다.

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

호스트에서 스풀러 서비스를 중지하려면 아래 PowerShell에서 다음 명령을 사용합니다.

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

컨테이너를 시작하고 프린터에 대한 액세스를 확인합니다.

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