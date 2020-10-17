---
title: HV_SYNIC_EVENT_FLAGS
description: HV_SYNIC_EVENT_FLAGS
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 741c01718890d453017fc22470b6b0e96298b05a
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121088"
---
# <a name="hv_synic_event_flags"></a>HV_SYNIC_EVENT_FLAGS

Syic 이벤트 플래그는 고정 크기의 비트 배열입니다. 배열의 첫 번째 바이트에는 0 ~ 7 플래그가 포함 되 고 (0은 최하위 비트), 배열의 두 번째 바이트에는 플래그 8-15 (가장 중요 하지 않은 비트)가 포함 됩니다.

## <a name="syntax"></a>구문

```c
#define HV_EVENT_FLAGS_COUNT (256 * 8)
#define HV_EVENT_FLAGS_BYTE_COUNT 256

typedef struct
{
    UINT8 Flags[HV_EVENT_FLAGS_BYTE_COUNT];
} HV_SYNIC_EVENT_FLAGS;
 ```
