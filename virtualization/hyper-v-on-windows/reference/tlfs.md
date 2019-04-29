---
title: 하이퍼바이저 사양
description: 하이퍼바이저 사양
keywords: windows 10, hyper-v
author: allenma
ms.date: 06/26/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: 26eaca5a583f8b2e11ca1e05e2fa9a4366fd8837
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9577414"
---
# <a name="hypervisor-specifications"></a>하이퍼바이저 사양

## <a name="hypervisor-top-level-functional-specification"></a>하이퍼바이저 최상위 기능 사양

Hyper-V 하이퍼바이저 TLFS(최상위 기능 사양)에서는 다른 운영 체제 구성 요소에 외부적으로 표시되는 하이퍼바이저의 동작을 설명합니다. 이 사양은 게스트 운영 체제 개발자를 위한 것입니다.
  
> 이 사양은 Microsoft Open Specification Promise에 따라 제공됩니다.  [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications)에 대한 자세한 내용은 다음을 참조하세요.  

#### <a name="download"></a>다운로드
릴리스 | 문서
--- | ---
Windows Server 2016(수정 버전 C) | [Hypervisor Top Level Functional Specification v5.0c.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2(수정 버전 B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Microsoft 하이퍼바이저 인터페이스 구현 요구 사항

"HV#1" 인터페이스로 게스트 가상 컴퓨터에 선언되어 있는 TLFS는 Microsoft의 하이퍼바이저 아키텍처에 대한 모든 측면을 설명합니다.  그러나 Microsoft HV#1 하이퍼바이저 사양을 준수하려는 타사 하이퍼바이저가 TLFS에 설명된 모든 인터페이스를 구현해야 하는 것은 아닙니다. Microsoft HV#1 인터페이스와의 호환성을 갖추기 위해 하이퍼바이저에서 구현해야 하는 최소한의 하이퍼바이저 인터페이스 모음은 "Microsoft 하이퍼바이저 인터페이스 구현을 위한 요구 사항" 문서에 설명되어 있습니다.

#### <a name="download"></a>다운로드

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
