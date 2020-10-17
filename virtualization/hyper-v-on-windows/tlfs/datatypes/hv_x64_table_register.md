---
title: HV_X64_TABLE_REGISTER
description: HV_X64_TABLE_REGISTER
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 1fddba94270b7c1591ac6f7aa92d3ce5e926fe1e
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121015"
---
# <a name="hv_x64_table_register"></a>HV_X64_TABLE_REGISTER

테이블 레지스터는 세그먼트 레지스터와 유사 하지만 선택기 나 특성이 없으며 제한이 16 비트로 제한 됩니다.

## <a name="syntax"></a>구문

```c
typedef struct
{
    UINT16 Pad[3];
    UINT16 Limit;
    UINT64 Base;
} HV_X64_TABLE_REGISTER;
 ```
