---
title: 하이퍼바이저 사양
description: 하이퍼바이저 사양
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 97615dc78f31a3a619130abb5a4d3786fdd412b2
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120897"
---
# <a name="hypervisor-top-level-functional-specification"></a>하이퍼바이저 최상위 기능 사양

Hyper-v 하이퍼바이저 (Top-Level 기능 사양)에서는 다른 운영 체제 구성 요소에 대 한 하이퍼바이저의 게스트 표시 동작을 설명 합니다. 이 사양은 게스트 운영 체제 개발자를 위한 것입니다.

> 이 사양은 Microsoft Open Specification Promise에 따라 제공됩니다.  [Microsoft Open Specification Promise](https://docs.microsoft.com/openspecs/dev_center/ms-devcentlp/51a0d3ff-9f77-464c-b83f-2de08ed28134)에 대한 자세한 내용은 다음을 참조하세요.

Microsoft는 이러한 자료의 주제를 다루는 특허, 특허 신청, 상표, 저작권 또는 기타 지적 재산권을 가질 수 있습니다. Microsoft Open Specification 약속에 명시적으로 제공 된 경우를 제외 하 고 이러한 자료의 제공은 이러한 특허권, 상표, 저작권 또는 기타 지적 재산에 대 한 라이선스를 제공 하지 않습니다.

## <a name="glossary"></a>용어

- **파티션** – hyper-v는 파티션 측면에서 격리를 지원 합니다. 파티션은 운영 체제가 실행 되는 하이퍼바이저에서 지 원하는 격리의 논리적 단위입니다.
- **루트 파티션** – 루트 파티션 ("부모" 또는 "호스트")은 권한 있는 관리 파티션입니다. 루트 파티션은 장치 드라이버, 전원 관리 및 장치 추가/제거와 같은 컴퓨터 수준 기능을 관리 합니다. 가상화 스택은 부모 파티션에서 실행 되 고 하드웨어 장치에 직접 액세스할 수 있습니다. 그런 다음 루트 파티션은 게스트 운영 체제를 호스트 하는 자식 파티션을 만듭니다.
- **자식 파티션** – 자식 파티션 ( "게스트"는 게스트 운영 체제를 호스팅합니다. 자식 파티션에서 실제 메모리 및 장치에 대 한 모든 액세스는 VMBus (가상 컴퓨터 버스) 또는 하이퍼바이저를 통해 제공 됩니다.
- **Hypercall** – hypercall는 하이퍼바이저와 통신 하기 위한 인터페이스입니다.

## <a name="specification-style"></a>사양 스타일

이 문서에서는 높은 수준의 하이퍼바이저 아키텍처에 대해 잘 알고 있다고 가정 합니다.

이 사양은 비공식적입니다. 즉, 인터페이스가 공식 언어로 지정 되지 않습니다. 그럼에도 불구 하 고 정확 하 게 하는 것이 목표입니다. 아키텍처 및 구현 별 동작을 지정 하는 것도 목표입니다. 호출자는 이후 구현에서 변경 될 수 있으므로 후자 범주에 속하는 동작을 사용 하면 안 됩니다.

## <a name="previous-versions"></a>이전 버전

해제 | 문서
--- | ---
Windows Server 2019 (수정 버전 B) | [하이퍼바이저 최상위 기능 사양 v6.0b.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v6.0b.pdf)
Windows Server 2016 (수정 버전 C) | [하이퍼바이저 최상위 기능 사양 v5.0c.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2(수정 버전 B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Microsoft 하이퍼바이저 인터페이스 구현 요구 사항

TLFS는 게스트 가상 컴퓨터에 "HV # 1" 인터페이스로 선언 된 Microsoft 전용 하이퍼바이저 아키텍처의 모든 측면을 완벽 하 게 설명 합니다.  그러나 Microsoft HV # 1 하이퍼바이저 사양을 준수 하도록 선언 하려는 타사 하이퍼바이저에서 TLFS에 설명 된 모든 인터페이스를 구현 해야 하는 것은 아닙니다. "Microsoft 하이퍼바이저 인터페이스 구현 요구 사항" 문서에서는 Microsoft HV # 1 인터페이스와의 호환성을 클레임 하는 하이퍼바이저에서 구현 해야 하는 최소 하이퍼바이저 인터페이스 집합을 설명 합니다.

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)