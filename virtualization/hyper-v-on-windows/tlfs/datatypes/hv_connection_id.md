---
title: HV_CONNECTION_ID
description: HV_CONNECTION_ID
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: a3b1238c23c017dfb2fe756269cc2d59de1af5ff
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120927"
---
# <a name="hv_connection_id"></a>HV_CONNECTION_ID

연결은 32 비트 Id로 식별 됩니다. 상위 8 비트는 예약 되어 있으며 0 이어야 합니다. 모든 연결 Id는 파티션 내에서 고유 합니다.

## <a name="syntax"></a>구문

 ```c
typedef union
{
    UINT32 AsUInt32;
    struct
    {
        UINT32 Id:24;
        UINT32 Reserved:8;
    };
} HV_CONNECTION_ID;
 ```
