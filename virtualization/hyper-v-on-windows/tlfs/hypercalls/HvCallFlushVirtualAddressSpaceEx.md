---
title: HvCallFlushVirtualAddressSpaceEx
description: HvCallFlushVirtualAddressSpaceEx hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: cb2fed3873df1a768a2098e7f69a3896a33e004c
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120729"
---
# <a name="hvcallflushvirtualaddressspaceex"></a>HvCallFlushVirtualAddressSpaceEx

HvCallFlushVirtualAddressSpaceEx hypercall는 [HvCallFlushVirtualAddressSpace](HvCallFlushVirtualAddressSpace.md)와 비슷하지만 가변 크기의 스파스 VP를 입력으로 사용할 수 있습니다.

이 hypercall의 가용성을 유추 하려면 다음 검사를 사용 해야 합니다.

- ExProcessorMasks는 CPUID 리프 0x40000004를 통해 표시 되어야 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallFlushVirtualAddressSpaceEx(
    _In_ HV_ADDRESS_SPACE_ID AddressSpace,
    _In_ HV_FLUSH_FLAGS Flags,
    _In_ HV_VP_SET ProcessorSet
    );
 ```

## <a name="call-code"></a>호출 코드

`0x0013` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | 주소 공간 ID (CR3 값)를 지정 합니다. |
| `Flags`                 | 8          | 8        | 플러시 작업을 수정 하는 플래그 비트 집합입니다. |
| `ProcessorSet`          | 16         | 변수 | 플러시 작업의 영향을 받는 프로세서를 나타내는 프로세서 집합입니다. |

## <a name="see-also"></a>추가 정보

[HV_VP_SET](../datatypes/HV_VP_SET.md)
