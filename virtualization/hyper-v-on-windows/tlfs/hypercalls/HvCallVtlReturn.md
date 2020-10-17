---
title: HvCallVtlReturn
description: HvCallVtlReturn hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7070443a4e41c7059980196f47fc8d31d6f5ced3
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120990"
---
# <a name="hvcallvtlreturn"></a>HvCallVtlReturn

HvCallVtlReturn은 "[VTL return](../vsm.md#vtl-return)"을 시작 하 고 부사장에서 사용 하도록 설정 된 다음으로 낮은 VTL로 전환 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallVtlReturn(
    VOID
    );
 ```

## <a name="call-code"></a>호출 코드

`0x0012` 쉽게