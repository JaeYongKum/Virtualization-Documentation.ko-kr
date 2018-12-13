---
title: Windows 및 Windows 10에서 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: docker, 컨테이너, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 4202908f8797a2b98ab657c45cd9a6b33191bd6f
ms.sourcegitcommit: 9dfef8d261f4650f47e8137a029e893ed4433a86
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/09/2018
ms.locfileid: "6224902"
---
# <a name="windows-and-linux-containers-on-windows-10"></a>Windows 및 Windows 10에서 Linux 컨테이너

이 연습 만들기 및 Windows 10의 Windows와 Linux 컨테이너를 실행 하는 방법을 안내. 완료 후 다음이 포함 됩니다.

1. Windows 용 Docker를 설치
2. 간단한 Windows 컨테이너를 실행 합니다.
3. Windows (LCOW)에서 Linux 컨테이너를 사용 하 여 간단한 Linux 컨테이너를 실행 합니다.

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는이 페이지의 왼쪽에 있는 목차에서 찾을 수 있습니다.

***Hyper-v 격리:*** Windows Server 컨테이너는 동일한 커널 버전 및 프로덕션 환경에서 사용할 수 있는 구성을 개발자에 게 제공 하기 위해 Windows 10에서 Hyper-v 격리가 필요, 더 Hyper-v에 대 한 격리에서 확인할 수 있습니다 [Windows 컨테이너 정보](../about/index.md) 페이지.

**필수 조건:**

- [Hyper-v를 실행할 수](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 는 Windows 10 Fall Creators Update (버전 1709) 또는 이후 (Professional 또는 Enterprise) 실행 하는 하나의 물리적 컴퓨터 시스템

> 이 자습서에서는 LCOW 실행 하려는 Windows 컨테이너 이상 Windows 10 1 주년 업데이트 (버전 1607)에서 실행할 수 없게 됩니다.

## <a name="1-install-docker-for-windows"></a>1. Windows용 Docker 설치

[Windows 용 Docker를 다운로드](https://store.docker.com/editions/community/docker-ce-desktop-windows) 및 실행 설치 프로그램 (됩니다 로그인 해야 합니다. 계정이 생성 이미 계정이 없는 경우). [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

> Docker 설치를 이미 있는 경우 버전 LCOW 지원 하기 위해 18.02 이상 있는지 확인 합니다. 검사를 실행 하 여 `docker -v` *Docker에 대 한*검사 또는 합니다.

> '실험적 기능' 옵션 *Docker 설정 > 디먼* LCOW 컨테이너를 실행 하려면 활성화 해야 합니다.

## <a name="2-switch-to-windows-containers"></a>2. Windows 컨테이너로 전환

설치가 완료되면 Windows용 Docker는 기본적으로 Linux 컨테이너를 실행하도록 설정됩니다. Docker 트레이 메뉴를 사용하거나 PowerShell 프롬프트에서 `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon` 명령을 실행하여 Windows 컨테이너로 전환합니다.

![](./media/docker-for-win-switch.png)
> Windows 컨테이너 외에도 LCOW 컨테이너에 대 한 Windows 컨테이너 모드에서는 note 합니다.

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

## <a name="4-run-your-first-windows-container"></a>4. 첫 번째 Windows 컨테이너를 실행 합니다.

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

## <a name="run-your-first-lcow-container"></a>첫 번째 LCOW 컨테이너를 실행 합니다.

이 예제에서는 BusyBox 컨테이너 배포 됩니다. 먼저 ' Hello World' BusyBox 이미지를 실행 하려고 합니다.

```
docker run --rm busybox echo hello_world
```

Note를 Docker 이미지를 가져오려면 하려고 시도할 때 오류가 반환 합니다. Dockers 통해 지시문 필요 하기 때문에이 발생 합니다 `--platform` 플래그를 이미지와 호스트 운영 체제는 적절 하 게 일치 하는지 확인 합니다. Windows 컨테이너 모드에서 기본 플랫폼 Windows 이므로 추가 `--platform linux` 플래그를 가져와 컨테이너를 실행 합니다.

```
docker run --rm --platform linux busybox echo hello_world
```

표시 되는 플랫폼을 사용 하 여 이미지가 끌어온 후의 `--platform` 플래그는 더 이상 필요 합니다. 이 테스트 하려면 없이 명령을 실행 합니다.

```
docker run --rm busybox echo hello_world
```

실행 `docker images` 설치 된 이미지의 목록을 반환 합니다. 이 경우 Windows와 Linux 이미지입니다.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>다음 단계

보너스: LCOW 실행에서 Docker의 해당 [블로그 게시물](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) 을 참조합니다

다음 자습서로 넘어가서 [샘플 앱 빌드](./building-sample-app.md) 예제를 살펴봅니다.
