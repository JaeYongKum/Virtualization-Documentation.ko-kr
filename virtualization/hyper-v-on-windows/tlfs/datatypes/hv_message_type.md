---
title: HV_MESSAGE_TYPE
description: HV_MESSAGE_TYPE
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 0d55012789000828292fb9e88995cc70d1bb13d2
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121152"
---
# <a name="hv_message_type"></a>HV_MESSAGE_TYPE

Syic 메시지는 메시지 유형을 32 비트 숫자로 인코딩합니다. 상위 비트가 설정 된 메시지 유형은 하이퍼바이저에서 사용 하도록 예약 되어 있습니다. 게스트에서 시작한 메시지는 하이퍼바이저 메시지 유형을 사용 하 여 메시지를 보낼 수 없습니다.

## <a name="syntax"></a>구문

 ```c
#define HV_MESSAGE_TYPE_HYPERVISOR_MASK 0x80000000

typedef enum
{
    HvMessageTypeNone = 0x00000000,

    // Memory access messages
    HvMessageTypeUnmappedGpa = 0x80000000,
    HvMessageTypeGpaIntercept = 0x80000001,

    // Timer notifications
    HvMessageTimerExpired = 0x80000010,

    // Error messages
    HvMessageTypeInvalidVpRegisterValue = 0x80000020,
    HvMessageTypeUnrecoverableException = 0x80000021,
    HvMessageTypeUnsupportedFeature = 0x80000022,
    HvMessageTypeTlbPageSizeMismatch = 0x80000023,

    // Hypercall intercept
    HvMessageTypeHypercallIntercept = 0x80000050,

    // Platform-specific processor intercept messages
    HvMessageTypeX64IoPortIntercept = 0x80010000,
    HvMessageTypeMsrIntercept = 0x80010001,
    HvMessageTypeX64CpuidIntercept = 0x80010002,
    HvMessageTypeExceptionIntercept = 0x80010003,
    HvMessageTypeX64ApicEoi = 0x80010004,
    HvMessageTypeX64LegacyFpError = 0x80010005,
    HvMessageTypeRegisterIntercept = 0x80010006,
} HV_MESSAGE_TYPE;
 ```
