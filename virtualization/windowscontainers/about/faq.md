---
title: Windows 컨테이너 FAQ
description: Windows Server 컨테이너 FAQ
keywords: Docker, 컨테이너
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 64573b539438de6ec5564b2949642ef12e55fc62
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/01/2019
ms.locfileid: "10274070"
---
# <a name="frequently-asked-questions-about-containers"></a>컨테이너에 대 한 자주 묻는 질문

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux와 Windows Server 컨테이너의 차이점은 무엇 인가요?

Linux 및 Windows Server는 모두 커널 및 핵심 운영 체제 내에서 유사한 기술을 구현 합니다. 차이는 컨테이너 안에서 실행되는 워크로드와 플랫폼에 있습니다.  

고객이 Windows Server 컨테이너를 사용 하는 경우 .NET, ASP.NET, PowerShell 등의 기존 Windows 기술과 통합할 수 있습니다.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Windows에서 컨테이너를 실행 하기 위한 필수 구성 요소는 무엇 인가요?

컨테이너는 Windows Server 2016 플랫폼에 도입 되었습니다. 컨테이너를 사용 하려면 Windows Server 2016 또는 Windows 10 기념일 업데이트 (버전 1607) 이상이 필요 합니다. 자세한 내용은 [시스템 요구 사항을](../deploy-containers/system-requirements.md) 참조 하세요.

## <a name="what-are-wcow-and-lcow"></a>WCOW 및 LCOW는 무엇 인가요?

"Windows의 Windows 컨테이너"에 WCOW가 짧습니다. LCOW "Windows의 Linux 컨테이너"에 대해 간략하게 작성 되었습니다.

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>컨테이너는 어떻게 사용이 허가 되나요? 실행할 수 있는 컨테이너 수에 제한이 있나요?

Windows 컨테이너 이미지 [EULA](../images-eula.md) 는 유효한 라이선스가 있는 호스트 OS를 보유 한 사용자에 따라 달라 지는 사용 방법을 설명 합니다. 사용자가 실행할 수 있는 컨테이너의 수는 호스트 OS 버전과 컨테이너를 실행 하는 [격리 모드](../manage-containers/hyperv-container.md) , 그리고 이러한 컨테이너가 개발자/테스트 목적으로 실행 되는지, 아니면 프로덕션에 사용 되 고 있는지에 따라 달라 집니다.

|호스트 OS                                                         |프로세스 격리 컨테이너 제한                   |Hyper-v-격리 된 컨테이너 제한                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |제한 없음                                          |2                                                  |
|Windows Server Datacenter                                       |제한 없음                                          |제한 없음                                          |
|Windows 10 Pro 및 Enterprise                                   |무제한 *(테스트 또는 개발 목적 으로만 사용 가능)*|무제한 *(테스트 또는 개발 목적 으로만 사용 가능)*|
|Windows 10 IoT Core 및 Enterprise                             |아무런                                         |아무런                                          |

Windows Server 컨테이너 이미지 사용은 해당 [버전](/windows-server/get-started-19/editions-comparison-19.md)에 대해 지원 되는 가상화 게스트의 수를 읽어 결정 됩니다. <br/>

>[!NOTE]
>\ * Windows 10 Core 런타임 이미지 또는 Windows 10 IoT Enterprise 디바이스 라이선스 ("Windows IoT 상업용 계약")에 대 한 Microsoft 상업용 사용 약관에 동의한 경우에 따라 설치 된 IoT 버전의 컨테이너에 대 한 제품 사용이 달라 집니다. Windows IoT 상업용 규약의 추가 약관은 프로덕션 환경에서 컨테이너 이미지 사용에 적용 됩니다. [컨테이너 이미지 사용권 계약서](../images-eula.md) 를 읽고 허용 되는 항목과 그렇지 않은 항목을 정확 하 게 파악 하십시오.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>개발자는 각 유형의 컨테이너에 대해 앱을 다시 작성 해야 하나요?

아니요. Windows 컨테이너 이미지는 Windows Server 컨테이너와 Hyper-v 격리 모두에서 공통적입니다. 컨테이너를 시작할 때 컨테이너 유형을 선택합니다. 개발자 관점에서 Windows Server 컨테이너와 Hyper-v 격리는 동일한 작업의 두 가지 특성입니다. 이는 동일한 개발, 프로그래밍, 관리 환경을 제공 하며, 개방 및 확장 가능 하며 Docker와 동일한 수준의 통합 및 지원 기능을 포함 합니다.

