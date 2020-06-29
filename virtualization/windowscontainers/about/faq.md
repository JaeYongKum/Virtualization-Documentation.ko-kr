---
title: Windows 컨테이너 FAQ
description: Windows Server 컨테이너 FAQ
keywords: Docker, 컨테이너
author: PatrickLang
ms.date: 10/25/2019
ms.topic: overview
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: cd0b4e24cddb434d0a4051888afa3ae3a4a8e096
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192870"
---
# <a name="frequently-asked-questions-about-containers"></a>컨테이너에 대한 질문과 대답

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux와 Windows Server 컨테이너 간의 차이는 무엇입니까?

Linux와 Windows Server는 커널과 코어 운영 체제 내에서 비슷한 기술을 구현합니다. 차이는 컨테이너 안에서 실행되는 워크로드와 플랫폼에 있습니다.

고객이 Windows Server 컨테이너를 사용하는 경우 .NET, ASP.NET, PowerShell 등의 기존 Windows 기술과 통합할 수 있습니다.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Windows에서 컨테이너를 실행하기 위한 전제 조건은 무엇인가요?

컨테이너는 Windows Server 2016을 사용하는 플랫폼에 도입되었습니다. 컨테이너를 사용하려면 Windows Server 2016 또는 Windows 10 1주년 업데이트(버전 1607) 이상이 필요합니다. 자세한 내용은 [시스템 요구 사항](../deploy-containers/system-requirements.md)을 참조하세요.

## <a name="what-are-wcow-and-lcow"></a>WCOW 및 LCOW란 무엇입니까?

WCOW는 "Windows containers on Windows(Windows의 Windows 컨테이너)"의 약어입니다. LCOW는 "Linux containers on Windows(Windows의 Linux 컨테이너)"의 약어입니다.

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>컨테이너의 라이선스는 어떻게 얻나요? 실행할 수 있는 컨테이너 수에 제한이 있나요?

Windows 컨테이너 이미지 [EULA](../images-eula.md)를 보시면 유효하게 사용이 허가된 호스트 OS를 보유한 사용자에 따라 달라지는 사용법이 설명되어 있습니다. 사용자가 실행할 수 있는 컨테이너 수는 호스트 OS 버전과 컨테이너가 실행되는 [격리 모드](../manage-containers/hyperv-container.md), 그리고 이러한 컨테이너가 개발/테스트용으로 실행되는지 아니면 프로덕션 환경에서 실행되는지 여부에 따라 다릅니다.

|호스트 OS                                                         |프로세스 격리 컨테이너 제한                   |Hyper-V 격리 컨테이너 제한                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |제한 없음                                          |2                                                  |
|Windows Server Datacenter                                       |제한 없음                                          |제한 없음                                          |
|Windows 10 Pro 및 Enterprise                                   |무제한 *(테스트 또는 개발 용도에 한해)*|무제한 *(테스트 또는 개발 용도에 한해)*|
|Windows 10 IoT Core 및 Enterprise                             |무제한*                                         |무제한*                                          |

Windows Server 컨테이너 이미지 사용량은 해당 [버전](/windows-server/get-started-19/editions-comparison-19.md)에 지원되는 가상화 게스트 수를 읽어 결정됩니다. <br/>

>[!NOTE]
>\*Windows IoT 버전에서 사용 가능한 컨테이너의 프로덕션 사용량은 Windows 10 Core 런타임 이미지 또는 Windows 10 IoT Enterprise 디바이스 라이선스("Windows IoT 상용 계약")에 대한 Microsoft 상용 사용 약관에 동의했는지 여부에 따라 다릅니다. Windows IoT 상용 계약의 추가 사용 약관 및 제한은 프로덕션 환경에서 컨테이너 이미지를 사용하는 경우에만 적용됩니다. [컨테이너 이미지 EULA](../images-eula.md)를 읽고 허용되는 내용과 허용되지 않는 내용을 정확하게 이해하세요.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>개발자가 컨테이너 유형마다 앱을 다시 작성해야 하나요?

아니요. Windows 컨테이너 이미지는 Windows Server 컨테이너와 Hyper-V 격리 간에 공통입니다. 컨테이너를 시작할 때 컨테이너 유형을 선택합니다. 개발자의 관점에서 Windows Server 컨테이너와 Hyper-V 격리는 같은 물건의 두 버전이라고 할 수 있습니다. 개발, 프로그래밍, 관리 환경이 동일하고, 확장 가능한 개방형이며, Docker를 통해 동일한 수준의 통합과 지원을 포함합니다.

