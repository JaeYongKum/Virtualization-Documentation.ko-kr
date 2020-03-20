---
title: Windows 10의 Windows 및 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: Docker, 컨테이너, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909583"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10의 Linux 컨테이너

이 연습에서는 Windows 10에서 Linux 컨테이너를 만들고 실행하는 방법을 안내합니다.

이 빠른 시작에서는 다음을 수행합니다.

1. Docker Desktop 설치
2. Windows에서 Linux 컨테이너를 사용하여 간단한 Linux 컨테이너 실행(LCOW)

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

## <a name="prerequisites"></a>필수 구성 요소

다음 요구 사항을 충족하는지 확인하세요.
- Windows 10 Professional, Windows 10 Enterprise 또는 Windows Server 2019 버전 1809 이상을 실행하는 물리적 컴퓨터 시스템 1대
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)가 사용하도록 설정되었습니다.

***Hyper-V 격리:*** Windows의 Linux 컨테이너에서 개발자에게 컨테이너를 실행하는 적절한 Linux 커널을 제공하려면 Windows 10에서 Hyper-V 격리가 필요합니다. Hyper-V 격리에 대한 자세한 내용은 [Windows 컨테이너 정보](../about/index.md) 페이지에서 찾을 수 있습니다.

## <a name="install-docker-desktop"></a>Docker Desktop 설치

[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)을 다운로드하고 설치 관리자를 실행합니다 (로그인 필요. 계정이 없는 경우 하나 만들어야 함). [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

> Docker가 이미 설치되어 있는 경우 LCOW를 지원하는 버전 18.02 이상이 설치되어 있는지 확인합니다. `docker -v`를 실행하거나 *Docker 정보*를 검사하여 확인합니다.

> LCOW 컨테이너를 실행하려면 *Docker 설정 > 디먼*의 '실험적 기능' 옵션을 활성화해야 합니다.

## <a name="run-your-first-lcow-container"></a>첫 번째 LCOW 컨테이너 실행

이 예제에서는 BusyBox 컨테이너를 배포합니다. 먼저 'Hello World' BusyBox 이미지를 실행해 봅니다.

```console
docker run --rm busybox echo hello_world
```

Docker에서 이미지를 가져오려고 시도하면 오류가 반환됩니다. Docker는 이미지와 호스트 운영 체제가 적절하게 매칭되는지 확인하기 위해 Docker 플래그를 통해 `--platform` 지시문이 필요하기 때문입니다. Windows 컨테이너 모드의 기본 플랫폼은 Windows이므로, 컨테이너를 풀링하여 실행하도록 `--platform linux` 플래그를 추가합니다.

```console
docker run --rm --platform linux busybox echo hello_world
```

표시된 플랫폼으로 이미지가 풀링되면 `--platform` 플래그가 더 이상 필요하지 않습니다. 플래그 없이 명령을 실행하여 테스트합니다.

```console
docker run --rm busybox echo hello_world
```

`docker images`를 실행하여 설치된 이미지 목록을 반환합니다. 이 예제에서는 Windows 이미지와 Linux 이미지가 모두 반환됩니다.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 보너스: LCOW 실행과 관련된 Docker의 [블로그 게시물](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)을 참조하세요.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 앱을 빌드하는 방법 알아보기](./building-sample-app.md)
