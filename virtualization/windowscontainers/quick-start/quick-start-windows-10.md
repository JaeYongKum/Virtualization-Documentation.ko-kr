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
ms.openlocfilehash: 07f5929505226a50a161b4ae7df5669c2ad89d83
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575524"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10의 Windows 컨테이너

> [!div class="op_single_selector"]
> - [Windows의 Linux 컨테이너](quick-start-windows-10-linux.md)
> - [Windows에서 Windows 컨테이너](quick-start-windows-10.md)

이 연습 생성 및 Windows 10에서 Windows 컨테이너를 실행 하는 방법을 안내.

이 빠른 시작에서 수행 합니다.

1. Windows 용 Docker 설치
2. 간단한 Windows 컨테이너를 실행

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는이 페이지의 왼쪽에 있는 목차에서 찾을 수 있습니다.

## <a name="prerequisites"></a>필수 구성 요소
다음 요구 사항을 충족 하는지 확인 하세요.
- Windows 10 Professional 또는 Enterprise 1 주년 업데이트 (버전 1607) 이상을 실행 물리적 컴퓨터 시스템 하나. 
- [Hyper-v가](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 활성화 되어 있는지 확인 합니다.

***Hyper-v 격리:*** Windows Server 컨테이너는 동일한 커널 버전 및 프로덕션 환경에서 사용할 수 있는 구성을 개발자에 게 제공 하기 위해 Windows 10에서 Hyper-v 격리가 필요, 더 Hyper-v에 대 한 격리에서 찾을 수 있습니다 [Windows 컨테이너 정보](../about/index.md) 페이지.

> [!NOTE]
> Windows 업데이트 2018 년 10 월 릴리스에서 더 이상 사용자가 개발자/테스트를 위해 Windows 10 Enterprise 또는 Professional에서 Windows 컨테이너 프로세스 격리 모드에서 실행을 허용 했습니다. 자세한 내용은 [FAQ](../about/faq.md) 를 참조 하세요.

## <a name="install-docker-for-windows"></a>Windows 용 Docker 설치

[Windows 용 Docker를](https://store.docker.com/editions/community/docker-ce-desktop-windows) 다운로드 하 고 (해야 합니다에 로그인 설치 관리자 실행 합니다. 계정이 생성 이미 계정이 없는 경우). [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

## <a name="switch-to-windows-containers"></a>Windows 컨테이너로 전환

설치가 완료되면 Windows용 Docker는 기본적으로 Linux 컨테이너를 실행하도록 설정됩니다. Windows 컨테이너 중 하나를 사용 하 여 Docker 트레이 메뉴 전환 하거나 PowerShell에서 다음 명령을 실행 하 여 메시지를 표시 합니다.

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
> Windows 컨테이너 OS 이미지 [EULA](../images-eula.md)를 읽어보세요.

## <a name="run-your-first-windows-container"></a>첫 번째 Windows 컨테이너를 실행 합니다.

이 간단한 예제에서는 ' Hello World' 컨테이너 이미지는 생성 되 고 배포 합니다. 최상의 환경을 유지하려면 관리자 권한 Windows CMD 셸 또는 PowerShell에서 이 명령을 실행합니다.

> Windows PowerShell ISE는 컨테이너가 있는 대화형 세션에서 작동하지 않습니다. 컨테이너가 실행 중인 경우에도 중단된 것으로 표시됩니다.

먼저 `nanoserver` 이미지에서 대화형 세션을 사용하여 컨테이너를 시작합니다. 컨테이너가 시작되면 컨테이너 내에서 명령 셸이 표시됩니다.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

컨테이너 내에서 간단한 ' Hello World' 텍스트 파일을를 만듭니다.

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

결과 `docker run` 명령에는 'HelloWorld' 이미지에서 Hyper-v 격리에서 실행 되는 컨테이너를 만들었습니다, cmd의 인스턴스를 컨테이너에서 시작 하 고 실행 (출력은 셸에 에코)는 파일을 선택한 다음 컨테이너의 읽기 중지 하 고 제거 합니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 응용 프로그램을 빌드하는 방법을 알아봅니다.](./building-sample-app.md)