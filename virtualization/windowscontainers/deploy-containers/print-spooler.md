---
title: Windows 컨테이너의 인쇄 스풀러
description: Windows 컨테이너에서 인쇄 스풀러 서비스의 현재 작동 동작에 대해 설명 합니다.
keywords: docker, 컨테이너, 프린터, 스풀러
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910533"
---
# <a name="print-spooler-in-windows-containers"></a>Windows 컨테이너의 인쇄 스풀러

인쇄 서비스에 대 한 종속성이 있는 응용 프로그램은 Windows 컨테이너에서 성공적으로 컨테이너 화 된 수 있습니다. 프린터 서비스 기능을 사용 하도록 설정 하기 위해 충족 해야 하는 특별 한 요구 사항이 있습니다. 이 가이드에서는 배포를 적절 하 게 구성 하는 방법을 설명 합니다.

> [!IMPORTANT]
> 컨테이너에서 인쇄 서비스에 대 한 액세스를 성공적으로 수행 하는 동안 기능은 폼에서 제한 됩니다. 일부 인쇄 관련 작업은 작동 하지 않을 수 있습니다. 예를 들어 호스트에 프린터 드라이버를 설치 하는 데 종속성이 있는 앱은 컨테이너 화 된 수 없습니다. **컨테이너 내에서 드라이버를 설치 하는 것은 지원**되지 않기 때문입니다. 컨테이너에서 지원 되는 지원 되지 않는 인쇄 기능을 찾은 경우 아래 피드백을 여세요.

## <a name="setup"></a>설치

* 호스트는 Windows Server 2019 또는 Windows 10 Pro/Enterprise 10 월 2018 업데이트 이상 이어야 합니다.
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) 이미지는 대상 기본 이미지 여야 합니다. 다른 Windows 컨테이너 기본 이미지 (예: Nano Server 및 Windows Server Core)는 인쇄 서버 역할을 수행 하지 않습니다.

### <a name="hyper-v-isolation"></a>Hyper-V 격리

Hyper-v 격리를 사용 하 여 컨테이너를 실행 하는 것이 좋습니다. 이 모드에서 실행 하는 경우 인쇄 서비스에 대 한 액세스 권한으로 실행할 컨테이너 수를 지정할 수 있습니다. 호스트에서 스풀러 서비스를 수정할 필요가 없습니다.

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

프로세스 격리 컨테이너의 공유 커널 특성으로 인해 현재 동작은 호스트와 모든 컨테이너 자식에서 프린터 스풀러 서비스의 **인스턴스** 를 하나만 실행 하도록 사용자를 제한 합니다. 호스트에서 프린터 스풀러를 실행 하는 경우 게스트에서 프린터 서비스를 시작 하려면 attemping 전에 호스트에서 서비스를 중지 해야 합니다.

> [!TIP]
> 컨테이너를 시작 하 고 컨테이너와 호스트 모두에서 스풀러 서비스에 대해 동시에 쿼리 하는 경우 둘 다 ' 실행 중 '으로 상태를 보고 합니다. 그러나 deceived는 안 됩니다. 컨테이너는 사용 가능한 프린터 목록을 쿼리할 수 없습니다. 호스트의 스풀러 서비스가 실행 되지 않아야 합니다. 

호스트가 프린터 서비스를 실행 하 고 있는지 확인 하려면 아래의 PowerShell에서 쿼리를 사용 합니다.

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