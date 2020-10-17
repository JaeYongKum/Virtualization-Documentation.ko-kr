---
title: HvExtCallQueryCapabilities
description: HvExtCallQueryCapabilities hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 5e1a896b22c5087d3cd9041224846b94557f5c10
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120987"
---
# <a name="hvextcallquerycapabilities"></a>HvExtCallQueryCapabilities

이 hypercall는 확장 된 hypercall의 가용성을 보고 합니다.

이 hypercall의 가용성은 EnableExtendedHypercalls partition 권한 플래그를 사용 하 여 쿼리해야 합니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvExtCallQueryCapabilities(
    _Out_ UINT64 Capabilities
    );
 ```

## <a name="call-code"></a>호출 코드

`0x8001` 쉽게

## <a name="output-parameters"></a>출력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `Capabilities`          | 0          | 8        | 지원 되는 확장 된 hypercall의 비트 마스크                          |

지원 되는 확장 된 hypercall의 비트 마스크는 다음과 같은 형식입니다.

| bit     | 확장 된 Hypercall                                          |
|---------|-------------------------------------------------------------|
| 0       | HvExtCallGetBootZeroedMemory                                |
| 1       | HvExtCallMemoryHeatHint                                     |
| 2       | HvExtCallEpfSetup                                           |
| 3       | HvExtCallSchedulerAssistSetup                               |
| 4       | HvExtCallMemoryHeatHintAsync                                |
| 63-5    | 예약됨                                                    |
