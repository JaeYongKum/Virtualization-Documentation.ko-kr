---
title: HvCallSignalEvent
description: HvCallSignalEvent hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7d64700cbb2589e9d9b7780ea5d1acb2f6b8cfe3
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120887"
---
# <a name="hvcallsignalevent"></a>HvCallSignalEvent

HvCallSignalEvent hypercall는 지정 된 연결과 관련 된 포트를 소유 하는 파티션의 이벤트를 신호로 보냅니다.

수신 파티션의 가상 프로세서 중 하나의 SIEF 페이지에서 비트를 설정 하 여 이벤트에 신호를 보낼 수 있습니다. 호출자는 상대 플래그 번호를 지정 합니다. 실제 SIEF 비트 수는 포트와 연결 된 기본 플래그 번호에 지정 된 플래그 번호를 추가 하 여 하이퍼바이저에서 계산 됩니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallSignalEvent(
    _In_ HV_CONNECTION_ID ConnectionId,
    _In_ UINT16 FlagNumber
    );
 ```

## <a name="call-code"></a>호출 코드

`0x005D` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `ConnectionId`          | 0          | 4        | 연결의 ID를 지정 합니다.       |
| `FlagNumber`            | 4          | 2        | 호출자가 대상 SIEF 영역 내에서 설정 하려는 이벤트 플래그의 상대 인덱스를 지정 합니다. 이 숫자는 포트와 연결 된 기본 플래그 번호를 기준으로 합니다. |
| RsvdZ                   | 6          | 2        |                                           |

## <a name="return-values"></a>반환 값

| 상태 코드                         | 오류 조건                                       |
|-------------------------------------|-------------------------------------------------------|
| `HV_STATUS_ACCESS_DENIED`           | 호출자의 파티션에 SignalEvents 권한이 없습니다. |
| `HV_STATUS_INVALID_CONNECTION_ID`   | 지정 된 연결 ID가 잘못 되었습니다.               |
| `HV_STATUS_INVALID_PORT_ID`         | 지정 된 연결과 연결 된 포트가 삭제 되었습니다. |
|                                     | 지정 된 연결과 관련 된 포트가 "활성" 상태가 아닌 파티션에 속해 있습니다. |
|                                     | 지정 된 연결과 관련 된 포트가 "event" 유형 포트가 아닙니다. |
| `HV_STATUS_INVALID_PARAMETER`       | 지정 된 플래그 번호가 포트의 플래그 개수 보다 크거나 같습니다. |
| `HV_STATUS_INVALID_VP_INDEX`        | 대상 VP가 더 이상 존재 하지 않거나 메시지를 게시할 수 있는 사용할 수 있는 VPs 없습니다. |
| `HV_STATUS_INVALID_SYNIC_STATE`     | 대상 VP의 Syic은 사용 하지 않도록 설정 되며 신호를 받은 이벤트를 받아들일 수 없습니다. |
|                                     | 대상 VP의 SIEF 페이지가 사용 하지 않도록 설정 되었습니다.                 |