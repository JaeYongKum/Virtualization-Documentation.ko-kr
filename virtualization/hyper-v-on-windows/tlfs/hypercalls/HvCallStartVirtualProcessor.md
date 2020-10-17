---
title: HvCallStartVirtualProcessor
description: HvCallStartVirtualProcessor hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 262ba48156d7b73d5a7aa3f7b72a12aacb09a2a0
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120997"
---
# <a name="hvcallstartvirtualprocessor"></a>HvCallStartVirtualProcessor

HvCallStartVirtualProcessor는 가상 프로세서를 시작 하기 위한 지원 메서드입니다. 부사장은 원하는 등록 상태로 시작할 수 있다는 점을 제외 하 고는 기존 INIT 기반 메서드와 기능적으로 동일 합니다.

0이 아닌 VTL에서 VP를 시작 하는 유일한 방법입니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallStartVirtualProcessor(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ HV_VP_INDEX VpIndex,
    _In_ HV_VTL TargetVtl,
    _In_ HV_INITIAL_VP_CONTEXT VpContext
    );
 ```

## <a name="call-code"></a>호출 코드

`0x0099` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `PartitionId`           | 0          | 8        | 파티션                                 |
| `VpIndex`               | 8          | 4        | 시작 하려면 VP 인덱스입니다. APIC ID에서 VP 인덱스를 가져오려면 HvGetVpIndexFromApicId을 사용 합니다. |
| `TargetVtl`             | 12         | 1        | 대상 VTL                                |
| `VpContext`             | 16         | 224      | VP를 시작 해야 하는 초기 컨텍스트를 지정 합니다. |

## <a name="return-values"></a>반환 값

| 상태 코드                         | 오류 조건                                       |
|-------------------------------------|-------------------------------------------------------|
| `HV_STATUS_ACCESS_DENIED`           | 액세스 거부됨                                         |
| `HV_STATUS_INVALID_PARTITION_ID`    | 지정 된 파티션 ID가 잘못 되었습니다.                |
| `HV_STATUS_INVALID_VP_INDEX`        | HV_VP_INDEX으로 지정 된 가상 프로세서가 잘못 되었습니다. |
| `HV_STATUS_INVALID_REGISTER_VALUE`  | 제공 된 레지스터 값이 잘못 되었습니다.               |
| `HV_STATUS_INVALID_VP_STATE`        | 지정 된 작업의 성능에 대 한 가상 프로세서가 올바른 상태가 아닙니다. |
| `HV_STATUS_INVALID_PARTITION_STATE` | 지정 된 파티션이 "활성" 상태가 아닙니다. |
| `HV_STATUS_INVALID_VTL_STATE`       | VTL 상태가 요청 된 작업과 충돌 합니다. |

## <a name="see-also"></a>추가 정보

[HV_INITIAL_VP_CONTEXT](../datatypes/HV_INITIAL_VP_CONTEXT.md)