---
title: 가상 프로세서
description: 하이퍼바이저 가상 프로세서 인터페이스
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: a27d41b5d459e75af0fcbaf80de02b8e07d66391
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120890"
---
# <a name="virtual-processors"></a>가상 프로세서

각 파티션에는 가상 프로세서가 0 개 이상 있을 수 있습니다.

## <a name="virtual-processor-indices"></a>가상 프로세서 인덱스

가상 프로세서는 파티션 ID 및 해당 프로세서 인덱스로 구성 된 튜플로 식별 됩니다. 프로세서 인덱스는 생성 될 때 가상 프로세서에 할당 되며, 가상 프로세서의 수명 동안 변경 되지 않습니다.

특정 상황에서 "ANY virtual processor"를 지정 하는 데 사용할 수 HV_ANY_VP 특수 값입니다. HV_VP_INDEX_SELF 값을 사용 하 여 고유한 VP 인덱스를 하나 지정할 수 있습니다.

```c
typedef UINT32 HV_VP_INDEX;

#define HV_ANY_VP ((HV_VP_INDEX)-1)

#define HV_VP_INDEX_SELF ((HV_VP_INDEX)-2)
 ```

가상 프로세서의 ID는 하이퍼바이저에서 정의한 MSR (모델 관련 레지스터) HV_X64_MSR_VP_INDEX을 통해 게스트에서 검색할 수 있습니다.

```c
#define HV_X64_MSR_VP_INDEX 0x40000002
 ```

## <a name="virtual-processor-idle-sleep-state"></a>가상 프로세서 유휴 상태 중지 상태

가상 프로세서는 가상 유휴 프로세서 전원 상태 또는 프로세서 절전 상태에 배치 될 수 있습니다. 이 향상 된 가상 유휴 상태를 사용 하면 가상 프로세서에서 인터럽트가 마스킹 된 경우에도 저전원 유휴 상태에 있는 가상 프로세서를 인터럽트 도착으로 해제 수 있습니다. 즉, 가상 유휴 상태를 사용 하면 게스트 파티션에서 운영 체제가 게스트 파티션에서 실행 될 때 사용할 수 없는 OS의 프로세서 전원 절약 기법을 활용할 수 있습니다.

AccessGuestIdleMsr 권한을 소유 하는 파티션은 하이퍼바이저에서 정의한 MSR에 대 한 읽기를 통해 가상 프로세서 유휴 절전 상태로의 진입을 트리거할 수 있습니다 `HV_X64_MSR_GUEST_IDLE` . 가상 프로세서에서 인터럽트가 사용 되는지 여부에 관계 없이 인터럽트가 도착 하면 가상 프로세서가 해제 됩니다.

## <a name="virtual-processor-assist-page"></a>가상 프로세서 지원 페이지

하이퍼바이저는 게스트 GPA 공간에 중첩 된 가상 프로세서 당 페이지를 제공 합니다. 이 페이지는 게스트 VP와 하이퍼바이저 간의 양방향 통신에 사용할 수 있습니다. 게스트 OS에는이 가상 VP 지원 페이지에 대 한 읽기/쓰기 권한이 있습니다.

게스트는 가상 VP 지원 MSR (0x40000073)에 기록 하 여 오버레이 페이지 (GPA space)의 위치를 지정 합니다. 가상 VP 지원 페이지 MSR의 형식은 다음과 같습니다.

| 비트      | 필드           | Description                                                                 |
|-----------|-----------------|-----------------------------------------------------------------------------|
| 0         | 사용          | VP 지원 페이지를 사용 하도록 설정 합니다.                                                  |
| 11:1      | RsvdP           | 예약됨                                                                    |
| 63:12     | PFN 페이지        | 가상 VP 지원 페이지 PFN                                                  |

## <a name="virtual-processor-run-time-register"></a>가상 프로세서 런타임 레지스터

하이퍼바이저의 스케줄러는 내부적으로 각 가상 프로세서가 코드를 실행 하는 동안 소비 하는 시간을 추적 합니다. 추적 된 시간은 가상 프로세서가 게스트 코드를 실행 하는 시간과 연결 된 논리 프로세서가 해당 게스트를 대신해 서 하이퍼바이저 코드를 실행 하는 데 소비한 시간을 조합한 것입니다. 이 누적 시간은 64 비트 읽기 전용 HV_X64_MSR_VP_RUNTIME 하이퍼바이저 MSR을 통해 액세스할 수 있습니다. 시간 수량은 (100ns 단위로 측정 됩니다.

## <a name="non-privileged-instruction-execution-prevention-npiep"></a>권한 없는 명령 실행 방지 (NPIEP)

NPIEP (권한 없는 명령 실행)는 사용자 모드 코드의 특정 지침 사용을 제한 하는 기능입니다. 특히이 기능을 사용 하도록 설정 하면 SIDT, SGDT, SLDT 및 STR 명령의 실행을 차단할 수 있습니다. 이러한 지침을 실행 하면 #GP 오류가 발생 합니다.

이 기능은 [HV_X64_MSR_NPIEP_CONFIG_CONTENTS](datatypes/HV_X64_MSR_NPIEP_CONFIG_CONTENTS.md)를 사용 하 여 VP 별로 구성 해야 합니다.

## <a name="guest-spinlocks"></a>게스트 Spinlock

일반적인 다중 프로세서 가능 운영 체제에서는 잠금을 사용 하 여 특정 작업의 원자성을 적용 합니다. 하이퍼바이저에서 제어 하는 가상 머신 내에서 이러한 운영 체제를 실행 하는 경우 잠금에 의해 보호 되는 이러한 중요 섹션은 임계 영역 코드에 의해 생성 된 가로채기로 확장할 수 있습니다. 핵심 섹션 코드도 하이퍼바이저 스케줄러에 의해 선점 될 수도 있습니다. 하이퍼바이저는 이러한 preemptions를 방지 하려고 하지만 발생할 수 있습니다. 따라서 잠금 홀더가 다시 예약 될 때까지 다른 잠금 contenders 회전을 종료 하므로 spinlock 획득 시간이 크게 연장 됩니다.

하이퍼바이저는 하이퍼바이저로의 과도 한 회전 상황을 나타내기 전에 spinlock 획득을 시도해 야 하는 횟수를 게스트 OS에 나타냅니다. 이 개수는 CPUID 리프 0x40000004에서 반환 됩니다.

[HvCallNotifyLongSpinWait](hypercalls/HvCallNotifyLongSpinWait.md) hypercall는 지원 게스트에 대 한 인터페이스를 제공 하 여 다중 프로세서 가상 컴퓨터 잠금의 통계 공평 속성을 개선 합니다. 게스트는 CPUID 리프 0x40000004에서 반환 하는 권장 개수의 모든 배수로이 알림을 만들어야 합니다. 이 hypercall을 통해 게스트는 긴 spinlock 획득의 하이퍼바이저에 알립니다. 이를 통해 하이퍼바이저는 더 나은 일정 결정을 내릴 수 있습니다.