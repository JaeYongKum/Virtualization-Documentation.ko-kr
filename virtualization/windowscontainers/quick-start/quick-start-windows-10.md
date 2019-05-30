---
title: Windows 10의 windows 및 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: docker, 컨테이너, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: ae311ecccdfbfc30b1079330a8eb02c1ce3ac94b
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680973"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10의 Windows 컨테이너

> [!div class="op_single_selector"]
> - [Windows의 Linux 컨테이너](quick-start-windows-10-linux.md)
> - [Windows의 windows 컨테이너](quick-start-windows-10.md)

이 연습에서는 Windows 10에서 Windows 컨테이너를 만들고 실행 하는 과정을 안내 합니다.

이 빠른 시작에서는 다음을 수행 합니다.

1. Docker 데스크톱 설치
2. 간단한 Windows 컨테이너 실행

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는이 페이지의 왼쪽에 있는 목차에 나와 있습니다.

## <a name="prerequisites"></a>필수 구성 요소
다음 요구 사항을 충족 하는지 확인 하세요.
- Windows 10 Professional 또는 Enterprise for 기념일 업데이트 (버전 1607) 이상을 실행 하는 물리적 컴퓨터 시스템 한 대 
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 가 사용 하도록 설정 되어 있는지 확인 합니다.

***Hyper-v 격리:*** Windows 10에서 Hyper-v를 격리 해야 하는 경우 프로덕션에 사용할 수 있는 것과 동일한 커널 버전 및 구성을 개발자에 게 제공 하려면 [Windows 정보 컨테이너](../about/index.md) 페이지에서 hyper-v 격리에 대해 자세히 알아보세요.

> [!NOTE]
> Windows 10 월 업데이트 2018 릴리스에서는 더 이상 사용자가 개발자/테스트 목적으로 Windows 10 Enterprise 또는 Professional의 프로세스 격리 모드에서 Windows 컨테이너를 실행할 수 없도록 허용 하지 않습니다. 자세히 알아보려면 [FAQ](../about/faq.md) 를 참조 하세요.

## <a name="install-docker-desktop"></a>Docker 데스크탑 설치

[Docker 데스크톱](https://store.docker.com/editions/community/docker-ce-desktop-windows) 을 다운로드 하 고 설치 관리자를 실행 합니다 (로그인이 필요 함). 아직 계정이 없는 경우 계정을 만듭니다. [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

## <a name="switch-to-windows-containers"></a>Windows 컨테이너로 전환

설치 후 Docker 데스크톱은 기본적으로 Linux 컨테이너를 실행 합니다. Docker 트레이 메뉴 중 하나를 사용 하거나 PowerShell 프롬프트에서 다음 명령을 실행 하 여 Windows 컨테이너로 전환 합니다.

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>기본 컨테이너 이미지 설치

Windows 컨테이너는 기본 이미지로 만듭니다. 다음 명령은 Nano Server 기본 이미지를 가져옵니다.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

이미지를 가져온 후 `docker images` 명령을 실행하면 설치된 이미지(여기서는 Nano Server 이미지) 목록이 반환됩니다.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Windows 컨테이너 OS 이미지 [EULA](../images-eula.md)를 참조 하세요.

## <a name="run-your-first-windows-container"></a>첫 번째 Windows 컨테이너 실행

이 간단한 예제에서는 ' Hello World ' 컨테이너 이미지가 만들어지고 배포 됩니다. 최상의 환경을 유지하려면 관리자 권한 Windows CMD 셸 또는 PowerShell에서 이 명령을 실행합니다.

> Windows PowerShell ISE는 컨테이너가 있는 대화형 세션에서 작동하지 않습니다. 컨테이너가 실행 중인 경우에도 중단된 것으로 표시됩니다.

먼저 `nanoserver` 이미지에서 대화형 세션을 사용하여 컨테이너를 시작합니다. 컨테이너가 시작되면 컨테이너 내에서 명령 셸이 표시됩니다.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

컨테이너 내에서 간단한 ' Hello World ' 텍스트 파일을 만듭니다.

```cmd
echo "Hello World!" > Hello.txt
```   

만들었으면 컨테이너를 종료합니다.

```cmd
exit
```

이제 수정된 컨테이너에서 새 컨테이너 이미지를 만듭니다. 컨테이너 목록을 보려면 다음을 실행하고 컨테이너 ID를 적어둡니다.

```console
docker ps -a
```

다음 명령을 실행하여 새 ‘HelloWorld’ 이미지를 만듭니다. <containerid>를 컨테이너 ID로 바꿉니다.

```console
docker commit <containerid> helloworld
```

완료되면 hello world 스크립트가 포함된 사용자 지정 이미지가 생깁니다. 이 이미지를 확인하려면 다음 명령을 사용합니다.

```console
docker images
```

마지막으로 컨테이너를 실행하려면 `docker run` 명령을 사용입니다.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

`docker run` 명령의 결과는 hyper-v 격리에서 실행 되는 컨테이너가 ' HelloWorld ' 이미지에서 만들어졌으며, cmd의 인스턴스가 컨테이너에서 시작 되 고 파일 읽기 (셸에 게 에코 된 출력)가 실행 된 후 컨테이너 중지 및 제거

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 앱을 작성 하는 방법 알아보기](./building-sample-app.md)
