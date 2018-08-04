---
title: Windows Server의 Windows 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: Docker, 컨테이너
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: 7d526aa64d478516a3f66acaf62b62b45282e5af
ms.sourcegitcommit: 3d72f15651da378908f134916cd5c9d2064f8f95
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/12/2018
ms.locfileid: "2256936"
---
# <a name="windows-containers-on-windows-server"></a>Windows Server의 Windows 컨테이너

이 연습에서는 Windows Server 2016에서 Windows 컨테이너 기능의 기본 배포 및 사용에 대해 안내합니다. 이 연습을 하는 동안 컨테이너 역할을 설치하고 간단한 Windows Server 컨테이너를 배포하게 됩니다. 컨테이너에 대해 좀 더 숙지해야 하는 경우 [컨테이너 정보](../about/index.md)에서 이 정보를 찾을 수 있습니다.

이 빠른 시작은 Windows Server 2016의 Windows Server 컨테이너와 관련이 있습니다. Windows 10의 컨테이너를 포함하여 추가적인 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

**필수 구성 요소:**

Windows Server 2016이 실행되는 컴퓨터 시스템(물리적 또는 가상) 1대. Windows Server 2016 TP5를 사용하는 경우 [Window Server 2016 평가판](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 )으로 업데이트하세요.

> 중요 업데이트는 함수에서 Windows 컨테이너 기능을 위해 필요합니다. 이 자습서를 수행하기 전에 모든 업데이트를 설치하세요.

Azure에서 배포 하려는 경우 이 [템플릿](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)이 편리합니다.<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="1-install-docker"></a>1. Docker 설치

Docker를 설치하기 위해 [OneGet 공급자 PowerShell 모듈](https://github.com/oneget/oneget)을 사용하겠습니다. 이 모듈은 공급자(여기서는 [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider))와 함께 작동하여 설치를 수행합니다. 공급자를 사용하여 컴퓨터에서 컨테이너 기능을 사용하도록 설정할 수 있습니다. Docker 설치를 위해 컴퓨터를 다시 부팅해야 할 수도 있습니다. Windows 컨테이너를 사용하려면 Docker가 필요합니다. Docker는 Docker 엔진 및 Docker 클라이언트로 구성됩니다.

관리자 권한으로 PowerShell 세션을 열고 다음 명령을 실행합니다.

먼저 [PowerShell 갤러리](https://www.powershellgallery.com/packages/DockerMsftProvider)에서 Docker-Microsoft PackageManagement Provider를 설치합니다.

```
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

그런 다음 PackageManagement PowerShell 모듈을 사용하여 최신 버전의 Docker를 설치합니다.
```
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell에서 'DockerDefault' 패키지 소스를 신뢰할 수 있는지 물으면 `A`를 입력하여 설치를 계속합니다. 설치가 완료되면 컴퓨터를 다시 부팅합니다.

```
Restart-Computer -Force
```

> 팁: Docker를 나중에 업데이트하려면:
>  - 다음을 사용하여 설치된 버전을 확인 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 다음을 사용하여 현재 버전을 검색 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 준비를 마쳤으면 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` 명령을 사용하여 업그레이드, 그리고 `Start-Service Docker`

## <a name="2-install-windows-updates"></a>2. Windows 업데이트 설치

다음을 실행하여 Windows Server 시스템이 최신 상태인지 확인합니다.

```
sconfig
```

텍스트 기반 구성 메뉴가 표시되는데, 여기서 업데이트 다운로드 및 설치를 위한 옵션 6을 선택할 수 있습니다.

```
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

## <a name="3-deploy-your-first-container"></a>3. 첫 번째 컨테이너 배포

이 연습에서는 미리 만든 .NET 샘플 이미지를 Docker 허브 레지스트리에서 다운로드하고 .Net Hello World 응용 프로그램을 실행하는 간단한 컨테이너를 배포합니다.  

`docker run`을 사용하여 .Net 컨테이너를 배포합니다. 그러면 컨테이너 이미지도 다운로드되며, 다운로드는 몇 분 정도 걸릴 수 있습니다.

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver
```

컨테이너가 시작되면 Hello World 메시지를 인쇄한 다음 종료합니다.

```console
         Dotnet-bot: Welcome to using .NET Core!
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


**Environment**
Platform: .NET Core 2.0
OS: Microsoft Windows 10.0.14393
```

Docker Run 명령에 대한 자세한 내용은 [Docker.com의 Docker Run Reference(Docker Run 참조)]( https://docs.docker.com/engine/reference/run/)를 참조하세요.

## <a name="next-steps"></a>다음 단계

[빌드 자동화 및 이미지 저장](./quick-start-images.md)
