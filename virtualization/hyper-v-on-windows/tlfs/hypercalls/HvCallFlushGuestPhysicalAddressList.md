---
title: HvCallFlushGuestPhysicalAddressList
description: HvCallFlushGuestPhysicalAddressList hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 990b89e1ba422fa169c3334e69de0f55dd609061
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120817"
---
# <a name="hvcallflushguestphysicaladdresslist"></a>HvCallFlushGuestPhysicalAddressList

HvCallFlushGuestPhysicalAddressList hypercall는 캐시 된 GVA/L2 GPA를 두 번째 수준 주소 공간의 일부에서 GPA 매핑으로 무효화 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallFlushGuestPhysicalAddressList(
    _In_ HV_SPA AddressSpace,
    _In_ UINT64 Flags,
    _In_reads_(RangeCount) PHV_GPA_PAGE_RANGE GpaRangeList
    );
 ```

이 hypercall는 중첩 된 가상화가 활성 상태인 경우에만 사용할 수 있습니다. 가상 TLB 무효화 작업은 모든 프로세서에 대해 작동 합니다.

이 호출은 시간 제어가 호출자에 게 반환 될 때 모든 플러시에 대 한 관찰 가능 효과가 발생 했음을 보장 합니다.
현재 .TLB가 "잠겨" 있으면 호출자의 가상 프로세서가 일시 중단 됩니다.

이 호출은 플러시할 L2 GPA 범위 목록을 사용 합니다. 각 범위에는 기본 L2 GPA 있습니다. 플러시는 페이지 세분성을 사용 하 여 수행 되므로 L2 GPA의 아래쪽 12 비트를 사용 하 여 범위 길이를 정의할 수 있습니다. 이러한 비트는 범위 내에서 초기 페이지를 벗어나 추가 페이지 수를 인코딩합니다. 이를 통해 각 항목은 1에서 4096 페이지로의 범위를 인코딩할 수 있습니다.

## <a name="call-code"></a>호출 코드

`0x00B0` Rep

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | 주소 공간 ID (EPT PML4 table 포인터)를 지정 합니다. |
| `Flags`                 | 8          | 8        | RsvdZ                                     |

## <a name="input-list-element"></a>입력 목록 요소

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `GpaRange`              | 0          | 8        | GPA 범위                                 |