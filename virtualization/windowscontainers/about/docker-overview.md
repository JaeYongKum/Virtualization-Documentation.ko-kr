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
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910793"
---
# <a name="about-docker"></a>Docker 정보

컨테이너에 대해 공부하다 보면 Docker에 대한 이야기가 반드시 나오게 됩니다. Docker 엔진은 컨테이너 이미지를 패키징하고 전달하는 컨테이너 관리 도구 세트입니다. 그 결과로 얻는 이미지는 온-프레미스, 클라우드, 개인용 머신 등 어디에서 컨테이너로 실행할 수 있습니다.

![](media/docker.png)

다른 컨테이너와 마찬가지로 [Docker](https://www.docker.com)를 사용하여 Windows Server 컨테이너를 관리할 수 있습니다.

컨테이너를 통한 네임스페이스 격리 및 리소스 관리라는 개념은 BSD Jails, Solaris Zones 및 기본 UNIX 루트 변경(chroot) 메커니즘으로 거슬러 올라가 오래전부터 사용되었습니다. Docker는 애플리케이션의 컨테이너화 및 배포를 간소화하는 공통 도구 세트, 패키징 모델 및 배포 메커니즘을 통해 개발을 위한 견고한 기반을 마련합니다. 이러한 애플리케이션은 모든 Linux 호스트와 Windows 어디서나 실행할 수 있습니다.

유비쿼터스 패키징 모델 및 배포 기술은 모든 호스트에서 동일한 관리 명령을 제공함으로써 관리를 간소화할 뿐 아니라 원활한 DevOps를 위한 고유한 기회를 창출합니다. 개발자의 데스크톱, 테스트 머신, 프로덕션 머신 등에서 몇 초 만에 모든 환경 전체에 동일하게 배포되는 Docker 이미지를 만들 수 있습니다. 덕분에 Docker가 관리하는 컨테이너화된 공개 애플리케이션 레지스트리인 DockerHub를 통해, Docker 컨테이너 안에 패키징된 애플리케이션의 거대한 생태계가 탄생했으며 지속적으로 확장되고 있습니다.

이제 이러한 애플리케이션 생태계와 Docker 개념을 바탕으로 자신의 요구에 부합하는 개발 및 배포 워크플로를 만드는 방법을 살펴보겠습니다.

## <a name="get-started-with-docker"></a>Docker 시작

Docker를 사용하여 컨테이너를 만드는 방법을 알아보려면 [Windows의 Docker 엔진](../manage-docker/configure-docker-daemon.md)을 참조하세요. [Docker 웹 사이트](https://www.docker.com)를 방문하여 Docker 사용 방법에 대해 자세히 알아볼 수도 있습니다.