개발자는 Windows Server 컨테이너를 사용 하 여 컨테이너 이미지를 만들 수 있으며, 적절 한 런타임 플래그를 지정 하는 것 외에는 변경 없이 Hyper-v 격리에 배포 하거나 그 반대의 경우도 마찬가지입니다.

Windows Server 컨테이너는 낮은 회전 시간과 더 빠른 런타임 성능 (예: 중첩 된 구성과 비교) 등의 속도를 높이기 위해 더 나은 밀도와 성능을 제공 합니다. Hyper-v 격리는 해당 이름에 적용 되며, 한 컨테이너에서 실행 되는 코드가 호스트 운영 체제 또는 동일한 호스트에서 실행 되는 다른 컨테이너에 영향을 줄 수 없도록 하는 격리를 제공 합니다. 이는 SaaS 응용 프로그램 및 계산 호스팅을 포함 하 여 신뢰할 수 없는 코드 호스팅을 위한 요구 사항이 있는 다중 테 넌 트 시나리오에 유용 합니다.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Windows 10의 프로세스 격리 모드에서 Windows 컨테이너를 실행할 수 있나요?

Windows 10 10 월 2018 업데이트부터 프로세스 격리를 사용 하 여 Windows 컨테이너를 실행할 수 있지만, 먼저 컨테이너를 실행할 때 플래그를 `--isolation=process` 사용 하 여 프로세스 격리를 직접 요청 해야 `docker run`합니다. 프로세스-격리는 Windows 10 Pro, Windows 10 Enterprise, Windows 10 IoT Core 및 Windows 10 IoT Enterprise에서 호환 됩니다.

이 방법으로 Windows 컨테이너를 실행 하려면 호스트가 Windows 10 build 17763 +를 실행 하 고 있는지, 그리고 엔진 18.09 이상이 포함 된 Docker 버전을 사용 하 고 있는 지 확인 해야 합니다.

> [!WARNING]
> IoT Core 및 IoT Enterprise 호스트 외에 (추가 약관을 적용 한 후)이 기능은 개발 및 테스트에만 사용 됩니다. Windows Server를 프로덕션 배포의 호스트로 계속 사용 해야 합니다. 이 기능을 사용 하 여 호스트 및 컨테이너 버전 태그가 일치 하도록 해야 하며 그렇지 않으면 컨테이너가 시작 되지 않거나 정의 되지 않은 동작이 발생할 수 있습니다.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Gapped 컴퓨터에서 컨테이너 이미지를 사용할 수 있게 하는 방법은 무엇 인가요?

Windows 컨테이너 기본 이미지에는 해당 분포가 라이선스에 의해 제한 된 아티팩트가 포함 됩니다. 이러한 이미지를 빌드하여 개인 또는 공용 레지스트리에 푸시하여 기본 계층이 푸시 되지 않는다는 것을 알 수 있습니다. 대신 Azure 클라우드 저장소에 상주 하는 실제 기본 계층을 가리키는 외래 계층의 개념을 사용 합니다.

이는 개인 컨테이너 레지스트리의 주소에서 이미지만 가져올 수 있는 air gapped 컴퓨터를 사용 하는 경우 복잡 해질 수 있습니다. 이 경우에는 기본 이미지를 가져오기 위해 외래 계층을 팔 로우 하려고 시도 하지 않습니다. 외래 레이어 동작을 재정의 하기 위해 Docker 데몬에 `--allow-nondistributable-artifacts` 플래그를 사용할 수 있습니다.

> [!IMPORTANT]
> 이 플래그를 사용 해도 사용자의 의무는 Windows 컨테이너 기본 이미지 라이선스의 조건을 준수 하지 않습니다. 공용 또는 타사 재배포를 위해 Windows 콘텐츠를 게시 하지 않아야 합니다. 자신의 환경 내에서 사용이 허용 됩니다.

## <a name="additional-feedback"></a>추가 피드백

FAQ에 항목을 추가 하 고 싶으신가요? 메모 섹션에서 새 피드백 문제를 열거나 GitHub를 사용 하 여이 페이지에 대 한 끌어오기 요청을 설정 합니다.
