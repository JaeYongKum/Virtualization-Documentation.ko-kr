---
title: HV_TIMER_MESSAGE_PAYLOAD
description: HV_TIMER_MESSAGE_PAYLOAD
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 48a66d841b20f575868a30f2f08d3377c724effd
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121080"
---
# <a name="hv_timer_message_payload"></a>HV_TIMER_MESSAGE_PAYLOAD

타이머 만료 메시지는 타이머 이벤트가 발생 하는 경우에 전송 됩니다. 이 구조는 메시지 페이로드를 정의 합니다.

## <a name="syntax"></a>구문

```c
typedef struct
{
    UINT32          TimerIndex;
    UINT32          Reserved;
    HV_NANO100_TIME ExpirationTime;     // When the timer expired
    HV_NANO100_TIME DeliveryTime;       // When the message was delivered
} HV_TIMER_MESSAGE_PAYLOAD;
 ```

타이머 인덱스는 메시지를 생성 한 가상 타이머 (0-3)의 인덱스입니다. 이를 통해 클라이언트는 동일한 인터럽트 벡터를 사용 하 고 메시지를 구분할 수 있도록 여러 타이머를 구성할 수 있습니다.

ExpirationTime는 파티션 참조 시간 카운터의 시간 기준을 사용 하 여 100 나노초 단위로 측정 된 타이머의 예상 만료 시간입니다. 만료 시간 이후 만료 메시지가 도착할 수 있습니다.

경우 deliverytime는 메시지가 SIM 페이지의 각 메시지 슬롯에 배치 되는 시간입니다. 시간은 파티션의 참조 시간 카운터를 기준으로 100 나노초 단위로 측정 됩니다.

## <a name="see-also"></a>추가 정보

 [HV_MESSAGE](HV_MESSAGE.md)