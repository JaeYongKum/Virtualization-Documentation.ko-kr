---
title: Windows 10의 Windows 및 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: Docker, 컨테이너, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: quickstart
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 1422b2b449d469867e1b68aab50274f2463c2b0c
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985407"
---
# <a name="get-started-run-your-first-windows-container"></a>시작: 첫 번째 Windows 컨테이너 실행

이 문서에서는 [시작: 컨테이너에 맞게 Windows 준비](./set-up-environment.md)에 설명된 대로 환경을 설정한 후 첫 번째 Windows 컨테이너를 실행하는 방법을 설명합니다. 컨테이너를 실행하려면 먼저 컨테이너에 운영 체제 서비스의 기본 계층을 제공하는 기본 이미지를 설치합니다. 그런 다음, 기본 이미지를 기반으로 컨테이너 이미지를 만들고 실행합니다. 자세한 내용을 보려면 계속 읽으십시오.

## <a name="install-a-container-base-image"></a>컨테이너 기본 이미지 설치

모든 컨테이너는 컨테이너 이미지에서 만들어집니다. Microsoft는 선택할 수 있는 이미지(기본 이미지)를 몇 가지 제공합니다. 자세한 내용은 [컨테이너 기본 이미지](../manage-containers/container-base-images.md)를 참조하세요. 이 절차는 경량 Nano 서버 기본 이미지를 끌어옵니다(다운로드 및 설치).

1. 명령 프롬프트 창(예: 내장된 명령 프롬프트, PowerShell 또는 [Windows 터미널](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab))을 열고 다음 명령을 실행하여 기본 이미지를 다운로드하고 설치합니다.

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > `no matching manifest for unknown in the manifest list entries`라는 오류 메시지가 표시되면 Docker가 Linux 컨테이너를 실행하도록 구성되어 있지 않은지 확인합니다.

2. 이미지 다운로드가 완료되면(기다리는 동안 [EULA](../images-eula.md)를 확인하세요) 로컬 Docker 이미지 리포지토리를 쿼리하여 시스템에 이미지가 있는지 확인합니다. `docker images` 명령을 실행하면 설치된 이미지 목록이 반환됩니다.

   다음은 Nano 서버 이미지를 보여주는 출력의 예입니다.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Windows 컨테이너 실행

여기 간단한 예제에서는 'Hello World' 컨테이너 이미지를 만들고 배포합니다. 최상의 환경을 위해 관리자 권한 명령 프롬프트 창에서 다음 명령을 실행합니다. (단, Windows PowerShell ISE는 사용하지 마십시오. 컨테이너가 중단된 것처럼 보여서 컨테이너와의 대화형 세션에서 작동하지 않습니다.)

1. 명령 프롬프트 창에 다음 명령을 입력하여 `nanoserver` 이미지에서 대화형 세션으로 컨테이너를 시작합니다.

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. 컨테이너가 시작되면 명령 프롬프트 창이 컨텍스트를 컨테이너로 변경합니다. 컨테이너 내에서 다음 명령을 입력하여 간단한 'Hello World'텍스트 파일을 만든 후 컨테이너를 종료합니다.

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```

3. [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) 명령을 실행하여 방금 종료한 컨테이너의 컨테이너 ID를 가져옵니다.

   ```console
   docker ps -a
   ```

4. 처음 실행한 컨테이너의 변경 내용이 포함된 'HelloWorld' 이미지를 새로 만듭니다. 이렇게 하려면 `<containerid>`를 컨테이너의 ID로 바꿔서 [docker commit](https://docs.docker.com/engine/reference/commandline/commit/) 명령을 실행합니다.

   ```console
   docker commit <containerid> helloworld
   ```

   완료되면 hello world 스크립트가 포함된 사용자 지정 이미지가 생깁니다. [docker images](https://docs.docker.com/engine/reference/commandline/images/) 명령으로 볼 수 있습니다.

   ```console
   docker images
   ```

   출력 예는 다음과 같습니다.

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 마지막으로 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 명령을 `--rm`(명령줄(cmd.exe)이 중지되면 컨테이너를 자동으로 제거함) 매개 변수와 함께 사용하여 새 컨테이너를 실행합니다.

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```
   그 결과, Docker는 'HelloWorld' 이미지에서 컨테이너를 만들었고, Docker는 컨테이너에서 cmd.exe 인스턴스를 시작했으며, cmd.exe는 파일을 읽고 콘텐츠를 셸에 출력했습니다. 마지막 단계로 Docker는 컨테이너를 중지하고 제거했습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 앱을 컨테이너화하는 방법 알아보기](./building-sample-app.md)
