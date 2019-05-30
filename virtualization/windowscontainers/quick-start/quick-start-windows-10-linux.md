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
ms.openlocfilehash: 91031f9394cb3fcb1af6c4813f8805ad6f79bf8c
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681103"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10의 Linux 컨테이너

> [!div class="op_single_selector"]
> - [Windows의 Linux 컨테이너](quick-start-windows-10-linux.md)
> - [Windows의 windows 컨테이너](quick-start-windows-10.md)

이 연습에서는 Windows 10에서 Linux 컨테이너를 만들고 실행 하는 과정을 안내 합니다.

이 빠른 시작에서는 다음을 수행 합니다.

1. Docker 데스크톱 설치
2. Windows에서 Linux 컨테이너를 사용 하 여 간단한 Linux 컨테이너 실행 (LCOW)

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는이 페이지의 왼쪽에 있는 목차에 나와 있습니다.

## <a name="prerequisites"></a>필수 구성 요소

다음 요구 사항을 충족 하는지 확인 하세요. <<<<<<< 머리
- Windows 10 Professional 또는 Enterprise를 실행 하는 단일 물리적 컴퓨터 시스템 (낙하 크리에이터 업데이트 (버전 1709) 이상
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 가 사용 하도록 설정 되어 있는지 확인 합니다.
=======
- Windows 10 Professional, Windows 10 Enterprise 또는 Windows Server 2019 버전 1809 이상을 실행 하는 물리적 컴퓨터 시스템 한 대
- [Hyper-v](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 가 사용 하도록 설정 되어 있는지 확인 합니다.
>>>>>>> 원본/마스터

***Hyper-v 격리:*** 개발자에 게 컨테이너를 실행 하는 적절 한 Linux 커널을 제공 하려면 Windows의 Linux 컨테이너에서 Windows 10에 대 한 Hyper-v 격리가 필요 합니다. Hyper-v 격리에 대 한 자세한 내용은 [Windows 컨테이너 정보](../about/index.md) 페이지에 나와 있습니다.

## <a name="install-docker-desktop"></a>Docker 데스크탑 설치

[Docker 데스크톱](https://store.docker.com/editions/community/docker-ce-desktop-windows) 을 다운로드 하 고 설치 관리자를 실행 합니다 (로그인이 필요 함). 아직 계정이 없는 경우 계정을 만듭니다. [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

> 이미 Docker가 설치 되어 있는 경우에는 LCOW를 지원 하기 위해 버전 18.02 이상이 있는지 확인 합니다. Docker를 실행 `docker -v` 하거나 확인 ** 하 여 확인 합니다.

> LCOW 컨테이너를 실행 하려면 *Docker 설정 _GT_ 데몬에* 대 한 ' 실험적 기능 ' 옵션을 활성화 해야 합니다.

## <a name="run-your-first-lcow-container"></a>첫 번째 LCOW 컨테이너를 실행 합니다.

이 예제에서는 BusyBox 컨테이너가 배포 됩니다. 먼저 ' Hello World ' BusyBox image를 실행 해 봅니다.

```console
docker run --rm busybox echo hello_world
```

이는 Docker가 이미지를 가져오려고 할 때 오류가 반환 된다는 점에 유의 하세요. 이는 이미지와 호스트 운영 체제가 적절 하 `--platform` 게 일치 하는 것을 확인 하기 위해 Dockers에서 플래그를 통해 지시문이 필요 하기 때문에 발생 합니다. Windows 컨테이너 모드의 기본 플랫폼은 Windows 이므로 컨테이너를 가져오고 실행할 `--platform linux` 플래그를 추가 합니다.

```console
docker run --rm --platform linux busybox echo hello_world
```

플랫폼이 표시 된 이미지를 끌어온 후에는 `--platform` 플래그가 더 이상 필요 하지 않습니다. 이를 테스트 하지 않고 명령을 실행 합니다.

```console
docker run --rm busybox echo hello_world
```

설치 `docker images` 된 이미지 목록을 반환 하려면 실행 합니다. 이 경우에는 Windows 및 Linux 이미지를 모두 지정 합니다.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 보너스: LCOW를 실행 하는 경우 Docker의 해당 [블로그 게시물](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) 을 참조 하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 앱을 작성 하는 방법 알아보기](./building-sample-app.md)
