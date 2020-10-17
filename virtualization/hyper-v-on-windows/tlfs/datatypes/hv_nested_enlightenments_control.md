---
title: HV_NESTED_ENLIGHTENMENTS_CONTROL
description: HV_NESTED_ENLIGHTENMENTS_CONTROL
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 71e4871aaa3cd50fac9e4b5e212ca029b180f157
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121143"
---
# <a name="hv_nested_enlightenments_control"></a>HV_NESTED_ENLIGHTENMENTS_CONTROL

하이퍼바이저가 현재 중첩 된 게스트 컨텍스트에 대해 중첩 된 계몽 권한을 부여할 부모 하이퍼바이저를 나타낼 수 있도록 하는 컨트롤 구조입니다.

## <a name="syntax"></a>구문

```c
typedef struct
{
    union {
        UINT32 AsUINT32;
        struct
        {
            UINT32 DirectHypercall:1;
            UINT32 VirtualizationException:1;
            UINT32 Reserved:30;
        };
    } Features;

    union
    {
        UINT32 AsUINT32;
        struct
        {
            UINT32 InterPartitionCommunication:1;
            UINT32 Reserved:31;
        };
     } HypercallControls;
} HV_NESTED_ENLIGHTENMENTS_CONTROL, *PHV_NESTED_ENLIGHTENMENTS_CONTROL;
 ```

## <a name="see-also"></a>참고 항목

[HV_VP_ASSIST_PAGE](HV_VP_ASSIST_PAGE.md)