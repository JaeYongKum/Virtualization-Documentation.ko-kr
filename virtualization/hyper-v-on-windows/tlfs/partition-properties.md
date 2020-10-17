---
title: 파티션
description: 하이퍼바이저 파티션 인터페이스
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: c2a040603a5b0f3d354731ed3c2cc95f4c3da49b
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120814"
---
# <a name="partitions"></a>파티션

하이퍼바이저는 파티션 측면에서 격리를 지원 합니다. 파티션은 운영 체제가 실행 되는 하이퍼바이저에서 지 원하는 격리의 논리적 단위입니다.

## <a name="partition-privilege-flags"></a>파티션 권한 플래그

각 파티션에는 하이퍼바이저에 의해 할당 된 권한 집합이 있습니다. 권한은 가상 MSRs 또는 hypercall에 대 한 액세스를 제어 합니다.

파티션은 "하이퍼바이저 기능 식별" CPUID 리프 (0x40000003)을 통해 해당 권한을 쿼리할 수 있습니다. 모든 권한에 대 한 설명은 [HV_PARTITION_PRIVILEGE_MASK](datatypes/HV_PARTITION_PRIVILEGE_MASK.md) 를 참조 하세요.

## <a name="partition-crash-enlightenment"></a>파티션 크래시 계몽

하이퍼바이저는 크래시 계몽 기능을 포함 하는 게스트 파티션을 제공 합니다. 이 인터페이스를 사용 하면 게스트 파티션에서 실행 되는 운영 체제가 크래시 덤프 절차의 일부로 하이퍼바이저에 심각한 OS 조건에 대 한 법정 정보를 제공 하는 옵션을 사용할 수 있습니다. 옵션에는 게스트 충돌 매개 변수 MSRs의 콘텐츠 유지와 충돌 메시지 지정이 포함 됩니다. 그런 다음 하이퍼바이저는이 정보를 루트 파티션에서 로깅을 사용할 수 있도록 합니다. 이 메커니즘을 통해 가상화 호스트 관리자는 크래시 덤프에 대 한 게스트 파티션에 연결 된 영구 저장소 또는 충돌 하는 게스트 OS에 의해 저장 될 수 있는 핵심 덤프 정보를 검사할 필요 없이 게스트 OS 충돌 이벤트에 대 한 정보를 수집할 수 있습니다.

이 메커니즘의 가용성은 `CPUID.0x400003.EDX:10` GuestCrashMsrsAvailable 플래그를 통해 표시 됩니다. [기능 검색](feature-discovery.md)을 참조 하세요.

### <a name="guest-crash-enlightenment-interface"></a>게스트 크래시 계몽 인터페이스

게스트 크래시 계몽 인터페이스는 아래에 정의 된 것 처럼 6 개의 종합 MSRs를 통해 제공 됩니다.

 ```c
#define HV_X64_MSR_CRASH_P0 0x40000100
#define HV_X64_MSR_CRASH_P1 0x40000101
#define HV_X64_MSR_CRASH_P2 0x40000102
#define HV_X64_MSR_CRASH_P3 0x40000103
#define HV_X64_MSR_CRASH_P4 0x40000104
#define HV_X64_MSR_CRASH_CTL 0x40000105
 ```

#### <a name="guest-crash-control-msr"></a>게스트 충돌 컨트롤 MSR

게스트 충돌 제어 MSR HV_X64_MSR_CRASH_CTL는 게스트 파티션에서 하이퍼바이저의 게스트 크래시 기능을 확인 하 고 수행할 지정 된 작업을 호출 하는 데 사용할 수 있습니다. [HV_CRASH_CTL_REG_CONTENTS](datatypes/HV_CRASH_CTL_REG_CONTENTS.md) 데이터 구조는 MSR의 콘텐츠를 정의 합니다.

##### <a name="determining-guest-crash-capabilities"></a>게스트 충돌 기능 확인

게스트 충돌 기능을 확인 하기 위해 게스트 파티션이 HV_X64_MSR_CRASH_CTL 등록을 읽을 수 있습니다. 하이퍼바이저에서 지원 되는 작업 및 기능 집합이 보고 됩니다.

##### <a name="invoking-guest-crash-capabilities"></a>게스트 충돌 기능 호출

지원 되는 하이퍼바이저 게스트 충돌 동작을 호출 하기 위해 게스트 파티션은 원하는 작업을 지정 하 여 HV_X64_MSR_CRASH_CTL 등록에 기록 합니다. 두 가지 변형이 지원 됩니다. CrashNotify 및 CrashMessage는 CrashNotify와 함께 사용 됩니다. 각 게스트 크래시 발생에 대해 두 변형 중 하나를 지정 하 여 하나의 MSR에 대 한 단일 쓰기 HV_X64_MSR_CRASH_CTL 수행 해야 합니다.

| 게스트 충돌 동작  | Description                                                 |
|---------------------|-------------------------------------------------------------|
| CrashMessage        | 이 작업은 하이퍼바이저에 대 한 크래시 메시지를 지정 하기 위해 CrashNotify와 함께 사용 됩니다. 이 값을 선택 하면 P3 및 P4의 값이 메시지의 위치와 크기로 처리 됩니다. HV_X64_MSR_CRASH_P3는 메시지의 게스트 실제 주소이 고 HV_X64_MSR_CRASH_P4는 메시지의 길이 (최대 4096 바이트)입니다. |
| CrashNotify         | 이 작업은 게스트 파티션이 원하는 데이터를 게스트 크래시 매개 변수 MSRs (즉, P0를 통한 P0)에 쓰기 완료 했음을 하이퍼바이저에 표시 하 고, 하이퍼바이저는 이러한 MSRs의 콘텐츠를 기록 하는 과정을 진행 해야 합니다. |