---
title: HvCallSendSyntheticClusterIpi
description: HvCallSendSyntheticClusterIpi hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 21d7a7f5f43123b984e64d76008978e152e4a8a6
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120745"
---
# <a name="hvcallsendsyntheticclusteripi"></a>HvCallSendSyntheticClusterIpi

이 hypercall는 지정 된 가상 프로세서 집합에 가상 고정 인터럽트를 보냅니다. NMIs을 지원 하지 않습니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallSendSyntheticClusterIpi(
    _In_ UINT32 Vector,
    _In_ HV_INPUT_VTL TargetVtl,
    _In_ UINT64 ProcessorMask
    );
 ```

## <a name="call-code"></a>호출 코드
`0x000b` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `Vector`                | 0          | 4        | 벡터가 어설션된로 지정 되었습니다. >= 0x10과 <= 0xFF 사이 여야 합니다.  |
| `TargetVtl`             | 4          | 1        | 대상 VTL                                |
| 안쪽 여백                 | 5          | 3        |                                           |
| `ProcessorMask`         | 8          | 1        | 대상으로 지정할 VPs을 나타내는 마스크를 지정 합니다.|
