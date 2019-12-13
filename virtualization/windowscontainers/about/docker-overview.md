---
title: Docker 정보
description: Docker에 대해 알아봅니다.
keywords: Docker, 컨테이너
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: cdb7771a1293e7c3fd505103f0010bdfba47cc31
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910793"
---
# <a name="about-docker"></a>Docker 정보

컨테이너에 대해 공부하다 보면 Docker에 대한 이야기가 반드시 나오게 됩니다. Docker 엔진은 컨테이너 이미지를 패키지 하 고 전달 하는 컨테이너 관리 도구 집합입니다. 결과 이미지는 온-프레미스, 클라우드 또는 개인용 컴퓨터에 있든 관계 없이 어디에서 나 실행할 수 있습니다.

![](media/docker.png)

다른 컨테이너와 마찬가지로 [Docker](https://www.docker.com) 를 사용 하 여 Windows Server 컨테이너를 관리할 수 있습니다.

컨테이너를 통한 네임 스페이스 격리 및 리소스 관리 개념은 BSD Jails, Solaris Zones 및 기본 UNIX 변경 루트 (chroot) 메커니즘으로 돌아가서 오랜 시간 동안 발생 했습니다. Docker는 응용 프로그램의 컨테이너 화 및 배포를 간소화 하는 일반적인 도구 집합, 패키징 모델, 배포 메커니즘을 통해 개발을 위한 견고한 기반을 만듭니다. 이러한 응용 프로그램은 모든 Linux 호스트 및 Windows의 어디에서 나 실행할 수 있습니다.

모든 호스트에 대해 동일한 관리 명령을 제공 하 고 원활한 DevOps를 위한 고유한 기회를 만들어 관리를 간소화 하는 패키징 모델 및 배포 기술입니다. 또한 개발자의 데스크톱, 테스트 컴퓨터 또는 일련의 프로덕션 컴퓨터와 상관 없이 모든 환경에서 동일 하 게 배포 되는 Docker 이미지를 만들 수 있습니다. Docker에서 관리 하는 공용 컨테이너 화 된 응용 프로그램 레지스트리 인 DockerHub를 사용 하 여 Docker 컨테이너에 패키지 된 대규모 응용 프로그램 에코 시스템을 만들었습니다.

이제 응용 프로그램의 에코 시스템에 대해 설명 하 고, 요구 사항에 적합 한 개발 및 배포 워크플로를 만들기 위해 Docker 개념에 대해 빌드하는 방법을 알아보겠습니다.

## <a name="get-started-with-docker"></a>Docker 시작

Docker를 사용 하 여 컨테이너를 빌드하는 방법에 대 한 자세한 내용은 [Windows의 Docker 엔진](../manage-docker/configure-docker-daemon.md)을 참조 하세요. Docker를 사용 하는 방법을 자세히 살펴보면 [docker 웹 사이트를](https://www.docker.com) 방문할 수도 있습니다.