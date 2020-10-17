---
title: HvCallModifyVtlProtectionMask
description: HvCallModifyVtlProtectionMask hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: a88dacf32a57eeb98c7971b5fd752dfe6562e1c8
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120737"
---
# <a name="hvcallmodifyvtlprotectionmask"></a>HvCallModifyVtlProtectionMask

HvCallModifyVtlProtectionMask hypercall는 기존 GPA 페이지 집합에 적용 된 VTL 보호를 수정 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvModifyVtlProtectionMask(
    _In_ HV_PARTITION_ID TargetPartitionId,
    _In_ HV_MAP_GPA_FLAGS MapFlags,
    _In_ HV_INPUT_VTL TargetVtl,
    _In_reads(PageCount) HV_GPA_PAGE_NUMBER GpaPageList
    );
 ```

VTL는 낮은 VTL에 보호를 추가할 수 있습니다.

비 RAM 범위에서 VTL 보호를 적용 하려는 시도는 HV_STATUS_INVALID_PARAMETER와 함께 실패 합니다.

## <a name="call-code"></a>호출 코드

`0x000C` Rep

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `TargetPartitionId`     | 0          | 8        | 이 요청에 대 한 파티션의 파티션 ID를 제공 합니다. |
| `MapFlags`              | 8          | 4        | 적용할 새 매핑 플래그를 지정 합니다. |
| `TargetVtl`             | 12         | 1        | 대상 VTL를 지정 했습니다.                 |
| RsvdZ                   | 13         | 3        |                                           |

## <a name="input-list-element"></a>입력 목록 요소

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `GpaPageList`           | 0          | 8        | 보호할 페이지를 제공 합니다.       |