---
title: Windows 컨테이너 FAQ
description: Windows Server 컨테이너 FAQ
keywords: Docker, 컨테이너
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 69783f0fc3dcc80eb9614031dc6c9b2c35eeefd1
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9577144"
---
# <a name="frequently-asked-questions"></a>자주 묻는 질문

## <a name="general"></a>일반

### <a name="what-is-wcow-what-is-lcow"></a>WCOW 란 무엇 인가요? LCOW 란 무엇 인가요?

WCOW 창과 LCOW의 Windows 컨테이너에 대 한 약어는 Windows에서 Linux 컨테이너에 대 한 약어입니다.

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Linux와 Windows Server 컨테이너 간의 차이는 무엇입니까?

Linux 및 Windows Server 컨테이너는 모두 하며 커널과 코어 운영 체제 안에서 비슷한 기술을 구현 한다는 점에서 유사 합니다. 차이는 컨테이너 안에서 실행되는 워크로드와 플랫폼에 있습니다.  

고객이 Windows Server 컨테이너를 사용 하는 경우.NET, ASP.NET, PowerShell, 등과 같은 기술 기존 windows 통합할 수 있습니다.

### <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>개발자가 각 유형의 컨테이너에 대 한 내 앱을 다시 작성을 하나요?

아니요. Windows 컨테이너 이미지는 Windows Server 컨테이너와 Hyper-v 격리 간에 공통입니다. 컨테이너를 시작할 때 컨테이너 유형을 선택합니다. 개발자의 관점에서 Windows Server 컨테이너와 Hyper-v 격리에는 동일 합니다. 동일한 개발, 프로그래밍, 관리 환경이, 개방형 및 확장 가능 하며 동일한 수준의 통합과 docker 지원을 포함 합니다.

개발자는 Windows Server 컨테이너를 사용 하 여 컨테이너 이미지를 만들고 Hyper-v 격리 또는 그 반대로 적절 한 런타임 플래그를 지정 하는 별다른 변경 없이 배포할 수 있습니다.

Windows Server 컨테이너 속도가 키, 시간 및 런타임 성능을 중첩 된 구성에 비해 낮은 회전 등의 경우 큰 밀도 및 성능을 제공 합니다. Hyper-v 격리 된 이름 true 이면 한 컨테이너에서 실행 되는 코드가 손상 없거나 호스트 운영 체제 또는 동일한 호스트에서 실행 되는 다른 컨테이너에 영향을 보장 하는 더 높은 격리 수준을 제공 합니다. 이 saas와 계산 호스팅 등, 신뢰할 수 없는 코드 호스팅에 대 한 요구 사항이 다중 테 넌 트 시나리오에 유용 합니다.

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Windows에서 컨테이너를 실행 하기 위한 필수 사항은 무엇입니까?

컨테이너는 Windows Server 2016을 사용 하 여 플랫폼에 도입 되었습니다. 컨테이너를 사용 하려면 해야 Windows Server 2016 또는 Windows 10 1 주년 업데이트 (버전 1607) 이상.

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>실행할 수 있습니까 Windows 컨테이너 격리 프로세스 모드에서 Windows 10 Enterprise 또는 Professional?

업데이트에서는 더 이상을 2018 년 10 월 10 일 windows 시작 프로세스 격리를 사용 하 여 Windows 컨테이너를 실행 한 사용자를 허용 하지 않습니다. 그러나 직접 요청 해야 프로세스 격리를 사용 하 여는 `--isolation=process` 플래그를 통해 컨테이너를 실행할 때 `docker run`.

에 관심이 있는 것 인 경우 호스트는 Windows 10 빌드 17763 + 실행 되 고 18.09 엔진을 사용 하 여 docker 버전 또는 최신 확인 해야 합니다.

> [!WARNING]
> 이 기능은 개발/테스트를 위해 위함입니다. 프로덕션 배포에 대 한 호스트와 Windows Server를 사용 하 여 계속 해야 합니다.
>
> 이 기능을 사용 하 여 확인 해야 호스트와 컨테이너 버전 태그 일치, 그렇지 않으면 컨테이너 오류가 발생할 수 또는 정의 되지 않은 동작이 발생할 수 있습니다.

## <a name="windows-container-management"></a>Windows 컨테이너 관리

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>만드는 방법 내 컨테이너 이미지를 사용할 수 있는 장치로 컴퓨터에서?

Windows 컨테이너 기본 이미지 분포가 라이선스에 따라 제한 된 아티팩트를 포함 합니다. 이러한 이미지를 토대로 하는 공용 또는 개인 레지스트리를 밀어 계층 되지 푸시됩니다 알 수 있습니다. 대신, Azure 클라우드 저장소에 있는 실제 기본 레이어를 가리키는 외부 계층 개념을 사용 합니다.

이 하면는 장치로 컴퓨터의 개인 컨테이너 레지스트리 주소에서 이미지를 시도해만 볼 수 있는 경우는 문제를 제공할 수 있습니다. 기본 이미지를 가져오기 위해 외부 계층에 따라 하려고 하면이 경우 실패 합니다. 외부 계층 동작을 재정의 하려면 사용할 수는 `--allow-nondistributable-artifacts` Docker 디먼 플래그.

> [!IMPORTANT]
> 이 플래그의 사용은 Windows 컨테이너 기본 이미지 라이선스; 약관을 준수 하 고 의무 배제 하지 공용 또는 제 3 자 재배포에 대 한 Windows 콘텐츠를 게시 하지 해야 합니다. 사용자가 자신의 환경 내에서 사용이 허용 됩니다.

## <a name="microsofts-open-ecosystem"></a>Microsoft의 오픈 에코 시스템

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft OCI( Open Container Initiative)에 해당하나요?

패키징 형식을 범용으로 유지하기 위해 최근 Docker는 기구에서 주도하는 형식에 따른 개방성 유지를 목표로 하는 OCI( Open Container Initiative)를 구성했습니다. Microsoft는 창립 구성원 중 하나입니다.

> [!TIP]
> FAQ에 대 한 추가 대 한 권장 사항이? 설명 섹션에서 새 피드백 문제를 열거나 GitHub를 사용 하 여 이러한 문서에 대 한 끌어오기 요청을 열!
