---
title: 약 Docker
description: Docker에 대해 알아보세요.
keywords: Docker, 컨테이너
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: cdb7771a1293e7c3fd505103f0010bdfba47cc31
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674889"
---
# <a name="about-docker"></a>약 Docker

컨테이너에 대해 공부하다 보면 Docker에 대한 이야기가 반드시 나오게 됩니다. Docker 엔진은 컨테이너 이미지를 패키지화 하 고 전달 하는 컨테이너 관리 도구 집합입니다. 결과 이미지는 온-프레미스, 클라우드 또는 개인 컴퓨터에 관계 없이 컨테이너로 실행할 수 있습니다.

![](media/docker.png)

다른 모든 컨테이너 처럼 [Docker](https://www.docker.com) 를 사용 하 여 Windows Server 컨테이너를 관리할 수 있습니다.

컨테이너를 통한 네임 스페이스 격리 및 리소스 관리의 개념은 오랜 시간 동안, BSD Jails, Solaris 영역, 기본 UNIX 변경 루트 (chroot) 메커니즘으로 진행 되었습니다. Docker는 응용 프로그램의 containerization 배포를 간소화 하는 공통 도구 집합, 패키징 모델 및 배포 메커니즘을 통해 개발을 위한 견고한 기반을 만듭니다. 이러한 응용 프로그램은 모든 Linux 호스트와 Windows에서 어디서 나 실행할 수 있습니다.

함께 사용할 수 있는 패키징 모델 및 배포 기술은 모든 호스트에 동일한 관리 명령을 제공 하 여 관리를 간소화 하 여 원활한 DevOps에 대 한 고유한 기회를 만듭니다. 또한 모든 환경에서 동일 하 게 배포 되는 Docker 이미지 (개발자 데스크톱, 테스트 컴퓨터 또는 프로덕션 컴퓨터 집합)를 만들 수 있습니다. 이는 docker가 관리 하는 공개 containerized 응용 프로그램 레지스트리 인 DockerHub를 사용 하 여 Docker 및 확대 된 응용 프로그램의 광범위 한 환경을 만들었습니다.

이제 응용 프로그램의 해당 에코 시스템에 대해 설명 하 고, 사용자의 요구에 맞는 개발 및 배포 워크플로를 만들기 위해 Docker 개념을 기반으로 구축 하는 방법에 대해 알아보겠습니다.

## <a name="get-started-with-docker"></a>Docker 시작 하기

Docker를 사용 하 여 컨테이너를 작성 하는 방법을 알아보려면 [Windows의 Docker 엔진](../manage-docker/configure-docker-daemon.md)을 참조 하세요. Docker를 사용 하는 방법에 대 한 자세한 내용은 [docker 웹 사이트를](https://www.docker.com) 방문 하 여 자세히 살펴볼 수도 있습니다.