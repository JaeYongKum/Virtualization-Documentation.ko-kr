---
title: HV_INTERRUPT_ENTRY
description: HV_INTERRUPT_ENTRY
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 00ef384e332090dfacf9e4a201024e97a8bfb2ae
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121160"
---
# <a name="hv_interrupt_entry"></a>HV_INTERRUPT_ENTRY

## <a name="syntax"></a>구문

 ```c
typedef enum
{
    HvInterruptSourceMsi = 1,
} HV_INTERRUPT_SOURCE;

typedef struct
{
    HV_INTERRUPT_SOURCE InterruptSource;
    UINT32 Reserved;

    union
    {
        HV_MSI_ENTRY MsiEntry;
        UINT64 Data;
    };
} HV_INTERRUPT_ENTRY;
 ```

## <a name="see-also"></a>참고 항목

[HV_MSI_ENTRY](HV_MSI_ENTRY.md)