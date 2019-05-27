---
title: Windows 컨테이너 정보
description: Windows 컨테이너에 대해 알아봅니다.
keywords: Docker, 컨테이너
author: taylorb-microsoft
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 80514884b4c95657f63cf585ece6aa8c8b23cc44
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674728"
---
# <a name="about-windows-containers"></a>Windows 컨테이너 정보

부엌을 상상해 보세요. 이 단일 채팅방에는 요리, 이동, 싱크 등의 식사에 필요한 모든 것이 포함 됩니다. 이는 컨테이너입니다.

![검정색 상자 안에 노란색 배경 무늬가 있는 완전 하 게 제공 된 주방 그림입니다.](media/box1.png)

이번에는 책을 bookshelf으로 이동 하는 것 처럼 건물 내에 주방을 배치 한다고 가정해 보겠습니다. 부엌이 작동 해야 하는 모든 것이 이미 있기 때문에, 전기 및 배관을 연결 하는 것은 요리를 시작 해야 한다는 것입니다.

![두 개의 블랙 박스 스택으로 이루어진 아파트 건물입니다. 이러한 상자 중 4 개는 주방 예에 사용 된 것과 동일한 노란색 상자로, 나머지는 다양 한 장소에 있으며, 비어 있거나 회색으로 표시 되어 있습니다.](media/apartment.png)

이 문제가 발생 하는 이유는 무엇 인가요? 건물을 원하는 대로 사용자 지정할 수 있습니다. 여러 종류의 채팅방을 채우거 나, 같은 채팅방에 기입 하거나, 두 가지를 혼합 하 여 사용할 수 있습니다.

컨테이너는 부엌에 따라 앱을 실행 하 여이 채팅방 처럼 작동 합니다. 컨테이너는 앱과 앱이 자체 격리 된 상자에 실행 되어야 하는 모든 것을 배치 합니다. 따라서 격리 된 앱은 해당 컨테이너 외부에 존재 하는 다른 앱 또는 프로세스에 대해 알지 못합니다. 컨테이너에 앱을 실행 해야 하는 모든 항목이 있으므로 컨테이너는 다른 컨테이너에 대해 프로 비전 된 리소스를 건드리지 않고 호스트 규정의 리소스만 사용 하 여 어디서 든 이동할 수 있습니다.

다음 비디오에서는 Windows 컨테이너에서 수행할 수 있는 작업에 대해 자세히 설명 하 고, Microsoft에서 Docker와의 파트너 관계가 열려 있는 컨테이너 개발에 대 한 frictionless를 만드는 데 도움을 줍니다.

<iframe width="800" height="450" src="https://www.youtube.com/embed/Ryx3o0rD5lY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## <a name="container-fundamentals"></a>컨테이너 기본 사항

Windows 컨테이너 작업을 시작할 때 유용한 몇 가지 용어에 대해 알아보겠습니다.

