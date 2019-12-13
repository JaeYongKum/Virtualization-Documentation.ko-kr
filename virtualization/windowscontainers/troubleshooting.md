---
title: Windows 컨테이너 문제 해결
description: Windows 컨테이너 및 Docker를 위한 문제 해결 팁, 자동화된 스크립트 및 로그 정보
keywords: docker, 컨테이너, 문제 해결, 로그
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: 1de86a2492ca899dc3fb932e0d57927fa4000fd0
ms.sourcegitcommit: 15b5ab92b7b8e96c180767945fdbb2963c3f6f88
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/06/2019
ms.locfileid: "74911713"
---
# <a name="troubleshooting"></a>문제 해결

컴퓨터를 설정하거나 컨테이너를 실행하는 데 문제가 있나요? 일반적인 문제를 확인하는 PowerShell 스크립트를 만들었습니다. 이 스크립트를 먼저 사용해 보고 찾는 내용과 결과를 공유해 주세요.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
일반적인 솔루션과 함께 실행하는 모든 테스트 목록은 스크립트의 [Readme file](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md)(추가 정보 파일)에 있습니다.

문제의 원인을 찾는 데 도움이 되지 않는 경우 스크립트 출력을 [컨테이너 포럼](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)에 게시하세요. 이 포럼은 Windows 참가자와 개발자를 비롯한 커뮤니티의 도움을 받을 수 있는 유용한 곳입니다.


### <a name="finding-logs"></a>로그 찾기
Windows 컨테이너를 관리 하는 데 사용 되는 여러 서비스가 있습니다. 다음 섹션에서는 각 서비스에 대한 로그를 확인할 수 있는 위치를 보여 줍니다.

## <a name="docker-container-logs"></a>Docker 컨테이너 로그 
`docker logs` 명령은 Linux 응용 프로그램에 대 한 표준 응용 프로그램 로그 보관 위치인 STDOUT/STDERR에서 컨테이너의 로그를 페치합니다. Windows 응용 프로그램은 일반적으로 STDOUT/STDERR에 기록 되지 않습니다. 대신, ETW, 이벤트 로그 또는 로그 파일에 기록 합니다. 

Microsoft에서 지 원하는 활용 도구인 [로그 모니터](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor)는 이제 github에서 사용할 수 있습니다. 로그 모니터는 Windows 응용 프로그램 로그를 STDOUT/STDERR에 연결 합니다. 로그 모니터는 구성 파일을 통해 구성 됩니다. 

### <a name="log-monitor-usage"></a>로그 모니터 사용

LogMonitor .exe 및 Logmonitorconfig.xml은 모두 동일한 LogMonitor 디렉터리에 포함 되어야 합니다. 

로그 모니터는 셸 사용 패턴에서 사용할 수 있습니다.

```
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "cmd", "/S", "/C"]
CMD c:\windows\system32\ping.exe -n 20 localhost
```

또는 ENTRYPOINT 사용 패턴:

```
ENTRYPOINT C:\LogMonitor\LogMonitor.exe c:\windows\system32\ping.exe -n 20 localhost
```

