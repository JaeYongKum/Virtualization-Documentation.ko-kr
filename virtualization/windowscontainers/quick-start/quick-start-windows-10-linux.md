---
title: Windows 10의 Windows 및 Linux 컨테이너
description: 컨테이너 배포 빠른 시작
keywords: Docker, 컨테이너, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: tutorial
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: b94f71949ad2e6c7d0164e3afdfc479fde4262b8
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87984737"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10의 Linux 컨테이너

이 연습에서는 Windows 10에서 Linux 컨테이너를 만들고 실행하는 방법을 안내합니다.

이 빠른 시작에서는 다음을 수행합니다.

1. Docker Desktop 설치
2. 간단한 Linux 컨테이너 실행

이 빠른 시작은 Windows 10에만 해당합니다. 추가 빠른 시작 설명서는 이 페이지 왼쪽에 있는 목차에서 확인할 수 있습니다.

## <a name="prerequisites"></a>필수 구성 요소

다음 요구 사항을 충족하는지 확인하세요.
- Windows 10 Professional, Windows 10 Enterprise 또는 Windows Server 2019 버전 1809 이상을 실행하는 물리적 컴퓨터 시스템 1대
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)가 사용하도록 설정되었습니다.

## <a name="install-docker-desktop"></a>Docker Desktop 설치

[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)을 다운로드하고 설치 관리자를 실행합니다 (로그인 필요. 계정이 없는 경우 하나 만들어야 함). [자세한 설치 지침](https://docs.docker.com/docker-for-windows/install)은 Docker 설명서에 제공됩니다.

## <a name="run-your-first-linux-container"></a>첫 번째 Linux 컨테이너 실행

Linux 컨테이너를 실행하기 위해서는 Docker가 올바른 디먼을 대상으로 하는지 확인해야 합니다. 시스템 트레이에서 Docker 고래 아이콘을 클릭하면 작업 메뉴에서 `Switch to Linux Containers`를 선택하여 이를 토글할 수 있습니다. `Switch to Windows Containers`가 표시되면 이미 Linux 디먼을 대상으로 하고 있는 것입니다.

!["Windows 컨테이너로 전환" 명령을 보여주는 Docker 시스템 트레이 메뉴](./media/switchDaemon.png)

올바른 디먼을 대상으로 하는 것을 확인했으면 다음 명령으로 컨테이너를 실행합니다.

```console
docker run --rm busybox echo hello_world
```

컨테이너는 실행되어 "hello_world"를 인쇄한 다음, 종료합니다.

`docker images`를 쿼리하면 방금 실행한 Linux 컨테이너 이미지가 표시됩니다.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [샘플 앱을 빌드하는 방법 알아보기](./building-sample-app.md)
