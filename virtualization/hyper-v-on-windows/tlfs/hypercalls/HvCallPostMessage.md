---
title: HvCallPostMessage
description: HvCallPostMessage hypercall
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7fc0693341316966f986bece60d007b651ce6870
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120753"
---
# <a name="hvcallpostmessage"></a>HvCallPostMessage

HvCallPostMessage hypercall는 지정 된 연결에 연결 된 대상 포트가 있는 메시지를 게시 (즉, 비동기적으로 보내기) 하려고 합니다. 메시지가 성공적으로 게시 되 면 해당 포트와 연결 된 파티션 내에서 가상 프로세서로 배달 될 수 있도록 큐에 대기 됩니다.

## <a name="interface"></a>인터페이스

 ```c
HV_STATUS
HvCallPostMessage(
    _In_ HV_CONNECTION_ID ConnectionId,
    _In_ HV_MESSAGE_TYPE MessageType,
    _In_ UINT32 PayloadSize,
    _In_reads_bytes_(PayloadSize) PCVOID Message
    );
 ```

## <a name="call-code"></a>호출 코드

`0x005C` 쉽게

## <a name="input-parameters"></a>입력 매개 변수

| Name                    | Offset     | 크기     | 제공된 정보                      |
|-------------------------|------------|----------|-------------------------------------------|
| `ConnectionId`          | 0          | 4        | 연결의 ID를 지정 합니다.       |
| RsvdZ                   | 4          | 4        |                                           |
| `MessageType`           | 8          | 4        | 메시지 헤더에 표시 되는 메시지 유형을 지정 합니다. 호출자는 0을 제외 하 고 가장 중요 한 비트가 지워지는 32 비트 메시지 유형을 지정할 수 있습니다. |
| `PayloadSize`           | 12         | 4        | 메시지에 포함 되는 바이트 수를 지정 합니다. |
| `Message`               | 16         | 240      | 사용할는 메시지의 페이로드를 최대 240 바이트까지 말합니다. 처음 n 바이트만 대상 파티션으로 전송 됩니다. 여기서 n은 PayloadSize 매개 변수에 제공 됩니다. |

## <a name="return-values"></a>반환 값

| 상태 코드                         | 오류 조건                                       |
|-------------------------------------|-------------------------------------------------------|
| `HV_STATUS_ACCESS_DENIED`           | 호출자의 파티션에 PostMessages 권한이 없습니다. |
| `HV_STATUS_INVALID_CONNECTION_ID`   | 지정 된 연결 ID가 잘못 되었습니다.               |
| `HV_STATUS_INVALID_PORT_ID`         | 지정 된 연결과 연결 된 포트가 삭제 되었습니다. |
|                                     | 지정 된 연결과 관련 된 포트가 "활성" 상태가 아닌 파티션에 속해 있습니다. |
|                                     | 지정 된 연결과 관련 된 포트가 "message" 유형 포트가 아닙니다. |
| `HV_STATUS_INVALID_PARAMETER`       | 지정 된 메시지 유형의 가장 중요 한 비트가 설정 되어 있습니다. |
|                                     | MessageType 매개 변수는 값 0을 지정 합니다.  |
|                                     | 지정 된 페이로드 크기가 240 바이트를 초과 합니다.         |
| `HV_STATUS_INSUFFICIENT_BUFFERS`    | 포트에 사용할 수 있는 게스트 메시지 버퍼가 없습니다.      |
| `HV_STATUS_INVALID_VP_INDEX`        | 대상 VP가 더 이상 존재 하지 않거나 메시지를 게시할 수 있는 사용할 수 있는 VPs 없습니다. |
| `HV_STATUS_INVALID_SYNIC_STATE`     | 대상 VP의 Syic은 사용 하지 않도록 설정 되며 게시 된 메시지를 수락할 수 없습니다. |
|                                     | 대상 VP의 SIM 페이지를 사용할 수 없습니다.                 |