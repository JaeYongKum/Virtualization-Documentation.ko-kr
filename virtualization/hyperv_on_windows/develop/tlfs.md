---
title: "하이퍼바이저 사양"
description: "하이퍼바이저 사양"
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: 82b5055c390ce6754403b4de571b4c75298ff462

---

# 하이퍼바이저 사양

## 하이퍼바이저 최상위 기능 사양

Hyper-V 하이퍼바이저 TLFS(최상위 기능 사양)에서는 다른 운영 체제 구성 요소에 외부적으로 표시되는 하이퍼바이저의 동작을 설명합니다. 이 사양은 게스트 운영 체제 개발자를 위한 것입니다.
  
> 이 사양은 Microsoft Open Specification Promise에 따라 제공됩니다.  [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications)에 대한 자세한 내용은 다음을 참조하세요.  

#### 다운로드
릴리스 | 문서
--- | ---
Windows Server 2012 R2(수정 버전 B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 R2 | [Hypervisor Top Level Functional Specification v4.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## Microsoft 하이퍼바이저 인터페이스 구현 요구 사항

Windows 운영 체제를 게스트 가상 컴퓨터("HV#1" 인터페이스라고도 함)에서 실행하려면 제한된 하이퍼바이저 인터페이스 집합이 필요합니다. 또한 Microsoft 호환 하이퍼바이저로 여러 가지 선택적 기능을 구현할 수 있습니다. 이러한 옵션은 가상 컴퓨터에서 Windows의 동작을 변경합니다. "Microsoft 하이퍼바이저 인터페이스 구현 요구 사항"에서는 Microsoft 호환 하이퍼바이저로 구현되는 필수 기능과 선택적 기능을 모두 설명합니다.

#### 다운로드

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)


<!--HONumber=Jun16_HO4-->


