---
title: HvCallEnablePartitionVtl
description: HvCallEnablePartitionVtl hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: fde108fff5a7b7ca9867234b8e8fcceb5a98722f
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120825"
---
# <a name="hvcallenablepartitionvtl"></a>HvCallEnablePartitionVtl

HvCallEnablePartitionVtl hypercall는 지정 된 파티션에 대 한 가상 신뢰 수준을 사용 하도록 설정 합니다. HvCallEnableVpVtl와 함께 사용 하 여 새 VTL를 시작 하 고 사용 해야 합니다.

## <a name="interface"></a>인터페이스

 ```c

typedef union
{
    UINT8 AsUINT8;
    struct {
        UINT8 EnableMbec:1;
        UINT8 Reserved:7;
    };
} HV_ENABLE_PARTITION_VTL_FLAGS;

HV_STATUS
HvCallEnablePartitionVtl(
    _In_ HV_PARTITION_ID TargetPartitionId,
    _In_ HV_VTL TargetVtl,
    _In_ HV_ENABLE_PARTITION_VTL_FLAGS Flags
    );
 ```

### <a name="restrictions"></a>제한

- 대상 VTL가 시작 VTL 보다 낮으면 시작 VTL는 항상 대상 VTL을 사용 하도록 설정할 수 있습니다.
- 시작 하는 VTL가 대상 VTL 보다 낮은 파티션에 대해 사용 하도록 설정 된 최고 VTL 설정 된 경우 시작 하는 VTL는 더 높은 대상을 사용할 수 있습니다.

## <a name="call-code"></a>호출 코드

`0x000D` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `TargetPartitionId`     | 0          | 8        | 이 요청에 대 한 파티션의 파티션 ID를 제공 합니다. |
| `TargetVtl`             | 8          | 1        | 이 hypercall에서 사용할 VTL를 지정 합니다. |
| `Flags`                 | 9          | 1        | VSM 관련 기능을 사용 하도록 설정 하는 마스크를 지정 합니다.|
| RsvdZ                   | 10         | 6        |                                           |