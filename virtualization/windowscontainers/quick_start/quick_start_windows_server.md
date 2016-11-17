---
title: "Windows Server의 Windows 컨테이너"
description: "컨테이너 배포 빠른 시작"
keywords: "Docker, 컨테이너"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 295199bbb8c93f26562ef918df30082e1dd78f6d
ms.openlocfilehash: bb61ccfb9756b341be2e989cb7c5bbe571072aab

---

# Windows Server의 Windows 컨테이너

이 연습에서는 Windows Server에서 Windows 컨테이너 기능의 기본 배포 및 사용에 대해 안내합니다. 완료 후에는 컨테이너 역할이 설치되고 간단한 Windows Server 컨테이너가 배포됩니다. 이 빠른 시작을 시작하기 전에 기본 컨테이너 개념과 용어를 잘 이해해야 합니다. 이 정보는 [빠른 시작 소개](./quick_start.md)에서 확인할 수 있습니다.

이 빠른 시작은 Windows Server 2016의 Windows Server 컨테이너와 관련이 있습니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

**필수 조건:**

Windows Server 2016이 실행되는 컴퓨터 시스템(물리적 또는 가상) 1대. Windows Server 2016 TP5를 사용하는 경우 [Window Server 2016 평가판](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 )으로 업데이트하세요. 

> 중요 업데이트는 함수에서 Windows 컨테이너 기능을 위해 필요합니다. 이 자습서를 수행하기 전에 모든 업데이트를 설치하세요.

Azure에서 배포 하려는 경우 이 [템플릿](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)이 편리합니다.<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## 1. Docker 설치

Docker를 설치하기 위해 [OneGet 공급자 PowerShell 모듈](https://github.com/oneget/oneget)을 사용합니다. 공급자는 컴퓨터에서 컨테이너 기능을 사용하도록 설정하고 Docker를 설치합니다. 그러면 컴퓨터를 다시 부팅해야 합니다. Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다.

관리자 권한 PowerShell 세션을 열고 다음 명령을 실행합니다.

먼저 OneGet PowerShell 모듈을 설치합니다.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

그런 다음 OneGet을 사용하여 최신 버전의 Docker를 설치합니다.
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell에서 'DockerDefault' 패키지 소스를 신뢰할 수 있는지 물으면 A를 입력하여 설치를 계속합니다. 설치가 완료되면 컴퓨터를 다시 부팅합니다.

```none
Restart-Computer -Force
```

## 2. Windows 업데이트 설치

Windows Server system 최신 상태로 유지하려면, 다음을 실행하여 Widows 업데이트를 설치해야 합니다.

```none
sconfig
```

텍스트 기반 구성 메뉴에서 표시되는데, 여기서 업데이트 다운로드 및 설치에 대한 옵션 6을 선택할 수 있습니다.

```none
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

메시지가 표시되면 모든 업데이트를 다운로드할 수 있는 옵션 A를 선택합니다.

## 3. 첫 번째 컨테이너 배포

이 연습에서는 미리 만든 .NET 샘플 이미지를 Docker 허브 레지스트리에서 다운로드하고 .Net Hello World 응용 프로그램을 실행하는 간단한 컨테이너를 배포합니다.  

`docker run`을 사용하여 .Net 컨테이너를 배포합니다. 그러면 컨테이너 이미지도 다운로드되며, 다운로드는 몇 분 정도 걸릴 수 있습니다.

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



<!--HONumber=Nov16_HO2-->