개발자는 적합한 런타임 플래그를 지정하는 것 말고는 별다른 변경 없이 Windows Server 컨테이너를 사용하여 컨테이너 이미지를 만들고 Hyper-V 격리에 배포할 수 있으며, 그 반대로 할 수도 있습니다.

Windows Server 컨테이너는 속도가 중요한 상황에서 우수한 밀도와 성능을 제공합니다(예: 중첩 구성 대비 스핀 시간 감소, 빠른 런타임 성능). Hyper-V 격리는 그 이름처럼 더 높은 격리 수준을 제공하므로 한 컨테이너에서 실행되는 코드가 호스트 운영 체제나, 동일한 호스트에서 실행되는 타 컨테이너에 손상을 입히거나 영향을 미치지 않습니다. SaaS 운영 체제, 컴퓨팅 호스팅 등의 다중 테넌트 시나리오에서 유용합니다(신뢰할 수 없는 코드 호스팅에 대한 요구 사항 있음).

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Windows 10에서 Windows 컨테이너를 프로세스 격리 모드로 실행할 수 있나요?

Windows 10 2018년 10월 업데이트부터 Windows 컨테이너를 프로세스 격리 모드로 실행할 수 있지만, `docker run`으로 컨테이너를 실행할 때 먼저 `--isolation=process` 플래그를 사용하여 프로세스 격리를 직접 요청해야 합니다. 프로세스 격리는 Windows 10 Pro, Windows 10 Enterprise, Windows 10 IoT Core 및 Windows 10 IoT Enterprise에서 호환됩니다.

이러한 방식으로 Windows 컨테이너를 실행하려면 호스트에서 Windows 10 빌드 17763 이상을 실행하고 있고 Docker 엔진 버전이 18.09 이상인지 확인해야 합니다.

> [!WARNING]
> 이 기능은 IoT Core 및 IoT Enterprise 호스트 외에는(추가 사용 약관 및 제한을 수락한 후) 개발 및 테스트 용도로만 사용해야 합니다. 프로덕션 배포를 위한 호스트로는 Windows Server를 계속 사용해야 합니다. 이 기능을 사용하면 호스트와 컨테이너 버전 태그가 일치하는지도 확인해야 합니다. 일치하지 않으면 컨테이너가 시작되지 않거나 정의되지 않은 동작이 발생할 수 있습니다.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>내 컨테이너 이미지를 인터넷에 연결되지 않은 머신에서 사용하려면 어떻게 할까요?

Windows 컨테이너 기본 이미지에는 배포하려면 라이선스가 필요한 아티팩트가 포함되어 있습니다. 이러한 이미지를 빌드하여 프라이빗 또는 퍼블릭 레지스트리에 푸시할 때 기본 레이어는 절대 푸시되지 않습니다. 그 대신, Azure 클라우드 스토리지에 상주하는 실제 기본 레이어를 가리키는 외래 레이어라는 개념을 사용합니다.

인터넷에 연결되지 않아 프라이빗 컨테이너 레지스트리의 주소에서만 이미지를 끌어올 수 있는 머신을 사용하는 경우 일이 복잡해질 수 있습니다. 이 경우 외래 레이어를 따라 기본 이미지를 가져오려는 시도는 통하지 않습니다. 외래 레이어 동작을 재정의하려면 Docker 디먼에서 `--allow-nondistributable-artifacts` 플래그를 사용하면 됩니다.

> [!IMPORTANT]
> 이 플래그를 사용해도 Windows 컨테이너 기본 이미지 라이선스의 사용 약관을 준수해야 하는 책임이 사라지지 않습니다. Windows 콘텐츠를 공용 또는 타사 재배포 용도로 게시하면 안 됩니다. 개발자의 자체 환경 내에서 사용하는 것은 허용됩니다.

## <a name="additional-feedback"></a>추가 피드백

FAQ에 추가하고 싶은 내용이 있나요? 댓글 섹션에서 새 피드백 이슈를 열거나 GitHub에서 이 페이지에 대한 끌어오기 요청을 설정하세요.
