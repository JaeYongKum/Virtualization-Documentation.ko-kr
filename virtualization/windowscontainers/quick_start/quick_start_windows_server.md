---
title: "Windows Server의 Windows 컨테이너"
description: "컨테이너 배포 빠른 시작"
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 891c9e9805bf2089fd11f86420de5ed251916c3f
ms.openlocfilehash: 77dca1499abf406b1d599c28afdb19dd823b8401

---

# Windows Server의 Windows 컨테이너

이 연습에서는 Windows Server에서 Windows 컨테이너 기능의 기본 배포 및 사용에 대해 안내합니다. 완료 후에는 컨테이너 역할이 설치되고 간단한 Windows Server 컨테이너가 배포됩니다. 이 빠른 시작을 시작하기 전에 기본 컨테이너 개념과 용어를 잘 이해해야 합니다. 이 정보는 [빠른 시작 소개](./quick_start.md)에서 확인할 수 있습니다.

이 빠른 시작은 Windows Server 2016의 Windows Server 컨테이너와 관련이 있습니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

**필수 조건:**

Windows Server 2016이 실행되는 컴퓨터 시스템(물리적 또는 가상) 1대. Windows Server 2016 TP5를 사용하는 경우 [Window Server 2016 평가판](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 )으로 업데이트하세요. 

## 1. 컨테이너 기능 설치

Windows 컨테이너를 사용하려면 먼저 컨테이너 기능을 사용하도록 설정해야 합니다. 이렇게 하려면 관리자 권한 PowerShell 세션에서 다음 명령을 실행합니다.

```none
Install-WindowsFeature containers
```

기능 설치가 완료되면 컴퓨터를 다시 부팅합니다.

```none
Restart-Computer -Force
```

## 2. Docker 설치

Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다. 이 연습에서는 둘 다 설치됩니다.

상업적으로 지원되는 Docker 엔진 및 클라이언트의 릴리스 후보를 zip 보관 파일로 다운로드합니다.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

zip 보관 파일을 프로그램 파일로 확장합니다.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

시스템 경로에 Docker 디렉터리를 추가합니다.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Docker를 Windows 서비스로 설치하려면 다음을 실행합니다.

```none
dockerd.exe --register-service
```

설치되면 서비스를 시작할 수 있습니다.

```none
Start-Service docker
```

## 3. 첫 번째 컨테이너 배포

이 연습에서는 미리 만든 .NET 샘플 이미지를 Docker 허브 레지스트리에서 다운로드하고 .Net Hello World 응용 프로그램을 실행하는 간단한 컨테이너를 배포합니다.  

`docker run`을 사용하여 .NET 컨테이너를 배포합니다. 그러면 컨테이너 이미지도 다운로드되며, 다운로드는 몇 분 정도 걸릴 수 있습니다.

```none
docker run microsoft/sample-dotnet
```

컨테이너가 시작되면 Hello World 메시지를 인쇄한 다음 종료합니다.

```none
       Welcome to .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....
```

Docker Run 명령에 대한 자세한 내용은 [Docker.com의 Docker Run Reference(Docker Run 참조)]( https://docs.docker.com/engine/reference/run/)를 참조하세요.

## 다음 단계

[Windows Server의 컨테이너 이미지](./quick_start_images.md)

[Windows 10의 Windows 컨테이너](./quick_start_windows_10.md)


<!--HONumber=Sep16_HO4-->


