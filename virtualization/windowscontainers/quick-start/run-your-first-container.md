---
title: Windows 10의 windows 및 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: docker, 컨테이너, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909573"
---
# <a name="get-started-run-your-first-windows-container"></a>시작 하기: 첫 번째 Windows 컨테이너를 실행 합니다.

이 항목에서는 [시작: 컨테이너에 대해 Windows 준비](./set-up-environment.md)에 설명 된 대로 환경을 설정한 후 첫 번째 Windows 컨테이너를 실행 하는 방법을 설명 합니다. 컨테이너를 실행 하려면 먼저 컨테이너에 운영 체제 서비스의 기본 계층을 제공 하는 기본 이미지를 설치 합니다. 그런 다음 기본 이미지를 기반으로 하는 컨테이너 이미지를 만들고 실행 합니다. 자세한 내용은을 참조 하세요.

## <a name="install-a-container-base-image"></a>컨테이너 기본 이미지 설치

모든 컨테이너는 컨테이너 이미지에서 만들어집니다. Microsoft는 기본 이미지 라고 하는 몇 가지 스타터 이미지를 제공 합니다. 자세한 내용은 [컨테이너 기본 이미지](../manage-containers/container-base-images.md)를 참조 하세요. 이 절차는 경량 Nano Server 기본 이미지를 가져오고 (다운로드 및 설치)

1. 명령 프롬프트 창을 열고 (예: 기본 제공 명령 프롬프트, PowerShell 또는 [Windows 터미널](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)) 다음 명령을 실행 하 여 기본 이미지를 다운로드 하 고 설치 합니다.

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > `no matching manifest for unknown in the manifest list entries`이라는 오류 메시지가 표시 되는 경우 Docker가 Linux 컨테이너를 실행 하도록 구성 되지 않았는지 확인 합니다.

2. 이미지 다운로드가 완료 된 후-대기 하는 동안 [EULA](../images-eula.md) 를 읽으면 로컬 docker 이미지 리포지토리를 쿼리하여 시스템에 있는지 확인 합니다. 명령 `docker images` 실행 하면 설치 된 이미지 목록이 반환 됩니다.

   Nano Server 이미지를 표시 하는 출력의 예는 다음과 같습니다.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Windows 컨테이너 실행

이 간단한 예제에서는 ' Hello World ' 컨테이너 이미지가 만들어지고 배포 됩니다. 최상의 환경을 위해 관리자 권한 명령 프롬프트 창에서 다음 명령을 실행 합니다. Windows PowerShell ISE 그러나 컨테이너가 중단 된 것 처럼 보일 때 컨테이너와의 대화형 세션에서는 작동 하지 않습니다.

1. 명령 프롬프트 창에서 다음 명령을 입력 하 여 `nanoserver` 이미지에서 대화형 세션으로 컨테이너를 시작 합니다.

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. 컨테이너가 시작 되 면 명령 프롬프트 창이 컨텍스트를 컨테이너에 변경 합니다. 컨테이너 내에서 간단한 ' Hello World ' 텍스트 파일을 만든 후 다음 명령을 입력 하 여 컨테이너를 종료 합니다.

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. [Docker ps](https://docs.docker.com/engine/reference/commandline/ps/) 명령을 실행 하 여 방금 종료 한 컨테이너의 컨테이너 ID를 가져옵니다.

   ```console
   docker ps -a
   ```

4. 실행 한 첫 번째 컨테이너의 변경 내용이 포함 된 새 ' HelloWorld ' 이미지를 만듭니다. 이렇게 하려면 [docker commit](https://docs.docker.com/engine/reference/commandline/commit/) 명령을 실행 하 고 `<containerid>`를 컨테이너의 ID로 바꿉니다.

   ```console
   docker commit <containerid> helloworld
   ```

   완료되면 hello world 스크립트가 포함된 사용자 지정 이미지가 생깁니다. 이는 [docker images](https://docs.docker.com/engine/reference/commandline/images/) 명령과 함께 볼 수 있습니다.

   ```console
   docker images
   ```

   출력 예는 다음과 같습니다.

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 마지막으로, 명령줄 (cmd.exe)이 중지 되 면 자동으로 컨테이너를 제거 하는 `--rm` 매개 변수와 함께 [docker run](https://docs.docker.com/engine/reference/commandline/run/) 명령을 사용 하 여 새 컨테이너를 실행 합니다.

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   결과적으로 컨테이너를 ' HelloWorld ' 이미지에서 만들었기 때문에 파일을 읽고 파일 콘텐츠를 셸에 출력 하는 컨테이너에서 cmd.exe의 인스턴스가 시작 된 후 컨테이너를 중지 하 고 제거 했습니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 앱을 컨테이너 화 하는 방법 알아보기](./building-sample-app.md)
