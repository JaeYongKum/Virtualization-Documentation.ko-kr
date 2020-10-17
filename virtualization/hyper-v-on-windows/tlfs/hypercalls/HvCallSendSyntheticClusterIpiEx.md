---
title: HvCallSendSyntheticClusterIpiEx
description: HvCallSendSyntheticClusterIpiEx hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: e6f2f46892a1a56bf13520ed94b81f6fce0a217b
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121007"
---
# <a name="hvcallsendsyntheticclusteripiex"></a>HvCallSendSyntheticClusterIpiEx

이 hypercall는 지정 된 가상 프로세서 집합에 가상 고정 인터럽트를 보냅니다. NMIs을 지원 하지 않습니다. 이 버전은 가변 크기의 VP 집합을 지정할 수 있다는 점에서 [HvCallSendSyntheticClusterIpi](HvCallSendSyntheticClusterIpi.md) 와 다릅니다.

이 hypercall의 가용성을 유추 하려면 다음 검사를 사용 해야 합니다.

- ExProcessorMasks는 CPUID 리프 0x40000004를 통해 표시 되어야 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallSendSyntheticClusterIpiEx(
    _In_ UINT32 Vector,
    _In_ HV_INPUT_VTL TargetVtl,
    _In_ HV_VP_SET ProcessorSet
    );
 ```

## <a name="call-code"></a>호출 코드
`0x0015` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `Vector`                | 0          | 4        | 벡터가 어설션된로 지정 되었습니다. >= 0x10과 <= 0xFF 사이 여야 합니다.  |
| `TargetVtl`             | 4          | 1        | 대상 VTL                                |
| 안쪽 여백                 | 5          | 3        |                                           |
| `ProcessorSet`          | 8          | 변수 | 대상으로 지정할 VPs을 나타내는 프로세서 집합을 지정 합니다.|

## <a name="see-also"></a>추가 정보

[HV_VP_SET](../datatypes/HV_VP_SET.md)
