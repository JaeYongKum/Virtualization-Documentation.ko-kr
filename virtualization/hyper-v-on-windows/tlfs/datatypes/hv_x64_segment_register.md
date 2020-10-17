---
title: HV_X64_SEGMENT_REGISTER
description: HV_X64_SEGMENT_REGISTER
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 6c661e3ce8314d6aeaf94286b0bba190a78f629a
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121016"
---
# <a name="hv_x64_segment_register"></a>HV_X64_SEGMENT_REGISTER

세그먼트 레지스터 상태는 다음과 같이 인코딩됩니다.

## <a name="syntax"></a>구문

```c
typedef struct
{
    UINT64 Base;
    UINT32 Limit;
    UINT16 Selector;
    union
    {
        struct
        {
            UINT16 SegmentType:4;
            UINT16 NonSystemSegment:1;
            UINT16 DescriptorPrivilegeLevel:2;
            UINT16 Present:1;
            UINT16 Reserved:4;
            UINT16 Available:1;
            UINT16 Long:1;
            UINT16 Default:1;
            UINT16 Granularity:1;
        };

        UINT16 Attributes;
    };
} HV_X64_SEGMENT_REGISTER;
 ```

제한은 32 비트 값으로 인코딩됩니다. X64 long 모드 세그먼트의 경우 제한이 무시 됩니다. 레거시 x86 세그먼트의 경우 x64 프로세서 아키텍처의 범위 내에서 제한을 표현할 수 있어야 합니다. 예를 들어 코드 또는 데이터 세그먼트의 특성 내에서 "G" (세분성) 비트가 설정 된 경우 제한의 하위 12 비트는 1 이어야 합니다.

"Present" 비트는 세그먼트가 null 세그먼트 처럼 동작 하는지 여부를 제어 합니다. 즉, 해당 세그먼트를 통해 수행 된 메모리 액세스에서 #GP 오류가 생성 되는지 여부를 제어 합니다.

MSRs IA32_FS_BASE 및 IA32_GS_BASE는 세그먼트 레지스터 구조의 기본 요소에 대 한 별칭 이므로 등록 목록에 정의 되어 있지 않습니다. HvX64RegisterFs 및 HvX64RegisterGs와 위의 구조를 대신 사용 합니다.