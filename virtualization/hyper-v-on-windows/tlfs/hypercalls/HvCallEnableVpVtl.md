---
title: HvCallEnableVpVtl
description: HvCallEnableVpVtl hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 0be265f26c7c0e3ee844505eee4e8743ec01faaf
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121208"
---
# <a name="hvcallenablevpvtl"></a>HvCallEnableVpVtl

HvCallEnableVpVtl를 사용 하면 VTL를 VP에서 실행할 수 있습니다. 이 hypercall는 HvCallEnablePartitionVtl와 함께 사용 하 여 VTL를 사용 하도록 설정 하 고 사용 해야 합니다. VP에서 VTL을 사용 하도록 설정 하려면 먼저 파티션에 대해 사용 하도록 설정 해야 합니다. 이 호출은 활성 VTL를 변경 하지 않습니다.

## <a name="interface"></a>인터페이스

 ```c

HV_STATUS
HvEnableVpVtl(
    _In_ HV_PARTITION_ID TargetPartitionId,
    _In_ HV_VP_INDEX VpIndex,
    _In_ HV_VTL TargetVtl,
    _In_ HV_INITIAL_VP_CONTEXT VpVtlContext
    );
 ```

### <a name="restrictions"></a>제한

일반적으로 VTL는 상위 VTL만 사용할 수 있습니다. 이 규칙에는 한 가지 예외가 있습니다. 파티션에 대해 사용 하도록 설정 된 가장 높은 VTL는 더 높은 대상 VTL을 사용할 수 있습니다.

VP에서 대상 VTL를 사용 하는 경우 VTL를 사용 하도록 설정 하는 다른 모든 호출은 동일한 VTLs에서 제공 되어야 합니다.
이 hypercall는 이미 VP에 대해 사용 하도록 설정 된 VTL을 사용 하도록 설정 하기 위해 호출 된 경우 실패 합니다.

## <a name="call-code"></a>호출 코드
`0x000F` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `TargetPartitionId`     | 0          | 8        | 이 요청에 대 한 파티션의 파티션 ID를 제공 합니다. |
| `VpIndex`               | 8          | 4        | VTL를 사용 하도록 설정할 가상 프로세서의 인덱스를 지정 합니다. |
| `TargetVtl`             | 12         | 1        | 이 hypercall에서 사용할 VTL를 지정 합니다. |
| RsvdZ                   | 13         | 3        |                                           |
| `VpVtlContext`          | 16         | 224      | 대상 VTL의 첫 번째 항목에 대해 VP를 시작 해야 하는 초기 컨텍스트를 지정 합니다. |

## <a name="see-also"></a>추가 정보

[HV_INITIAL_VP_CONTEXT](../datatypes/HV_INITIAL_VP_CONTEXT.md)