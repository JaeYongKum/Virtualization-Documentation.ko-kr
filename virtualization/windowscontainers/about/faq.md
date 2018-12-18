---
title: Windows 컨테이너 FAQ
description: Windows 컨테이너 FAQ
keywords: Docker, 컨테이너
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 42e1a0bf58ada40a8f135861646d5c9429e0d5db
ms.sourcegitcommit: a5f8f99bf8f512a9058b72f1f617f77ecf488c71
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/18/2018
ms.locfileid: "8974015"
---
# <a name="frequently-asked-questions"></a>질문과 대답

## <a name="general"></a>일반

### <a name="what-is-wcow-what-is-lcow"></a>WCOW 란 무엇 인가요? LCOW 란 무엇 인가요?

WCOW는 LCOW 및 Windows에서 Windows 컨테이너에 대 한 약어는 Windows의 Linux 컨테이너에 대 한 약어입니다.

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Linux와 Windows Server 컨테이너 간의 차이는 무엇입니까?

Linux와 Windows Server 컨테이너는 유사하며 커널과 코어 운영 체제 안에서는 비슷한 기술을 구현합니다. 차이는 컨테이너 안에서 실행되는 워크로드와 플랫폼에 있습니다.  
고객이 Windows Server 컨테이너를 사용할 경우 .NET, ASP.NET, PowerShell 등의 기존 Windows 기술과 통합할 수 있습니다.

### <a name="as-a-developer-do-i-have-to-re-write-my-app-for-each-type-of-container"></a>개발자가 각 컨테이너 유형마다 앱을 다시 작성해야 하나요?

아니요. Windows 컨테이너 이미지는 Windows Server 컨테이너와 Hyper-V 컨테이너 간에 공통입니다. 컨테이너를 시작할 때 컨테이너 유형을 선택합니다. 개발자의 관점에서 Windows Server 컨테이너와 Hyper-V 컨테이너는 같은 물건의 두 버전이라고 할 수 있습니다. 개발, 프로그래밍, 관리 환경이 동일하고, 개방형 및 확장 가능하며, Docker를 통해 동일한 수준의 통합과 지원을 포함합니다. 

개발자는 적합한 런타임 플래그를 지정하는 것 말고는 별다른 변경 없이도 Windows Server 컨테이너를 사용하여 컨테이너 이미지를 만들고 Hyper-V 컨테이너로 배포하거나 혹은 반대의 경우로 할 수 있습니다.

Windows Server 컨테이너는 속도가 중요한 상황에서 더 높은 밀도와 성능을 제공합니다(중첩 구성 대비 스핀 시간 감소, 런타임 성능 빠름). Hyper-V 컨테이너는 더 높은 격리 수준을 제공하므로 한 컨테이너에서 실행되는 코드가 호스트 운영 체제나, 동일한 호스트에서 실행되는 타 컨테이너에 손상을 입히거나 영향을 끼지지 않습니다. 이 유형은 SaaS 운영 체제와 계산 호스팅 등, 다중 테넌트 시나리오에서 유용합니다(신뢰할 수 없는 코드 호스팅에 대한 요구 사항 있음).

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Windows에서 컨테이너를 실행 하기 위한 필수 사항은 무엇입니까?

컨테이너는 Windows Server 2016부터 플랫폼에 도입 되었습니다. 실행 해야 Windows Server 2016 또는 Windows 10 1 주년 업데이트 (버전 1607) 또는 컨테이너를 사용 하 여 최신 합니다.

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>실행할 수 있습니까 Windows 컨테이너 격리 프로세스 모드에서 Windows 10 Enterprise 또는 Professional?

프로세스 격리를 사용 하 여 Windows 컨테이너를 실행에서 사용자를 허용 하지 않습니다 업데이트에서는 더 이상 2018 년 10 월 10 일 Windows로 시작 합니다. 직접 사용 하 여 격리 프로세스에 대 한 요청 해야 합니다 `--isolation=process` 플래그를 통해 컨테이너를 실행할 때 `docker run`.

이에 관심이 있는 것은, 또는 최신 호스트에서 Windows 10 빌드 17763 + 실행 되 고 18.09 엔진을 사용 하 여 docker 버전 있는지 확인 해야 합니다.

> [!WARNING]
> 이 기능은 개발/테스트를 위해 위함입니다. 호스트와 Windows Server를 사용 하 여 프로덕션 배포를 계속 해야 합니다.
>
> 이 기능을 사용 하 여 확인 해야 호스트와 컨테이너 버전 태그 일치, 그렇지 않으면 컨테이너 오류가 발생할 수 또는 정의 되지 않은 동작이 발생할 수 있습니다.

## <a name="windows-container-management"></a>Windows 컨테이너 관리

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>만드는 방법 내 컨테이너 이미지를 사용할 수 있는 장치로 컴퓨터에서?

Windows 컨테이너 기본 이미지 분포가 라이선스에 따라 제한 된 아티팩트를 포함 합니다. 이러한 이미지를 빌드하고 공용 또는 개인 레지스트리를 푸시 때 기본 계층 되지 푸시됩니다 알 수 있습니다. 대신, Azure 클라우드 저장소에 있는 기본 실제 레이어를 가리키는 외부 계층 개념을 사용 합니다.

이 하면 장치로 컴퓨터의 _ _개인 컨테이너 레지스트리_ 주소에서 끌어오기 이미지만_ 수 있는 경우는 문제를 제공할 수 있습니다. 기본 이미지를 가져오기 위해 외부 계층에 따라 하려고 하면이 경우 실패 합니다. 외부 계층 동작을 재정의 하려면 사용 합니다 `--allow-nondistributable-artifacts` Docker 디먼에서 플래그.

> [!IMPORTANT]
> 이 플래그의 사용은 Windows 컨테이너 기본 이미지 라이선스; 약관을 준수 하 고 의무 배제 하지 공용 또는 제 3 자 재배포에 대 한 Windows 콘텐츠를 게시 하지 해야 합니다. 사용자가 자신의 환경 내에서 사용이 허용 됩니다.

## <a name="microsofts-open-ecosystem"></a>Microsoft의 오픈 에코 시스템

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft OCI( Open Container Initiative)에 해당하나요?

패키징 형식을 범용으로 유지하기 위해 최근 Docker는 기구에서 주도하는 형식에 따른 개방성 유지를 목표로 하는 OCI( Open Container Initiative)를 구성했습니다. Microsoft는 창립 구성원 중 하나입니다.

> ! [팁] FAQ에 대 한 추가 대 한 권장 사항이? 아래 새 피드백 발급할 새 또는 제안으로 이러한 문서에 대 한 프로젝트를 열고!
