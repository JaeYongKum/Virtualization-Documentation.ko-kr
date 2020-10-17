---
title: HvCallVtlCall
description: HvCallVtlCall hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: c2481cc0da0b185c45a0285f2cd0394dff7af9c5
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120734"
---
# <a name="hvcallvtlcall"></a>HvCallVtlCall

HvCallVtlCall은 "[VTL call](../vsm.md#vtl-call)"을 시작 하 고 부사장에서 사용 하도록 설정 된 다음 최고 VTL로 전환 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallVtlCall(
    VOID
    );
 ```

## <a name="call-code"></a>호출 코드

`0x0011` 쉽게