---
title: 데이터 형식
description: 하이퍼바이저 데이터 형식
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 068d1c93d42422ac0c255c7201178a33040f3b47
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121000"
---
# <a name="data-types"></a>데이터 형식

## <a name="reserved-values"></a>예약 된 값

이 사양에서는 일부 필드를 "예약 됨"으로 문서화 합니다. 이후 버전의 하이퍼바이저 아키텍처에서는 이러한 필드에 특정 의미가 제공 될 수 있습니다. 최대의 이전 버전과의 호환성을 위해 하이퍼바이저 인터페이스의 클라이언트는이 문서에 제공 된 지침을 따라야 합니다. 일반적으로 두 가지 형태의 지침이 제공 됩니다. 값 유지 (다이어그램 및 코드 세그먼트의 ReservedP에 RsvdP로 설명 됨) – 최대의 이후 버전과의 호환성을 위해 클라이언트는이 필드의 값을 유지 해야 합니다. 일반적으로이 작업은 현재 값을 읽고, 예약 되지 않은 필드의 값을 수정 하 고, 값을 다시 쓰는 방법으로 수행 됩니다. 0 값 (다이어그램 및 코드 세그먼트에서 RsvdZ로 설명 됨)-최대 전방 호환성을 위해 클라이언트는이 필드의 값을 0으로 지정 해야 합니다.

읽기 전용 구조 내의 예약 된 필드는 단순히 다이어그램에서 Rsvd로 문서화 되며 단순히 코드 세그먼트에서 예약 된 것입니다. 최대 이후 버전과의 호환성을 위해 이러한 필드 내의 값을 무시 해야 합니다. 클라이언트는 이러한 값이 항상 0이 아닌 것으로 가정 하면 안 됩니다.

## <a name="simple-scalar-types"></a>단순 스칼라 형식

하이퍼바이저 데이터 형식은 간단한 스칼라 형식 UINT8, UINT16, UINT32, UINT64 및 UINT128에서 작성 됩니다. 이러한 각는 지정 된 비트 수를 사용 하 여 부호 없는 단순 정수 스칼라를 나타냅니다. 해당 하는 부호 있는 정수 스칼라 (INT8, INT16, INT32 및 INT64)도 정의 됩니다.
하이퍼바이저는 부동 소수점 명령 또는 부동 소수점 형식을 사용 하지 않습니다.

## <a name="hypercall-status-code"></a>Hypercall 상태 코드

모든 hypercall는 HV_STATUS 형식의 16 비트 상태 코드를 반환 합니다.

 ```c
typedef UINT16 HV_STATUS;
 ```

## <a name="memory-address-space-types"></a>메모리 주소 공간 유형

하이퍼바이저 아키텍처는 세 개의 독립 된 주소 공간을 정의 합니다.

- **SPAs (시스템 실제 주소)** 는 cpu에 표시 되는 기본 하드웨어의 실제 주소 공간을 정의 합니다. 전체 컴퓨터에 대 한 시스템 실제 주소 공간은 하나 뿐입니다.
- **게스트 실제 주소 (gpa)** 는 게스트의 실제 메모리 뷰를 정의 합니다. Gpa는 기본 SPAs에 매핑될 수 있습니다. 파티션당 하나의 게스트 실제 주소 공간이 있습니다.
- 게스트 **가상 주소 (GVAs)** 는 주소 변환을 사용 하도록 설정 하 고 유효한 게스트 페이지 테이블을 제공 하는 경우 게스트 내에서 사용 됩니다.

이러한 주소 공간의 세 가지 모두 크기가 최대 264 바이트입니다. 따라서 다음 형식이 정의 됩니다.

 ```c
typedef UINT64 HV_SPA;
typedef UINT64 HV_GPA;
typedef UINT64 HV_GVA;
 ```

많은 하이퍼바이저 인터페이스가 단일 바이트가 아닌 메모리 페이지에서 작동 합니다. 최소 페이지 크기는 아키텍처에 따라 달라 집니다. X 64의 경우 4K로 정의 됩니다.

 ```c
#define X64_PAGE_SIZE 0x1000

#define HV_X64_MAX_PAGE_NUMBER (MAXUINT64/X64_PAGE_SIZE)
#define HV_PAGE_SIZE X64_PAGE_SIZE
#define HV_LARGE_PAGE_SIZE X64_LARGE_PAGE_SIZE
#define HV_PAGE_MASK (HV_PAGE_SIZE - 1)

typedef UINT64 HV_SPA_PAGE_NUMBER;
typedef UINT64 HV_GPA_PAGE_NUMBER;
typedef UINT64 HV_GVA_PAGE_NUMBER;
typedef UINT32 HV_SPA_PAGE_OFFSET
typedef HV_GPA_PAGE_NUMBER *PHV_GPA_PAGE_NUMBER;
 ```

`HV_SPA`을로 변환 하려면 `HV_SPA_PAGE_NUMBER` 를으로 나눕니다 `HV_PAGE_SIZE` .

## <a name="structures-enumerations-and-bit-fields"></a>구조체, 열거형 및 비트 필드

이 사양에서 나중에 정의 된 많은 데이터 구조와 상수 값은 C 스타일 열거와 구조를 기준으로 정의 됩니다. C 언어는 의도적으로 특정 구현 세부 정보를 정의 하지 않습니다. 그러나이 문서에서는 다음을 가정 합니다.

- "Enum" 키워드를 사용 하 여 선언 된 모든 열거형은 32 비트 부호 있는 정수 값을 정의 합니다.
- 모든 구조체는 필드가 자연스럽 게 정렬 되는 방식으로 채워집니다 (즉, 8 바이트 필드가 8 바이트의 오프셋에 정렬 됨).
- 모든 비트 필드는 패딩을 사용 하지 않고 하위 비트에서 상위 비트로 압축 됩니다.

## <a name="endianness"></a>endian

하이퍼바이저 인터페이스는 endian 중립적으로 설계 되었습니다 (즉, 하이퍼바이저를 빅 endian 또는 작은 endian 시스템으로 이식할 수 있어야 함). 하지만이 사양에서 나중에 정의 된 일부 데이터 구조는 작은 endian 레이아웃을 가정 합니다. 이러한 데이터 구조는 빅 endian 포트가 시도 될 때 수정 해야 합니다.

## <a name="pointer-naming-convention"></a>포인터 명명 규칙

이 문서에서는 포인터 형식에 명명 규칙을 사용 합니다. 특히 정의 된 형식에 대 한 "P"는 해당 형식에 대 한 포인터를 나타냅니다. 정의 된 형식 앞에 있는 "PC"는 해당 형식의 상수 값에 대 한 포인터를 나타냅니다.