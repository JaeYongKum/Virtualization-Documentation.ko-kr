---
title: HV_X64_FP_REGISTER
description: HV_X64_FP_REGISTER
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 10a6c03c9ba9d3d985b2980f8d893556f76f9cae
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121055"
---
# <a name="hv_x64_fp_register"></a>HV_X64_FP_REGISTER

부동 소수점 레지스터는 80 비트 값으로 인코딩됩니다.

## <a name="syntax"></a>구문

```c
typedef struct
{
    UINT64 Mantissa;
    UINT64 BiasedExponent:15;
    UINT64 Sign:1;
    UINT64 Reserved:48;
} HV_X64_FP_REGISTER;
 ```
