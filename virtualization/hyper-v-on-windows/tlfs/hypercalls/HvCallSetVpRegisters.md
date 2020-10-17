---
title: HvCallSetVpRegisters
description: HvCallSetVpRegisters hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 82b5aa39c1cf2d9e1372e9fba6c86579b11e15bf
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121207"
---
# <a name="hvcallsetvpregisters"></a>HvCallSetVpRegisters

HvCallSetVpRegisters hypercall는 가상 프로세서의 상태를 기록 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallSetVpRegisters(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ HV_VP_INDEX VpIndex,
    _In_ HV_INPUT_VTL InputVtl,
    _Inout_ PUINT32 RegisterCount,
    _In_reads(RegisterCount) PCHV_REGISTER_NAME RegisterNameList,
    _In_reads(RegisterCount) PCHV_REGISTER_VALUE RegisterValueList
    );
 ```

상태는 일련의 레지스터 값으로 기록 되며 각각은 입력으로 제공 된 레지스터 이름에 해당 합니다.

레지스터 값을 수정할 때 최소 오류 검사가 수행 됩니다. 특히 하이퍼바이저는 등록의 예약 비트가 0으로 설정 되어 있는지 확인 하 고, 항상 0 또는 1을 포함 하는 것으로 정의 된 비트의 비트는 적절 하 게 설정 되며, 레지스터의 아키텍처 크기를 벗어난 지정 된 비트는 0으로 설정 됩니다.

이 호출을 사용 하 여 읽기 전용 레지스터의 값을 수정할 수 없습니다.

레지스터 수정으로 인 한 부작용은 수행 되지 않습니다. 여기에는 예외 생성, 파이프라인 동기화, TLB 플러시 등이 포함 됩니다.

### <a name="restrictions"></a>제한

- 호출자는 PartitionId에서 지정한 파티션의 부모 이거나, 지정 된 파티션이 "self" 여야 하 고 파티션에 AccessVpRegisters 권한이 있어야 합니다.

## <a name="call-code"></a>호출 코드

`0x0051` Rep

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `PartitionId`           | 0          | 8        | 파티션 Id를 지정 합니다.               |
| `VpIndex`               | 8          | 4        | 가상 프로세서의 인덱스를 지정 합니다. |
| `TargetVtl`             | 12         | 1        | 대상 VTL를 지정 합니다.                 |
| RsvdZ                   | 13         | 3        |                                           |

## <a name="input-list-element"></a>입력 목록 요소

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `RegisterName`          | 0          | 4        | 수정할 레지스터의 이름을 지정 합니다. |
| RsvdZ                   | 4          | 12       |                                           |
| `RegisterValue`         | 16         | 16       | 지정 된 레지스터의 새 값을 지정 합니다. |

## <a name="see-also"></a>추가 정보

[HV_REGISTER_NAME](../datatypes/HV_REGISTER_NAME.md)

[HV_REGISTER_VALUE](../datatypes/HV_REGISTER_VALUE.md)
