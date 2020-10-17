---
title: HvFlushVirtualAddressList
description: HvFlushVirtualAddressList hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: af8f736dbe6926c5712a26cf6978ae0fe020b89d
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120822"
---
# <a name="hvflushvirtualaddresslist"></a>HvFlushVirtualAddressList

HvCallFlushVirtualAddressList hypercall는 지정 된 주소 공간에 속하는 가상 TLB의 부분을 무효화 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallFlushVirtualAddressList(
    _In_ HV_ADDRESS_SPACE_ID AddressSpace,
    _In_ HV_FLUSH_FLAGS Flags,
    _In_ UINT64 ProcessorMask,
    _Inout_ PUINT32 GvaCount,
    _In_reads_(GvaCount) PCHV_GVA GvaRangeList
    );
 ```

가상 TLB 무효화 작업은 하나 이상의 프로세서에 대해 작동 합니다.

플러시 해야 할 수 있는 프로세서에 대 한 정보가 게스트가 있는 경우 프로세서 마스크를 지정할 수 있습니다. 마스크의 각 비트는 가상 프로세서 인덱스에 해당 합니다. 예를 들어 0x0000000000000051의 마스크는 하이퍼바이저가 가상 프로세서 0, 4, 6의 TLB만 플러시해야 함을 나타냅니다.

다음 플래그를 사용 하 여 플러시 동작을 수정할 수 있습니다.

- HV_FLUSH_ALL_PROCESSORS 작업을 파티션 내의 모든 가상 프로세서에 적용 해야 함을 나타냅니다. 이 플래그가 설정 된 경우 ProcessorMask 매개 변수는 무시 됩니다.
- HV_FLUSH_ALL_VIRTUAL_ADDRESS_SPACES는 모든 가상 주소 공간에 작업을 적용 해야 함을 나타냅니다. 이 플래그가 설정 된 경우 AddressSpace 매개 변수는 무시 됩니다.
- HV_FLUSH_NON_GLOBAL_MAPPINGS_ONLY는이 호출에 적합 하지 않으며 잘못 된 옵션으로 처리 됩니다.

다른 모든 플래그는 예약 되어 있으므로 0으로 설정 해야 합니다.

이 호출은 GVA 범위 목록을 사용 합니다. 각 범위에는 기본 GVA가 있습니다. 페이지 세분성을 사용 하 여 플러시를 수행 하므로 GVA의 하위 12 비트를 사용 하 여 범위 길이를 정의할 수 있습니다. 이러한 비트는 범위 내에서 초기 페이지를 벗어나 추가 페이지 수를 인코딩합니다. 이를 통해 각 항목은 1에서 4096 페이지로의 범위를 인코딩할 수 있습니다.

"Large page" 매핑 (2MB 또는 4MB) 내에 있는 GVA는 가상 TLB에서 전체 페이지를 플러시하는 것을 유발 합니다.

이 호출은 시간 제어가 호출자에 게 다시 반환 될 때 지정 된 가상 프로세서의 모든 플러시에 대 한 관찰 가능한 효과를 발생 하도록 보장 합니다.

잘못 된 GVAs (파티션의 GVA 공간 끝을 벗어난 주소를 지정 하는)는 무시 됩니다.

대상 가상 프로세서의 TLB에 플러시가 필요 하 고 해당 가상 프로세서가 활용 하지 못해 TLB 플러시가 면 호출자의 가상 프로세서가 일시 중단 됩니다. TLB 플러시를 더 이상 사용할 수 없는 경우 가상 프로세서는 "일시 중단이 해제"이 고 hypercall가 다시 발급 됩니다.

## <a name="call-code"></a>호출 코드
`0x0003` Rep

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | 주소 공간 ID (CR3 값)를 지정 합니다. |
| `Flags`                 | 8          | 8        | 플러시 작업을 수정 하는 플래그 비트 집합입니다. |
| `ProcessorMask`         | 16         | 8        | 플러시 작업의 영향을 받을 프로세서를 나타내는 프로세서 마스크입니다. |

## <a name="input-list-element"></a>입력 목록 요소

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `GvaRange`              | 0          | 8        | GVA 범위                                 |
