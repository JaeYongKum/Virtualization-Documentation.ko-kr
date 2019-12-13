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
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910803"
---
# <a name="frequently-asked-questions-about-containers"></a>컨테이너에 대 한 질문과 대답

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux와 Windows Server 컨테이너 간의 차이점은 무엇 인가요?

Linux와 Windows Server는 모두 커널 및 핵심 운영 체제 내에서 유사한 기술을 구현 합니다. 차이는 컨테이너 안에서 실행되는 워크로드와 플랫폼에 있습니다.  

고객이 Windows Server 컨테이너를 사용 하는 경우 .NET, ASP.NET, PowerShell 등의 기존 Windows 기술과 통합할 수 있습니다.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Windows에서 컨테이너를 실행 하기 위한 필수 구성 요소는 무엇 인가요?

컨테이너는 Windows Server 2016를 사용 하는 플랫폼에 도입 되었습니다. 컨테이너를 사용 하려면 Windows Server 2016 또는 Windows 10 기념일 업데이트 (버전 1607) 이상이 필요 합니다. 자세한 내용은 [시스템 요구 사항](../deploy-containers/system-requirements.md) 을 참조 하세요.

## <a name="what-are-wcow-and-lcow"></a>WCOW 및 LCOW 이란?

WCOW은 "Windows의 Windows 컨테이너"에 대해 짧습니다. LCOW은 "Windows의 Linux 컨테이너"의 약어입니다.

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>컨테이너는 어떻게 사용이 허가 되나요? 실행할 수 있는 컨테이너 수에 제한이 있나요?

Windows 컨테이너 이미지 [EULA](../images-eula.md) 는 유효 하 게 사용이 허가 된 호스트 OS가 있는 사용자에 따라 달라 지는 사용을 설명 합니다. 사용자가 실행할 수 있는 컨테이너 수는 컨테이너를 실행 하는 호스트 OS 버전과 [격리 모드](../manage-containers/hyperv-container.md) 를 비롯 하 여 이러한 컨테이너가 개발/테스트 목적으로 실행 되는지 또는 프로덕션 환경에서 실행 되는지 여부에 따라 달라 집니다.

|호스트 OS                                                         |프로세스 격리 컨테이너 제한                   |Hyper-v 격리 컨테이너 제한                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |제한 없음                                          |2                                                  |
|Windows Server Datacenter                                       |제한 없음                                          |제한 없음                                          |
|Windows 10 Pro 및 Enterprise                                   |무제한 *(테스트 또는 개발 목적 으로만 사용)*|무제한 *(테스트 또는 개발 목적 으로만 사용)*|
|Windows 10 IoT Core 및 Enterprise                             |없음을                                         |없음을                                          |

Windows Server 컨테이너 이미지 사용은 해당 [버전](/windows-server/get-started-19/editions-comparison-19.md)에 대해 지원 되는 가상화 게스트 수를 읽어 결정 됩니다. <br/>

>[!NOTE]
>windows IoT 버전의 컨테이너에 대 한 프로덕션 사용 약관은 windows 10 Core 런타임 이미지 또는 Windows 10 IoT Enterprise 장치 라이선스 ("Windows IoT 상업적 계약")에 대 한 Microsoft 상용 사용 약관에 동의한 경우에 따라 달라 집니다. \* Windows IoT 상업적 계약의 추가 사용 약관은 프로덕션 환경에서 컨테이너 이미지 사용에 적용 됩니다. [컨테이너 이미지 EULA](../images-eula.md) 를 확인 하 여 허용 되는 내용과 그렇지 않은 항목을 정확 하 게 이해 하세요.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>개발자는 각 컨테이너 형식에 대해 앱을 다시 작성 해야 하나요?

아니요. Windows 컨테이너 이미지는 Windows Server 컨테이너와 Hyper-v 격리 모두에서 공통적으로 유지 됩니다. 컨테이너를 시작할 때 컨테이너 유형을 선택합니다. 개발자 관점에서 Windows Server 컨테이너와 Hyper-v 격리는 동일한 작업의 두 가지 특성입니다. 이러한 기능은 동일한 개발, 프로그래밍 및 관리 환경을 제공 하며, 공개 및 확장 가능 하며 Docker와 동일한 수준의 통합과 지원을 포함 합니다.

