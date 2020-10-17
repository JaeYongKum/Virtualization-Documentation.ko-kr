---
title: HV_MESSAGE
description: HV_MESSAGE
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: af5a6a314f51178021ffaa7b3cc07c8baa016990
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121159"
---
# <a name="hv_message"></a>HV_MESSAGE

Syic 메시지는 메시지 헤더 (메시지 유형 및 메시지의 시작 위치에 대 한 정보 포함)와 페이로드가 구성 된 고정 크기입니다. [HvCallPostMessage](../hypercalls/HvCallPostMessage.md) 에 대 한 응답으로 전송 되는 메시지에는 포트 ID가 포함 됩니다. 가로채기 메시지는 가상 프로세서가 절편을 생성 한 파티션의 파티션 ID를 포함 합니다. 타이머 가로채기에는 발생 ID (즉, 지정 된 ID가 0)가 없습니다.

MessagePending 플래그는 가상 인터럽트 원본의 메시지 큐에 보류 중인 메시지가 있는지 여부를 나타냅니다. 메시지가 있는 경우 메시지 슬롯을 비운 후 게스트가 "메시지 끝"을 수행 해야 합니다. 이를 통해 EOM MSR (필요한 경우에만)에 대 한 oplocks를 수행할 수 있습니다. 이 플래그는 메시지 배달 시 하이퍼바이저에 의해 설정 되거나 나중에 언제 든 지 설정할 수 있습니다. 플래그는 메시지 슬롯을 비운 후에 테스트 해야 하며, 설정 된 경우 보류 중인 메시지가 하나 이상 있고 "메시지 끝"이 수행 되어야 합니다.

## <a name="syntax"></a>구문

 ```c
#define HV_MESSAGE_SIZE 256
#define HV_MESSAGE_MAX_PAYLOAD_BYTE_COUNT 240
#define HV_MESSAGE_MAX_PAYLOAD_QWORD_COUNT 30

typedef struct
{
    UINT8 MessagePending:1;
    UINT8 Reserved:7;
} HV_MESSAGE_FLAGS;

typedef struct
{
    HV_MESSAGE_TYPE MessageType;
    UINT16 Reserved;
    HV_MESSAGE_FLAGS MessageFlags;
    UINT8 PayloadSize;
    union
    {
        UINT64 OriginationId;
        HV_PARTITION_ID Sender;
        HV_PORT_ID Port;
    };
} HV_MESSAGE_HEADER;

typedef struct
{
    HV_MESSAGE_HEADER Header;
    UINT64 Payload[HV_MESSAGE_MAX_PAYLOAD_QWORD_COUNT];
} HV_MESSAGE;
 ```
