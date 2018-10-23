---
title: Windows 10의 Windows 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: Docker, 컨테이너
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: aa69739697e9e2c668aaf5026c6f5ddc69d43c4d
ms.sourcegitcommit: f6f53fd0da65ac44d16fb793c5aa1af56c14d90e
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/23/2018
ms.locfileid: "5428995"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10의 Windows 컨테이너

이 연습에서는 Windows 10 Professional 또는 Enterprise(Anniversary Edition)에서 Windows 컨테이너 기능의 기본 배포 및 사용에 대해 안내합니다. 이 연습을 통해 Windows용 Docker를 설치하고 간단한 컨테이너를 실행해 볼 수 있습니다. 컨테이너에 대해 좀 더 숙지해야 하는 경우 [컨테이너 정보](../about/index.md)에서 이 정보를 찾을 수 있습니다.

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

***Hyper-V 격리:*** Windows Server 컨테이너는 프로덕션에 사용되는 동일한 커널 버전 및 구성을 개발자에게 제공하려면 Windows 10에서 Hyper-V 격리가 필요합니다. 자세한 내용은 [Windows 컨테이너 정보](../about/index.md) 페이지에서 확인할 수 있습니다.

**필수 조건:**

- Windows 10 Anniversary Edition 또는 크리에이터스 업데이트(Professional 또는 Enterprise)를 실행하는 물리적 컴퓨터 시스템 하나.   
- 이 빠른 시작은 Windows 10 가상 컴퓨터에서 실행할 수 있지만 중첩된 가상화를 사용하도록 설정해야 합니다. 자세한 정보는 [중첩된 가상화 가이드](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)에서 확인할 수 있습니다.

> 작업하기 위해 Windows 컨테이너에 대한 중요 업데이트를 설치해야 합니다.
> 실행 중인 OS 버전을 확인하려면 `winver.exe`, 를 실행한 후, [Windows 10 업데이트 기록](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)

> 계속하기 전에 14393.222 이상인지 확인합니다.  이 버전 1607 이상의 모든 버전을 완전히 지원 해야 하므로 Windows 10 버전 1607에 해당 합니다.

## <a name="1-install-docker-for-windows"></a>1. Windows용 Docker 설치

[Windows용 Docker를 다운로드](https://download.docker.com/win/stable/InstallDocker.msi)하고 설치 프로그램을 실행합니다. [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

## <a name="2-switch-to-windows-containers"></a>2. Windows 컨테이너로 전환

설치가 완료되면 Windows용 Docker는 기본적으로 Linux 컨테이너를 실행하도록 설정됩니다. Docker 트레이 메뉴를 사용하거나 PowerShell 프롬프트에서 `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon` 명령을 실행하여 Windows 컨테이너로 전환합니다.

![](./media/docker-for-win-switch.png)

## <a name="3-install-base-container-images"></a>3. 기본 컨테이너 이미지 설치

Windows 컨테이너는 기본 이미지로 만듭니다. 다음 명령은 Nano Server 기본 이미지를 가져옵니다.

```
docker pull microsoft/nanoserver
```

이미지를 가져온 후 `docker images` 명령을 실행하면 설치된 이미지(여기서는 Nano Server 이미지) 목록이 반환됩니다.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [EULA](../images-eula.md)에서 Windows 컨테이너 OS 이미지 EULA를 읽어보세요.

## <a name="4-run-your-first-container"></a>4. 첫 번째 컨테이너 실행

이 간단한 예제에서는 'Hello World' 컨테이너 이미지를 만들고 배포합니다. 최상의 환경을 유지하려면 관리자 권한 Windows CMD 셸 또는 PowerShell에서 이 명령을 실행합니다.

> Windows PowerShell ISE는 컨테이너가 있는 대화형 세션에서 작동하지 않습니다. 컨테이너가 실행 중인 경우에도 중단된 것으로 표시됩니다.

먼저 `nanoserver` 이미지에서 대화형 세션을 사용하여 컨테이너를 시작합니다. 컨테이너가 시작되면 컨테이너 내에서 명령 셸이 표시됩니다.  

```
docker run -it microsoft/nanoserver cmd
```

컨테이너 내에서 간단한 'Hello World' 스크립트를 만듭니다.

```
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

만들었으면 컨테이너를 종료합니다.

```
exit
```

이제 수정된 컨테이너에서 새 컨테이너 이미지를 만듭니다. 컨테이너 목록을 보려면 다음을 실행하고 컨테이너 ID를 적어둡니다.

```
docker ps -a
```

다음 명령을 실행하여 새 ‘HelloWorld’ 이미지를 만듭니다. <containerid>를 컨테이너 ID로 바꿉니다.

```
docker commit <containerid> helloworld
```

완료되면 hello world 스크립트가 포함된 사용자 지정 이미지가 생깁니다. 이 이미지를 확인하려면 다음 명령을 사용합니다.

```
docker images
```

마지막으로 컨테이너를 실행하려면 `docker run` 명령을 사용입니다.

```
docker run --rm helloworld powershell c:\helloworld.ps1
```

`docker run` 명령의 결과로 'HelloWorld' 이미지에서 Hyper-V 컨테이너를 만들고 샘플 'Hello World' 스크립트를 실행(출력은 셸에 에코됨)한 다음 컨테이너를 중지하고 제거했습니다.
이후 Windows 10 및 컨테이너 빠른 시작에서는 Windows 10의 컨테이너에서 응용 프로그램을 만들고 배포하는 과정을 자세히 살펴봅니다.

## <a name="next-steps"></a>다음 단계

다음 자습서로 넘어가서 [샘플 앱 빌드](./building-sample-app.md) 예제를 살펴봅니다.
