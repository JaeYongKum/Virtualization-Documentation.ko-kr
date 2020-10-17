---
title: HV_CRASH_CTL_REG_CONTENTS
description: HV_CRASH_CTL_REG_CONTENTS
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: fe106c7998d5851c813e3a3a7c493c9c5aec550a
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120920"
---
# <a name="hv_crash_ctl_reg_contents"></a>HV_CRASH_CTL_REG_CONTENTS

다음 데이터 구조는 게스트 크래시 계몽 컨트롤 등록 (HV_X64_MSR_CRASH_CTL)의 콘텐츠를 정의 하는 데 사용 됩니다.

## <a name="syntax"></a>구문

 ```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Reserved : 62;         // Reserved bits
        UINT64 CrashMessage : 1;      // P3 is the PA of the message
                                      // P4 is the length in bytes
        UINT64 CrashNotify : 1;       // Log contents of crash parameter
    };
} HV_CRASH_CTL_REG_CONTENTS;
 ```

## <a name="see-also"></a>참고 항목

 [파티션 크래시 계몽](../partition-properties.md#partition-crash-enlightenment)