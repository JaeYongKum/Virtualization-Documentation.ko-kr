---
title: HvCallFlushGuestPhysicalAddressSpace
description: HvCallFlushGuestPhysicalAddressSpace hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7c37a08bea56cce7460c89b2625ca9cc3e4239aa
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120830"
---
# <a name="hvcallflushguestphysicaladdressspace"></a>HvCallFlushGuestPhysicalAddressSpace

HvCallFlushGuestPhysicalAddressSpace hypercall는 캐시 된 L2 GPA를 두 번째 수준 주소 공간 내의 GPA 매핑으로 무효화 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallFlushGuestPhysicalAddressSpace(
    _In_ HV_SPA AddressSpace,
    _In_ UINT64 Flags
    );
 ```

이 hypercall는 중첩 된 가상화가 활성 상태인 경우에만 사용할 수 있습니다. 가상 TLB 무효화 작업은 모든 프로세서에 대해 작동 합니다.

Intel 플랫폼에서 HvCallFlushGuestPhysicalAddressSpace hypercall는 모든 프로세서에서 "단일 컨텍스트" 유형의 INVEPT 명령을 실행 하는 것과 같습니다.

이 호출은 시간 제어가 호출자에 게 반환 될 때 모든 플러시에 대 한 관찰 가능 효과가 발생 했음을 보장 합니다.
현재 .TLB가 "잠겨" 있으면 호출자의 가상 프로세서가 일시 중단 됩니다.

## <a name="call-code"></a>호출 코드

`0x00AF` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `AddressSpace`          | 0          | 8        | 주소 공간 ID (EPT PML4 table 포인터)를 지정 합니다. |
| `Flags`                 | 8          | 8        | RsvdZ                                     |