- 컨테이너 호스트: Windows 컨테이너 기능을 사용 하 여 구성 된 실제 또는 가상 컴퓨터 시스템입니다. 컨테이너 호스트는 하나 이상의 Windows 컨테이너를 실행 합니다.
- 샌드박스: 실행 되는 동안 컨테이너에 대해 변경한 모든 내용을 캡처하는 레이어입니다 (파일 시스템 수정, 레지스트리 수정 또는 소프트웨어 설치 등).
- 기본 이미지: 컨테이너의 운영 체제 환경을 제공 하는 컨테이너의 이미지 계층에 있는 첫 번째 레이어입니다. 기본 이미지를 수정할 수 없습니다.
- 컨테이너 이미지: 컨테이너 만들기에 대 한 지침의 읽기 전용 서식 파일입니다. 이미지는 변경 되지 않은 기본 운영 체제 환경을 기반으로 할 수 있지만 수정 된 컨테이너의 샌드박스에서 만들 수도 있습니다. 이러한 수정 된 이미지는 기본 이미지 레이어의 위쪽에 변경 내용을 계층화 하 고, 이러한 레이어를 다른 기본 이미지에 복사 및 재적용 하 여 동일한 변경 내용이 있는 새 이미지를 만들 수 있습니다.
- 컨테이너 리포지토리: 새 이미지를 만들 때마다 컨테이너 이미지와 해당 종속성을 저장 하는 로컬 리포지토리입니다. 저장 된 이미지를 컨테이너 호스트에서 원하는 횟수 만큼 다시 사용할 수 있습니다. 또한 컨테이너 이미지를 Docker 허브와 같은 공용 또는 개인 레지스트리에 저장 하 여 다양 한 컨테이너 호스트에서 사용할 수 있습니다.
- 컨테이너 orchestrator: 많은 수의 컨테이너와 상호 작용 하는 방식을 자동화 하 고 관리 하는 프로세스입니다. 자세히 알아보려면 [Windows 컨테이너 Orchestrators 정보](overview-container-orchestrators.md)를 참조 하세요.
- Docker: 컨테이너 이미지를 패키지화 하 고 전달 하는 자동화 된 프로세스입니다. 자세한 내용은 [Windows의](../manage-docker/configure-docker-daemon.md) docker [개요](docker-overview.md), docker 엔진을 참조 하거나 [docker 웹 사이트](https://www.docker.com)를 방문 하 여 자세히 알아보세요.

![컨테이너를 만드는 방법을 보여 주는 순서도 응용 프로그램 및 기본 이미지를 사용 하 여 새 컨테이너를 빌드할 수 있도록 기본 이미지의 맨 위에 있는 샌드박스 및 새 응용 프로그램 이미지를 만듭니다.](media/containerfund.png)

가상 컴퓨터에 익숙한 사용자는 컨테이너와 가상 컴퓨터가 비슷한 것으로 생각 될 것입니다. 컨테이너는 운영 체제를 실행 하 고, 파일 시스템을가지고 있으며, 실제 또는 가상 컴퓨터 시스템 처럼 네트워크를 통해 액세스할 수 있습니다. 그렇긴 하지만 컨테이너의 바탕이 되는 기술 및 개념은 가상 컴퓨터와는 상당한 차이가 있습니다. 이러한 개념에 대해 자세히 알아보려면 차이점을 자세히 설명 하는 [블로그 게시물](https://azure.microsoft.com/blog/containers-docker-windows-and-trends/) 표시 Russinovich를 참조 하세요.

### <a name="windows-container-types"></a>Windows 컨테이너 유형

다른 방법으로는 런타임 라고도 하는 두 가지 컨테이너 형식이 있다는 것을 알아야 합니다.

Windows Server 컨테이너는 프로세스 및 네임 스페이스 격리 기술을 통해 응용 프로그램 격리를 제공 하므로 이러한 컨테이너가 프로세스 격리 컨테이너 라고도 합니다. Windows Server 컨테이너는 컨테이너 호스트와 호스트에서 실행되는 모든 컨테이너와 커널을 공유합니다. 이러한 프로세스 격리 된 컨테이너는 악의적인 보안 경계를 제공 하지 않으며 신뢰 되지 않는 코드를 격리 하는 데 사용할 수 없습니다. 공유 커널 공간 때문에 이러한 컨테이너에 동일한 커널 버전과 구성이 필요합니다.

Hyper-v 격리는 고도로 최적화 된 가상 컴퓨터에서 각 컨테이너를 실행 하 여 Windows Server 컨테이너에서 제공 하는 격리를 확장 합니다. 이 구성에서 컨테이너 호스트는 해당 커널을 동일한 호스트의 다른 컨테이너와 공유 하지 않습니다. 이러한 컨테이너는 가상 컴퓨터와 보안 보장 수준이 같은 적대적 다중 테넌트 호스팅을 위해 디자인되었습니다. 이러한 컨테이너는 호스트의 호스트나 다른 컨테이너와 커널을 공유 하지 않으므로 서로 다른 버전 및 구성 (지원 되는 버전 내)을 사용 하 여 커널을 실행할 수 있습니다. 예를 들어 Windows 10의 모든 Windows 컨테이너는 Hyper-v 격리를 사용 하 여 Windows Server 커널 버전 및 구성을 활용 합니다.

Windows에서 Hyper-v 격리 여부에 관계 없이 컨테이너를 실행 하는 것은 런타임 결정입니다. 먼저 Hyper-v 격리를 사용 하 여 컨테이너를 만든 다음 런타임에서 나중에 Windows Server 컨테이너로 실행 되도록 선택할 수 있습니다.

## <a name="container-users"></a>컨테이너 사용자

### <a name="containers-for-developers"></a>개발자를 위한 컨테이너

컨테이너는 개발자가 고품질의 응용 프로그램을 더 빠르게 빌드하고 제공 하는 데 도움이 됩니다. 개발자는 모든 환경에서 동일 하 게 배포 되는 Docker 이미지를 몇 초 안에 만들 수 있습니다. Docker 컨테이너에는 대규모의 성장 하는 응용 프로그램의 환경도 있습니다. Docker에 의해 유지 관리 되는 공개 containerized 응용 프로그램 레지스트리 인 DockerHub는 공용 커뮤니티 리포지토리에서 18만 개 이상의 응용 프로그램을 게시 했지만 해당 숫자가 여전히 증가 합니다.

개발자가 앱을 containerizes 하는 경우 앱 및 실행 해야 하는 구성 요소만 이미지로 결합할 수 있습니다. 그런 다음 이 이미지를 통해 필요할 때 컨테이너가 만들어집니다. 이미지를 기반으로 다른 이미지를 만들 수도 있으므로 이미지 만들기 작업이 매우 신속해집니다. 여러 컨테이너는 같은 이미지를 공유할 수 있으며,이 경우 컨테이너는 매우 빠르게 시작 하 고 리소스를 적게 사용 하기도 합니다. 예를 들어 개발자는 컨테이너를 사용 하 여 분산 된 앱에 대 한 경량 및 이식 가능한 앱 구성 요소를 회전 하 고 각 서비스를 개별적으로 신속 하 게 확장할 수 있습니다.

컨테이너는 이식 가능 하 고 다양 한 언어로 작성할 수 있으며, Windows Server 2016를 실행 하는 컴퓨터와 호환 됩니다. 개발자는 자신의 랩톱이 나 데스크톱에서 컨테이너를 로컬로 만들고 테스트 한 다음 동일한 컨테이너 이미지를 해당 회사의 사설 클라우드, 공용 클라우드 또는 서비스 공급자에 배포할 수 있습니다. 컨테이너의 자연 민첩성은 가상화 된 대규모 클라우드 환경에서 최신 앱 개발 패턴을 지원 합니다.

### <a name="containers-for-it-professionals"></a>IT 전문가를 위한 컨테이너

컨테이너는 관리자가 업데이트 및 유지 관리 하기 쉬운 인프라를 만드는 데 도움이 됩니다. IT 전문가는 컨테이너를 사용 하 여 개발, QA, 프로덕션 팀에 게 표준화 된 환경을 제공할 수 있습니다. 더 이상 복잡 한 설치 및 구성 절차에 대해 걱정할 필요가 없습니다. 컨테이너를 사용 하 여 시스템 관리자는 OS 설치 및 기본 인프라의 차이를 추상화 합니다.

## <a name="containers-101-video-presentation"></a>컨테이너 101 비디오 프레젠테이션

다음 비디오 프레젠테이션에서는 Windows 컨테이너의 기록 및 구현에 대 한 세부적인 개요를 제공 합니다.

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## <a name="try-windows-server-containers"></a>Windows Server 컨테이너 체험

컨테이너의 놀라운 성능을 활용할 준비가 되었나요? 다음 문서는 시작 하는 데 도움이 됩니다.

Windows Server에서 컨테이너를 설정 하려면 [Windows server 빠른](../quick-start/quick-start-windows-server.md)시작을 참조 하세요.

Windows 10에서 컨테이너를 설정 하려면 [windows 10 빠른](../quick-start/quick-start-windows-10.md)시작을 참조 하세요.