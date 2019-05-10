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
ms.openlocfilehash: 357fc101b2b0e4d6ccdf53a948ab8d91d19a1522
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621571"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10에서 Linux 컨테이너

> [!div class="op_single_selector"]
> - [Windows의 Linux 컨테이너](quick-start-windows-10-linux.md)
> - [Windows에서 Windows 컨테이너](quick-start-windows-10.md)

이 연습 만들기 및 Windows 10에서 Linux 컨테이너를 실행 하는 방법을 안내.

이 빠른 시작에서 수행 합니다.

1. Windows 용 Docker를 설치
2. Windows (LCOW)에서 Linux 컨테이너를 사용 하 여 간단한 Linux 컨테이너를 실행

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는이 페이지의 왼쪽에 있는 목차에서 찾을 수 있습니다.

## <a name="prerequisites"></a>필수 구성 요소

다음 요구 사항을 충족 하는지 확인 하세요.
- Windows 10 Professional 또는 Enterprise가 크리에이터 스 업데이트 (버전 1709) 이상을 실행 하는 하나의 물리적 컴퓨터 시스템
- [Hyper-v가](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 활성화 되어 있는지 확인 합니다.

***Hyper-v 격리:*** Windows의 Linux 컨테이너 개발자에 게 제공 하기 위해 Windows 10에서 Hyper-v 격리가 필요는 적절 한 Linux 커널을 컨테이너를 실행 합니다. 더 Hyper-v에 대 한 격리에 [대 한 Windows 컨테이너](../about/index.md) 페이지에서 찾을 수 있습니다.

## <a name="install-docker-for-windows"></a>Windows 용 Docker 설치

[Windows 용 Docker를](https://store.docker.com/editions/community/docker-ce-desktop-windows) 다운로드 하 고 (해야 합니다에 로그인 설치 관리자 실행 합니다. 계정이 생성 이미 계정이 없는 경우). [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

> Docker 설치를 이미 있는 경우 버전 LCOW 지원 하기 위해 18.02 이상 있는지 확인 합니다. 검사를 실행 하 여 `docker -v` 또는 *Docker에 대 한*검사 합니다.

> LCOW 컨테이너를 실행 하려면 *Docker 설정 gt_ 디먼에서* 에서 '실험적 기능' 옵션을 활성화 해야 합니다.

## <a name="run-your-first-lcow-container"></a>첫 번째 LCOW 컨테이너 실행

이 예제에서는 BusyBox 컨테이너 배포 됩니다. 먼저 ' Hello World' BusyBox 이미지를 실행 하려고 합니다.

```console
docker run --rm busybox echo hello_world
```

Note를 Docker 이미지를 가져오려면 시도할 때 오류가 반환 합니다. Dockers 지시문을 통해 필요 하기 때문에이 발생 합니다 `--platform` 플래그를 이미지와 호스트 운영 체제는 적절 하 게 일치 하는지 확인 합니다. Windows 컨테이너 모드에서 기본 플랫폼은 Windows 이므로 추가 `--platform linux` 플래그를 가져와 컨테이너를 실행 합니다.

```console
docker run --rm --platform linux busybox echo hello_world
```

플랫폼에 표시 된 이미지가 끌어온 후의 `--platform` 플래그는 더 이상 필요 합니다. 이 테스트 하려면 없이 명령을 실행 합니다.

```console
docker run --rm busybox echo hello_world
```

실행 `docker images` 설치 된 이미지의 목록을 반환 합니다. 이 경우 Windows와 Linux 이미지.

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
> [샘플 응용 프로그램을 빌드하는 방법을 알아봅니다.](./building-sample-app.md)