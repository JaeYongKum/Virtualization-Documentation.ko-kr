---
title: HvCallGetVpRegisters
description: HvCallGetVpRegisters hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: fbddc59feefb9025a4abf27f8ae55524f0adb30a
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120742"
---
# <a name="hvcallgetvpregisters"></a>HvCallGetVpRegisters

HvCallGetVpRegisters hypercall는 가상 프로세서의 상태를 읽습니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallGetVpRegisters(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ HV_VP_INDEX VpIndex,
    _In_ HV_INPUT_VTL InputVtl,
    _Inout_ PUINT32 RegisterCount,
    _In_reads(RegisterCount) PCHV_REGISTER_NAME RegisterNameList,
    _In_writes(RegisterCount) PHV_REGISTER_VALUE RegisterValueList
    );
 ```

상태는 일련의 등록 값으로 반환 되며 각각은 입력으로 제공 된 레지스터 이름에 해당 합니다.

### <a name="restrictions"></a>제한

- 호출자는 PartitionId에서 지정한 파티션의 부모 이거나, 지정 된 파티션이 "self" 여야 하 고 파티션에 AccessVpRegisters 권한이 있어야 합니다.

## <a name="call-code"></a>호출 코드

`0x0050` Rep

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
| `RegisterName`          | 0          | 4        | 읽을 레지스터의 이름을 지정 합니다. |

## <a name="output-list-element"></a>출력 목록 요소

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `RegisterValue`         | 0          | 16       | 지정 된 레지스터의 값을 반환 합니다. |

## <a name="see-also"></a>추가 정보

[HV_REGISTER_NAME](../datatypes/HV_REGISTER_NAME.md)

[HV_REGISTER_VALUE](../datatypes/HV_REGISTER_VALUE.md)
