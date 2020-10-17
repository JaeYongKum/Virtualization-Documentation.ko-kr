---
title: HvCallGetVpIndexFromApicId
description: HvCallGetVpIndexFromApicId hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: d7dc0d9314e1a51a693ca041039e0d6386f24c0f
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120750"
---
# <a name="hvcallgetvpindexfromapicid"></a>HvCallGetVpIndexFromApicId

HvCallGetVpIndexFromApicId는 호출자가 지정 된 APID ID를 사용 하 여 VP의 VP 인덱스를 검색할 수 있도록 허용 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallGetVpIndexFromApicId(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ HV_VTL TargetVtl,
    _Inout_ PUINT32 ApicIdCoount,
    _In_reads_(ApicIdCount) PHV_APIC_ID ApicIdList,
    _Out_writes(ApicIdCount) PHV_VP_INDEX VpIndexList
    );

 ```

## <a name="call-code"></a>호출 코드

`0x009A` Rep

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `PartitionId`           | 0          | 8        | 파티션                                 |
| `TargetVtl`             | 8          | 1        | 대상 VTL                                |
| 안쪽 여백                 | 9          | 7        |                                           |

## <a name="input-list-element"></a>입력 목록 요소

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `ApicId`                | 0          | 4        | VP의 APIC ID                         |
| 안쪽 여백                 | 4          | 4        |                                           |

## <a name="output-list-element"></a>출력 목록 요소

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `VpIndex`               | 0          | 4        | 지정 된 APIC ID를 사용 하는 VP의 인덱스입니다.|
| 안쪽 여백                 | 4          | 4        |                                           |

## <a name="return-values"></a>반환 값

| 상태 코드                         | 오류 조건                                       |
|-------------------------------------|-------------------------------------------------------|
| `HV_STATUS_ACCESS_DENIED`           | 액세스 거부됨                                         |
| `HV_STATUS_INVALID_PARAMETER`       | 잘못 된 매개 변수가 지정 되었습니다.                    |
| `HV_STATUS_INVALID_PARTITION_ID`    | 지정 된 파티션 ID가 잘못 되었습니다.                |
| `HV_STATUS_INVALID_REGISTER_VALUE`  | 제공 된 레지스터 값이 잘못 되었습니다.               |
| `HV_STATUS_INVALID_VP_STATE`        | 지정 된 작업의 성능에 대 한 가상 프로세서가 올바른 상태가 아닙니다. |
| `HV_STATUS_INVALID_PARTITION_STATE` | 지정 된 파티션이 "활성" 상태가 아닙니다. |
| `HV_STATUS_INVALID_VTL_STATE`       | VTL 상태가 요청 된 작업과 충돌 합니다. |