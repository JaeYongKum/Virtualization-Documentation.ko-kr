---
title: HV_VP_VTL_CONTROL
description: HV_VP_VTL_CONTROL
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 2d307b85e073cc0d8e29d45ef8fdf3f5371de05c
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121064"
---
# <a name="hv_vp_vtl_control"></a>HV_VP_VTL_CONTROL

하이퍼바이저는 VP 지원 페이지의 일부를 사용 하 여 VTL0 보다 높은 VTL에서 실행 되는 코드와의 통신을 용이 하 게 합니다. 각 VTL에는 고유한 제어 구조 (VTL0 제외)가 있습니다.

다음 정보는 제어 페이지를 사용 하 여 전달 됩니다.

1. VTL 항목의 원인입니다.
2. VINA가 어설션 중임을 나타내는 플래그입니다.
3. VTL 반환 시 로드할 레지스터의 값입니다.

## <a name="syntax"></a>구문

```c
typedef enum
{
    // This reason is reserved and is not used.
    HvVtlEntryReserved = 0,

    // Indicates entry due to a VTL call from a lower VTL.
    HvVtlEntryVtlCall = 1,

    // Indicates entry due to an interrupt targeted to the VTL.
    HvVtlEntryInterrupt = 2
} HV_VTL_ENTRY_REASON;

typedef struct
{
    // The hypervisor updates the entry reason with an indication as to why
    // the VTL was entered on the virtual processor.
    HV_VTL_ENTRY_REASON EntryReason;

    // This flag determines whether the VINA interrupt line is asserted.
    union
    {
        UINT8 AsUINT8;
        struct
        {
            UINT8 VinaAsserted :1;
            UINT8 VinaReservedZ :7;
        };
    } VinaStatus;

    UINT8 ReservedZ00;
    UINT16 ReservedZ01;

    // A guest updates the VtlReturn* fields to provide the register values
    // to restore on VTL return. The specific register values that are
    // restored will vary based on whether the VTL is 32-bit or 64-bit.
    union
    {
        struct
        {
            UINT64 VtlReturnX64Rax;
            UINT64 VtlReturnX64Rcx;
        };

        struct
        {
            UINT32 VtlReturnX86Eax;
            UINT32 VtlReturnX86Ecx;
            UINT32 VtlReturnX86Edx;
            UINT32 ReservedZ1;
        };
    };
} HV_VP_VTL_CONTROL;
 ```

## <a name="see-also"></a>참고 항목

[HV_VP_ASSIST_PAGE](HV_VP_ASSIST_PAGE.md)
