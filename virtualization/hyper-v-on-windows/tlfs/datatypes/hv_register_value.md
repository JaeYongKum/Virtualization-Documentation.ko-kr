---
title: HV_REGISTER_VALUE
description: HV_REGISTER_VALUE
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 6f83aec87169c2a6902bd8b8b361cd1a93c70b3b
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121136"
---
# <a name="hv_register_value"></a>HV_REGISTER_VALUE

가상 프로세서 레지스터 값의 크기는 모두 128 비트입니다. 전체 128 비트를 사용 하지 않는 값은 0으로 확장 되어 전체 128 비트를 채웁니다.

## <a name="syntax"></a>구문

```c
typedef union
{
    UINT128 Reg128;
    UINT64 Reg64;
    UINT32 Reg32;
    UINT16 Reg16;
    UINT8 Reg8;
    HV_X64_FP_REGISTER Fp;
    HV_X64_FP_CONTROL_STATUS_REGISTER FpControlStatus;
    HV_X64_XMM_CONTROL_STATUS_REGISTER XmmControlStatus;
    HV_X64_SEGMENT_REGISTER Segment;
    HV_X64_TABLE_REGISTER Table;
    HV_EXPLICIT_SUSPEND_REGISTER ExplicitSuspend;
    HV_INTERCEPT_SUSPEND_REGISTER InterceptSuspend;
    HV_X64_INTERRUPT_STATE_REGISTER InterruptState;
    HV_X64_PENDING_INTERRUPTION_REGISTER PendingInterruption;
    HV_X64_MSR_NPIEP_CONFIG_CONTENTS NpiepConfig;
    HV_X64_PENDING_EXCEPTION_EVENT PendingExceptionEvent;
} HV_REGISTER_VALUE, *PHV_REGISTER_VALUE;
 ```