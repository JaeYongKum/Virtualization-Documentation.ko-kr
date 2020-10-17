---
title: HvCallNotifyLongSpinWait
description: HvCallNotifyLongSpinWait hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 468a1c941d547d3a1243ac67e6d63812341fe589
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120758"
---
# <a name="hvcallnotifylongspinwait"></a>HvCallNotifyLongSpinWait

HvCallNotifyLongSpinWait hypercall는 게스트 OS에서 호출 하는 가상 프로세서가 동일한 파티션 내의 다른 가상 프로세서에서 잠재적으로 보유 하 고 있는 리소스를 얻으려고 시도 하 고 있음을 하이퍼바이저에 알리는 데 사용 됩니다. 이 예약 힌트는 가상 프로세서가 두 개 이상인 파티션의 확장성을 향상 시킵니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvNotifyLongSpinWait(
    _In_ UINT64 SpinCount
    );
 ```

## <a name="call-code"></a>호출 코드

`0x0008` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `SpinCount`             | 0          | 4        | 게스트가 회전 한 누적 횟수를 지정 합니다. |
| RsvdZ                   | 4          | 4        |                                           |

## <a name="return-values"></a>반환 값

이 HV_STATUS_SUCCESS hypercall에 대 한 오류 상태는 없으며이는 자문 hypercall 반환 됩니다.