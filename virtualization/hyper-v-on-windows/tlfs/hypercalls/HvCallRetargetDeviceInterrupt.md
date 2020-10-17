---
title: HvCallRetargetDeviceInterrupt
description: HvCallRetargetDeviceInterrupt hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: df12f74a0eb34947e20afb61264c2434689b052f
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120726"
---
# <a name="hvcallretargetdeviceinterrupt"></a>HvCallRetargetDeviceInterrupt

이 hypercall은 게스트 내에서 Irq를 균형을 재조정 하는 데 유용할 수 있는 장치 인터럽트를 가져옵니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvRetargetDeviceInterrupt(
    _In_ HV_PARTITION_ID PartitionId,
    _In_ UINT64 DeviceId,
    _In_ HV_INTERRUPT_ENTRY InterruptEntry,
    _In_ UINT64 Reserved,
    _In_ HV_DEVICE_INTERRUPT_TARGET InterruptTarget
    );
 ```

## <a name="call-code"></a>호출 코드
`0x007e` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `PartitionId`           | 0          | 8        | 파티션 Id (HV_PARTITION_SELF 수 있음)   |
| `DeviceId`              | 8          | 6        | 호스트에서 할당 한 고유한 (게스트 내) 논리적 장치 ID를 제공 합니다.   |
| RsvdZ                   | 32         | 8        | 예약됨                                  |
| `InterruptEntry`        | 16         | 16       | 는 인터럽트를 식별 하는 MSI 주소와 데이터를 제공 합니다. |
| `InterruptTarget`       | 40         | 16       | 대상으로 지정할 VPs을 나타내는 마스크를 지정 합니다.|

## <a name="see-also"></a>추가 정보

[HV_INTERRUPT_ENTRY](../datatypes/HV_INTERRUPT_ENTRY.md)

[HV_DEVICE_INTERRUPT_TARGET](../datatypes/HV_DEVICE_INTERRUPT_TARGET.md)