개발자는 적절 한 런타임 플래그를 지정 하는 것 외에는 변경 없이 Windows Server 컨테이너를 사용 하 여 컨테이너 이미지를 만들고 Hyper-v 격리에 배포 하거나 그 반대로 배포할 수 있습니다.

Windows Server 컨테이너는 중첩 된 구성에 비해 낮은 스핀 시간 및 빠른 런타임 성능 등 속도를 키로 하는 경우의 고밀도 및 성능을 제공 합니다. Hyper-v 격리를 통해 해당 이름에 대해 적용 하면 한 컨테이너에서 실행 되는 코드가 호스트 운영 체제나 동일한 호스트에서 실행 되는 다른 컨테이너에 영향을 주지 않도록 보장 하는 더 큰 격리를 제공 합니다. 이는 SaaS 응용 프로그램 및 계산 호스팅을 비롯 하 여 신뢰할 수 없는 코드 호스팅을 위한 요구 사항이 있는 다중 테 넌 트 시나리오에 유용 합니다.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Windows 10에서 Windows 컨테이너를 프로세스 격리 모드에서 실행할 수 있나요?

Windows 10 10 월 2018 업데이트부터 프로세스 격리를 통해 Windows 컨테이너를 실행할 수 있지만, `docker run`으로 컨테이너를 실행할 때 `--isolation=process` 플래그를 사용 하 여 먼저 프로세스 격리를 직접 요청 해야 합니다. 프로세스 격리는 Windows 10 Pro, Windows 10 Enterprise, Windows 10 IoT Core 및 Windows 10 IoT Enterprise에서 호환 됩니다.

이러한 방식으로 Windows 컨테이너를 실행 하려면 호스트에서 Windows 10 build 17763 +를 실행 하 고 있으며, 18.09 이상의 Docker 버전이 있는지 확인 해야 합니다.

> [!WARNING]
> IoT Core 및 IoT Enterprise 호스트 (추가 사용 약관을 수락한 후) 외에도이 기능은 개발 및 테스트에만 사용 됩니다. 프로덕션 배포를 위한 호스트로 Windows Server를 계속 사용 해야 합니다. 이 기능을 사용 하 여 호스트 및 컨테이너 버전 태그가 일치 하는지 확인 해야 합니다. 그렇지 않으면 컨테이너가 시작 되지 않거나 정의 되지 않은 동작이 발생할 수 있습니다.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>내 컨테이너 이미지를 gapped 컴퓨터에서 사용할 수 있도록 어떻게 할까요? 있나요?

Windows 컨테이너 기본 이미지에는 라이선스가 라이선스에 의해 제한 되는 아티팩트가 포함 됩니다. 이러한 이미지를 빌드하고 개인 또는 공용 레지스트리에 푸시할 때 기본 계층이 푸시되 지 않습니다. 대신, Azure cloud storage에 상주 하는 실제 기본 계층을 가리키는 외부 계층의 개념을 사용 합니다.

이는 개인 컨테이너 레지스트리의 주소에서 이미지를 끌어올 수 있는 gapped 컴퓨터가 있는 경우 복잡 해질 수 있습니다. 이 경우 기본 이미지를 가져오기 위해 외래 계층을 따르는 시도는 작동 하지 않습니다. 외래 계층 동작을 재정의 하기 위해 Docker 디먼에서 `--allow-nondistributable-artifacts` 플래그를 사용할 수 있습니다.

> [!IMPORTANT]
> 이 플래그를 사용 하면 Windows 컨테이너 기본 이미지 라이선스의 조건을 준수 하지 않습니다. 공용 또는 타사 재배포를 위해 Windows 콘텐츠를 게시 해서는 안 됩니다. 사용자 환경 내에서 사용이 허용 됩니다.

## <a name="additional-feedback"></a>추가 피드백

FAQ에 항목을 추가 하 시겠습니까? 설명 섹션에서 새 피드백 문제를 열거나 GitHub를 사용 하 여이 페이지에 대 한 끌어오기 요청을 설정 합니다.