두 예제 모두 ping 응용 프로그램을 래핑합니다. 다른 응용 프로그램 (예: [IIS) ServiceMonitor]( https://github.com/microsoft/IIS.ServiceMonitor))는 유사한 방식으로 로그 모니터로 중첩 될 수 있습니다.

```
COPY LogMonitor.exe LogMonitorConfig.json C:\LogMonitor\
WORKDIR /LogMonitor
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "powershell.exe"]
 
# Start IIS Remote Management and monitor IIS
ENTRYPOINT      Start-Service WMSVC; `
                    C:\ServiceMonitor.exe w3svc;
```


로그 모니터는 래핑된 응용 프로그램을 자식 프로세스로 시작 하 고 응용 프로그램의 STDOUT 출력을 모니터링 합니다.

셸 사용 패턴에서 CMD/ENTRYPOINT 명령은 exec 형식이 아니라 SHELL 형식으로 지정 해야 합니다. CMD/ENTRYPOINT 명령의 exec 폼을 사용 하는 경우 셸이 시작 되지 않고 로그 모니터 도구가 컨테이너 내에서 시작 되지 않습니다.

추가 사용 정보는 [로그 모니터 wiki](https://github.com/microsoft/windows-container-tools/wiki)에서 찾을 수 있습니다. 핵심 Windows 컨테이너 시나리오 (IIS 등)의 구성 파일 예제는 [github 리포지토리](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor/src/LogMonitor/sample-config-files)내에서 찾을 수 있습니다. 추가 컨텍스트는이 [블로그 게시물](https://techcommunity.microsoft.com/t5/Containers/Windows-Containers-Log-Monitor-Opensource-Release/ba-p/973947)에서 찾을 수 있습니다.

## <a name="docker-engine"></a>Docker 엔진
Docker 엔진은 파일 대신 Windows '응용 프로그램' 이벤트 로그에 기록합니다. 이러한 로그는 Windows PowerShell을 사용하여 쉽게 읽고 정렬하고 필터링할 수 있습니다.

예를 들어 가장 오래된 순서부터 시작하여 최근 5분의 Docker 엔진 로그가 표시됩니다.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

또한 다른 도구 또는 스프레드시트로 읽을 수 있도록 CSV 파일로 쉽게 파이프할 수 있습니다.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

### <a name="enabling-debug-logging"></a>디버그 로깅 사용
Docker 엔진에서 디버그 수준 로깅을 사용하도록 설정할 수도 있습니다. 그러면 일반 로그에 정보가 부족한 경우의 문제 해결에 유용할 수 있습니다.

먼저 관리자 권한 명령 프롬프트를 연 다음 `sc.exe qc docker`를 실행하여 Docker 서비스에 대한 현재 명령줄을 가져옵니다.
예제:
```
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

현재 `BINARY_PATH_NAME`을 다음과 같이 수정합니다.
- 끝에 -D 추가
- 각 "를 \로 이스케이프
- 전체 명령 "로 묶기

그런 다음 새 문자열 다음에 오는 `sc.exe config docker binpath=`을(를) 실행합니다. 예를 들어 다음과 같은 가치를 제공해야 합니다. 
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


이제 Docker 서비스를 다시 시작합니다.
```
sc.exe stop docker
sc.exe start docker
```

이렇게 하면 응용 프로그램 이벤트 로그에 더 많은 정보가 기록되므로 문제 해결을 마치면 `-D` 옵션을 제거하는 것이 좋습니다. 위의 단계를 `-D` 없이 수행하고 서비스를 다시 시작하여 디버그 로깅을 사용하지 않도록 설정합니다.

위의 방법 대신 관리자 권한 PowerShell 프롬프트에서 Docker 데몬을 디버그 모드로 실행하여 출력을 파일로 직접 캡처하는 방법도 있습니다.
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

### <a name="obtaining-stack-dump"></a>스택 덤프를 가져오는 중

일반적으로 Microsoft 지원 또는 docker 개발자가 명시적으로 요청한 경우에만 유용 합니다. Docker가 멈춘 것 처럼 보이는 상황을 진단 하는 데 사용할 수 있습니다. 

[docker-signal.exe](https://github.com/jhowardmsft/docker-signal)를 다운로드하세요.

사용량:
```PowerShell
docker-signal --pid=$((Get-Process dockerd).Id)
```

출력 파일은 docker가 실행 되 고 있는 데이터 루트 디렉터리에 배치 됩니다. 기본 디렉터리는 `C:\ProgramData\Docker`입니다. 실제 디렉터리는 `docker info -f "{{.DockerRootDir}}"` 명령을 실행하여 확인할 수 있습니다.

파일이 `goroutine-stacks-<timestamp>.log`됩니다.

`goroutine-stacks*.log`에는 개인 정보가 포함 되어 있지 않습니다.


## <a name="host-compute-service"></a>호스트 계산 서비스
Docker 엔진은 Windows 관련 호스트 컨테이너 서비스를 사용합니다. 이 서비스에는 다음과 같은 별도의 로그가 있습니다. 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

이 로그는 이벤트 뷰어에서 볼 수 있고 PowerShell에서 쿼리할 수도 있습니다.

예를 들어 다음과 같은 가치를 제공해야 합니다.
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```

### <a name="capturing-hcs-analyticdebug-logs"></a>HCS 분석/디버그 로그 캡처

Hyper-V 계산에 분석/디버그 로그를 사용하려면 로그를 `hcslog.evtx`에 저장하세요.

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

### <a name="capturing-hcs-verbose-tracing"></a>HCS 자세한 추적 캡처

일반적으로 Microsoft 지원에서 요청한 경우에만 유용합니다. 

[HcsTraceProfile.wprp](https://gist.github.com/jhowardmsft/71b37956df0b4248087c3849b97d8a71) 다운로드

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

지원 담당자에게 `HcsTrace.etl`을 제공하세요.
