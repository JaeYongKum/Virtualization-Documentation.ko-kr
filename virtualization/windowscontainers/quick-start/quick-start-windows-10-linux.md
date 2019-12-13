---
title: Windows 10의 windows 및 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: docker, 컨테이너, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909583"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10의 Linux 컨테이너

이 연습에서는 Windows 10에서 Linux 컨테이너를 만들고 실행 하는 과정을 안내 합니다.

이 빠른 시작에서는 다음을 수행 합니다.

1. Docker Desktop 설치
2. Windows에서 Linux 컨테이너를 사용 하 여 간단한 Linux 컨테이너 실행 (LCOW)

이 빠른 시작은 Windows 10에만 해당합니다. 이 페이지의 왼쪽에 있는 목차에서 추가 빠른 시작 설명서를 찾을 수 있습니다.

## <a name="prerequisites"></a>필수 구성 요소

다음 요구 사항을 충족 하는지 확인 하세요.
- Windows 10 Professional, Windows 10 Enterprise 또는 Windows Server 2019 버전 1809 이상을 실행 하는 물리적 컴퓨터 시스템 1 대
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 를 사용 하도록 설정 했는지 확인 합니다.

***Hyper-v 격리:*** Windows의 linux 컨테이너는 개발자에 게 컨테이너를 실행 하는 데 적절 한 Linux 커널을 제공 하기 위해 Windows 10에서 Hyper-v 격리가 필요 합니다. Hyper-v 격리에 대 한 자세한 내용은 [Windows 컨테이너 정보](../about/index.md) 페이지에서 찾을 수 있습니다.

## <a name="install-docker-desktop"></a>Docker Desktop 설치

[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) 을 다운로드 하 고 설치 관리자를 실행 합니다 (로그인 해야 함). 계정이 아직 없는 경우 계정을 만듭니다. [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

> Docker가 이미 설치 되어 있는 경우 LCOW을 지원 하려면 버전 18.02 이상이 설치 되어 있는지 확인 합니다. `docker -v`를 실행 하거나 *Docker 정보*를 확인 하 여 확인 합니다.

> LCOW 컨테이너를 실행 하려면 *Docker 설정 > 디먼* 의 ' 실험적 기능 ' 옵션을 활성화 해야 합니다.

## <a name="run-your-first-lcow-container"></a>첫 번째 LCOW 컨테이너 실행

이 예에서는 BusyBox 컨테이너가 배포 됩니다. 먼저 ' Hello World ' BusyBox 이미지를 실행 해 봅니다.

```console
docker run --rm busybox echo hello_world
```

Docker에서 이미지를 가져오려고 하면 오류가 반환 됩니다. 이는 이미지 및 호스트 운영 체제가 적절 하 게 일치 하는지 확인 하기 위해 Docker 플래그를 `--platform` 통해 지시문이 필요 하기 때문에 발생 합니다. Windows 컨테이너 모드의 기본 플랫폼은 Windows 이므로 컨테이너를 풀 하 고 실행 하는 `--platform linux` 플래그를 추가 합니다.

```console
docker run --rm --platform linux busybox echo hello_world
```

표시 된 플랫폼과 함께 이미지를 끌어온 후에는 `--platform` 플래그가 더 이상 필요 하지 않습니다. 이를 테스트 하지 않고 명령을 실행 합니다.

```console
docker run --rm busybox echo hello_world
```

`docker images`를 실행 하 여 설치 된 이미지 목록을 반환 합니다. 이 경우 Windows 및 Linux 이미지가 모두 있습니다.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 보너스: LCOW 실행에서 Docker의 해당 [블로그 게시물](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) 을 참조 하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 앱을 빌드하는 방법 알아보기](./building-sample-app.md)
