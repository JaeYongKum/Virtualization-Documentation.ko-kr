---
title: HvFlushVirtualAddressListEx
description: HvFlushVirtualAddressListEx hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 60d56bdc4735c9b16534d3c7ad813e69734254af
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120809"
---
# <a name="hvflushvirtualaddresslistex"></a>HvFlushVirtualAddressListEx

HvFlushVirtualAddressListEx hypercall는 [Hvcallflushvirtualaddresslist](HvCallFlushVirtualAddressList.md)와 유사 하지만 가변 크기가 지정 된 스파스 VP를 입력으로 사용할 수 있습니다.
이 hypercall의 가용성을 유추 하려면 다음 검사를 사용 해야 합니다.

- ExProcessorMasks는 CPUID 리프 0x40000004를 통해 표시 되어야 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallFlushVirtualAddressListEx(
    _In_ HV_ADDRESS_SPACE_ID AddressSpace,
    _In_ HV_FLUSH_FLAGS Flags,
    _In_ HV_VP_SET ProcessorSet,,
    _Inout_ PUINT32 GvaCount,
    _In_reads_(GvaCount) PCHV_GVA GvaRangeList
    );
 ```

## <a name="call-code"></a>호출 코드
`0x0014` Rep

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | 주소 공간 ID (CR3 값)를 지정 합니다. |
| `Flags`                 | 8          | 8        | 플러시 작업을 수정 하는 플래그 비트 집합입니다. |
| `ProcessorSet`          | 16         | 변수 | 플러시 작업의 영향을 받는 프로세서를 나타내는 프로세서 집합입니다. |

## <a name="input-list-element"></a>입력 목록 요소

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `GvaRange`              | 0          | 8        | GVA 범위                                 |

## <a name="see-also"></a>추가 정보

[HV_VP_SET](../datatypes/HV_VP_SET.md)
