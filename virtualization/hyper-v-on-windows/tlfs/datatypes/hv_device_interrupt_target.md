---
title: HV_DEVICE_INTERRUPT_TARGET
description: HV_DEVICE_INTERRUPT_TARGET
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 44d31155b667a1e1f0114720150717318ab52451
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120910"
---
# <a name="hv_device_interrupt_target"></a>HV_DEVICE_INTERRUPT_TARGET

## <a name="syntax"></a>구문

 ```c

#define HV_DEVICE_INTERRUPT_TARGET_MULTICAST 1
#define HV_DEVICE_INTERRUPT_TARGET_PROCESSOR_SET 2

typedef struct
{
    HV_INTERRUPT_VECTOR Vector;
    UINT32 Flags;
    union
    {
        UINT64 ProcessorMask;
        UINT64 ProcessorSet[];
    };
} HV_DEVICE_INTERRUPT_TARGET;
 ```

"Flags"는 인터럽트 대상에 대 한 선택적 플래그를 제공 합니다. "멀티 캐스트"는 대상 집합의 모든 프로세서에 인터럽트가 전송 됨을 나타냅니다. 기본적으로 인터럽트는 대상 집합의 임의의 단일 프로세서에 전송 됩니다.
