---
title: Windows 10의 windows 및 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: docker, 컨테이너, LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 3d651a4a68acefa25f1b647b1b33618bbfb91ae9
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129373"
---
# <a name="get-started-run-your-first-container"></a>시작 하기: 첫 번째 컨테이너 실행

[이전 세그먼트](./set-up-environment.md)에서 컨테이너를 실행 하기 위한 환경을 구성 했습니다. 이 연습에서는 컨테이너 이미지를 가져와 실행 하는 방법을 보여 줍니다.

## <a name="install-container-base-image"></a>설치 컨테이너 기본 이미지

모든 컨테이너가 인스턴스화됩니다 `container images`. Microsoft는 여러 개의 "스타터" 이미지 ( `base images`이라고 함)를 제공 하 여 선택할 수 있습니다. 다음 명령은 Nano Server 기본 이미지를 가져옵니다.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!TIP]
> 오류 메시지가 표시 되는 경우에는 `no matching manifest for unknown in the manifest list entries`Docker가 Linux 컨테이너를 실행 하도록 구성 되어 있지 않은지 확인 합니다.

이미지를 끌어온 후 로컬 docker 이미지 리포지토리를 쿼리하여 컴퓨터에 있는지 확인할 수 있습니다. 이 명령을 `docker images` 실행 하면 설치 된 이미지 목록 (이 경우 Nano 서버 이미지)이 반환 됩니다.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Windows 컨테이너 OS 이미지 [EULA](../images-eula.md)를 참조 하세요.

## <a name="run-your-first-windows-container"></a>첫 번째 Windows 컨테이너 실행

이 간단한 예제에서는 ' Hello World ' 컨테이너 이미지가 만들어지고 배포 됩니다. 최상의 환경을 위해서는 관리자 권한 Windows CMD 셸 또는 PowerShell에서 명령을 실행 합니다.

> Windows PowerShell ISE는 컨테이너가 있는 대화형 세션에서 작동하지 않습니다. 컨테이너가 실행 중인 경우에도 중단된 것으로 표시됩니다.

먼저 `nanoserver` 이미지에서 대화형 세션을 사용하여 컨테이너를 시작합니다. 컨테이너를 시작 하면 컨테이너 내에서 명령 셸이 표시 됩니다.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

컨테이너 내에서는 간단한 ' Hello World ' 텍스트 파일을 만듭니다.

```cmd
echo "Hello World!" > Hello.txt
```   

만들었으면 컨테이너를 종료합니다.

```cmd
exit
```

수정 된 컨테이너에서 새 컨테이너 이미지를 만듭니다. 실행 중이거나 종료 된 컨테이너 목록을 보려면 다음을 실행 하 여 컨테이너 id를 기록해 둡니다.

```console
docker ps -a
```

다음 명령을 실행하여 새 ‘HelloWorld’ 이미지를 만듭니다. `<containerid>`를 컨테이너 ID로 바꿉니다.

```console
docker commit <containerid> helloworld
```

완료되면 hello world 스크립트가 포함된 사용자 지정 이미지가 생깁니다. 이 이미지를 확인하려면 다음 명령을 사용합니다.

```console
docker images
```

마지막으로, `docker run` 명령을 사용 하 여 컨테이너를 실행 합니다.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

이 `docker run` 명령 결과는 컨테이너가 ' HelloWorld ' 이미지에서 만들어지고, cmd의 인스턴스가 컨테이너에서 시작 되 고, 파일 (셸에 에코 된 출력)을 읽은 다음 컨테이너를 중지 하 고 제거 하는 것입니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 앱을 containerize 하는 방법 알아보기](./building-sample-app.md)
