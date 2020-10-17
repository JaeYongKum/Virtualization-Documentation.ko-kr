---
title: 중첩된 가상화
description: 중첩된 가상화
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 36fb58759f7236f5fac0a66a2b3621d35b382c28
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121215"
---
# <a name="nested-virtualization"></a>중첩된 가상화

중첩 된 가상화는 하드웨어 가상화 확장을 에뮬레이트하는 Hyper-v 하이퍼바이저를 참조 합니다. 이러한 에뮬레이트된 확장은 Hyper-v 플랫폼에서 실행 되는 다른 가상화 소프트웨어 (예: 중첩 된 하이퍼바이저)에서 사용할 수 있습니다.

이 기능은 게스트 파티션에만 사용할 수 있습니다. 가상 컴퓨터 마다 사용 하도록 설정 해야 합니다. 중첩 된 가상화는 Windows 루트 파티션에서 지원 되지 않습니다.

다음 용어는 다양 한 수준의 중첩 된 가상화를 정의 하는 데 사용 됩니다.

| 용어                   | 정의                                                              |
|------------------------|-------------------------------------------------------------------------|
| **L0 하이퍼바이저**      | 실제 하드웨어에서 실행 되는 Hyper-v 하이퍼바이저입니다.                    |
| **L1 루트**            | Windows 루트 운영 체제입니다.                                      |
| **L1 게스트**           | 중첩 된 하이퍼바이저가 없는 Hyper-v 가상 컴퓨터                  |
| **L1 하이퍼바이저**      | Hyper-v 가상 컴퓨터 내에서 실행 되는 중첩 된 하이퍼바이저입니다.           |
| **L2 루트**            | Hyper-v 가상 컴퓨터의 컨텍스트 내에서 실행 되는 루트 Windows 운영 체제입니다.|
| **L2 게스트**           | Hyper-v 가상 컴퓨터의 컨텍스트 내에서 실행 되는 중첩 된 가상 컴퓨터입니다. |

운영 체제 미 설치에 비해 하이퍼바이저는 VM에서 실행 되는 경우 상당한 성능 회귀를 유발할 수 있습니다. L1 하이퍼바이저는 L0 하이퍼바이저에서 제공 하는 지원 인터페이스를 사용 하 여 Hyper-v VM에서 실행 되도록 최적화할 수 있습니다.

## <a name="enlightened-vmcs"></a>지원 VMCS

Intel 플랫폼에서 가상화 소프트웨어는 가상 컴퓨터 제어 데이터 구조 (VMCSs)를 사용 하 여 가상화와 관련 된 프로세서 동작을 구성 합니다. VMPTRLD 명령을 사용 하 여 VMCSs를 활성화 하 고 VMREAD 및 WATCH-VMWRITE 명령을 사용 하 여 수정 해야 합니다. 이러한 지침은 중첩 된 가상화가 에뮬레이션 해야 하기 때문에 종종 심각한 병목 현상이 발생 합니다.

하이퍼바이저는 게스트 실제 메모리의 데이터 구조를 사용 하 여 가상화 관련 프로세서 동작을 제어 하는 데 사용할 수 있는 "지원 VMCS" 기능을 노출 합니다. 일반 메모리 액세스 명령을 사용 하 여이 데이터 구조를 수정할 수 있으므로 L1 하이퍼바이저가 VMREAD 또는 WATCH-VMWRITE 또는 VMPTRLD 명령을 실행할 필요가 없습니다.

L1 하이퍼바이저는 [가상 프로세서 지원 페이지](vp-properties.md#virtual-processor-assist-page)의 해당 필드에 1을 써서 지원 vmcss를 사용 하도록 선택할 수 있습니다. VP 지원 페이지의 다른 필드는 현재 활성화 된 지원 VMCS를 제어 합니다. 각 지원 VMCS는 크기는 정확히 하나의 페이지 (4kb) 이며 처음에는 0으로 계산 되어야 합니다. 지원 VMCS를 활성 또는 최신 상태로 만들려면 VMPTRLD 명령을 실행 해야 합니다.

L1 하이퍼바이저가 지원 VMCS를 사용 하 여 VM 항목을 수행 하 고 나면 VMCS가 프로세서에서 활성화 된 것으로 간주 됩니다. 지원 VMCS는 단일 프로세서 에서만 동시에 활성화 될 수 있습니다. L1 하이퍼바이저는 VMCLEAR 명령을 실행 하 여 지원 VMCS를 활성에서 비 활성 상태로 전환할 수 있습니다. 지원 VMCS가 활성화 된 상태에서 VMREAD 또는 WATCH-VMWRITE 지침은 지원 되지 않으며 예기치 않은 동작이 발생할 수 있습니다.

[HV_VMX_ENLIGHTENED_VMCS](datatypes/HV_VMX_ENLIGHTENED_VMCS.md) 구조는 지원 vmcs의 레이아웃을 정의 합니다. 모든 비 가상 필드는 Intel 실제 VMCS 인코딩에 매핑됩니다.

### <a name="feature-discovery"></a>기능 검색

지원 VMCS 인터페이스에 대 한 지원은 CPUID 리프 0x40000004로 보고 됩니다.

지원 VMCS 구조는 이후 변경 내용을 고려 하 여 버전을 지정 합니다. 각 지원 VMCS 구조에는 L0 하이퍼바이저에서 보고 하는 버전 필드가 포함 됩니다.

현재 지원 되는 VMCS 버전은 1 뿐입니다.

### <a name="clean-fields"></a>필드 정리

L0 하이퍼바이저는 지원 VMCS의 일부를 캐시 하도록 선택할 수 있습니다. 지원 VMCS clean fields는 중첩 된 VM 항목의 게스트 메모리에서 다시 로드 되는 지원 VMCS의 부분을 제어 합니다. L1 하이퍼바이저는 지원 VMCS를 수정할 때마다 해당 VMCS 정리 필드를 지워야 합니다. 그러지 않으면 L0 하이퍼바이저에서 오래 된 버전을 사용할 수 있습니다.

정리 필드 계몽은 지원 VMCS의 가상 "CleanFields" 필드를 통해 제어 됩니다. 기본적으로 모든 비트는 L0 하이퍼바이저가 중첩 된 각 VM 항목에 대해 해당 VMCS 필드를 다시 로드 해야 하도록 설정 됩니다.

## <a name="enlightened-msr-bitmap"></a>지원 MSR 비트맵

Intel 플랫폼에서 L0 하이퍼바이저는 가상화 소프트웨어에서 가로채기를 생성 하는 MSR 액세스를 제어할 수 있도록 하는 .VMX "MSR-비트맵 주소" 컨트롤을 에뮬레이트합니다.

L1 하이퍼바이저는 L0 하이퍼바이저와 공동 작업 하 여 MSR 액세스의 효율성을 높일 수 있습니다. 지원 VMCS의 해당 필드를 1로 설정 하 여 지원 MSR 비트맵을 사용 하도록 설정할 수 있습니다. 사용 하도록 설정 하면 L0 하이퍼바이저가 변경 내용에 대 한 MSR 비트맵을 모니터링 하지 않습니다. 대신, L1 하이퍼바이저는 MSR 비트맵 중 하나를 변경한 후 해당 정리 필드를 무효화 해야 합니다.

## <a name="compatibility-with-live-migration"></a>실시간 마이그레이션와의 호환성

Hyper-v에는 한 호스트에서 다른 호스트로 자식 파티션을 실시간으로 마이그레이션할 수 있는 기능이 있습니다. 실시간 마이그레이션은 일반적으로 자식 파티션에 투명 합니다. 그러나 중첩 된 가상화의 경우 L1 하이퍼바이저는 마이그레이션을 인식 해야 할 수 있습니다.

### <a name="live-migration-notification"></a>실시간 마이그레이션 알림

L1 하이퍼바이저는 파티션이 마이그레이션될 때 알리도록 요청할 수 있습니다. 이 기능은 CPUID에서 "AccessReenlightenmentControls" 권한으로 열거 됩니다. L0 하이퍼바이저는 L1 하이퍼바이저에서 인터럽트 벡터 및 대상 프로세서를 구성 하는 데 사용할 수 있는 가상 MSR (HV_X64_MSR_REENLIGHTENMENT_CONTROL)을 노출 합니다. L0 하이퍼바이저는 각 마이그레이션 후 지정 된 벡터를 사용 하 여 인터럽트를 삽입 합니다.

```c
#define HV_X64_MSR_REENLIGHTENMENT_CONTROL (0x40000106)

typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Vector :8;
        UINT64 RsvdZ1 :8;
        UINT64 Enabled :1;
        UINT64 RsvdZ2 :15;
        UINT64 TargetVp :32;
    };
} HV_REENLIGHTENMENT_CONTROL;
```

지정 된 벡터는 고정 된 APIC 인터럽트와 일치 해야 합니다. TargetVp 가상 프로세서 인덱스를 지정 합니다.

### <a name="tsc-emulation"></a>TSC 에뮬레이션

게스트 파티션은 서로 다른 TSC 주파수를 사용 하는 두 컴퓨터 간에 실시간으로 마이그레이션될 수 있습니다. 이러한 경우 [참조 TSC 페이지](timers.md#partition-reference-time-enlightenment) 의 TscScale 값을 다시 계산 해야 할 수 있습니다.

L0 하이퍼바이저는 경우에 따라 L1 하이퍼바이저가 TscScale 값을 다시 계산할 기회가 될 때까지 마이그레이션 후에 모든 TSC 액세스를 에뮬레이션 합니다. L1 하이퍼바이저는 HV_X64_MSR_TSC_EMULATION_CONTROL MSR에 기록 하 여 TSC 에뮬레이션을 옵트인 (opt in) 할 수 있습니다. 옵트인 (opt in) 인 경우,이 경우에는 L0 하이퍼바이저가 마이그레이션이 수행 된 후에 TSC 액세스를 에뮬레이트합니다.

L1 하이퍼바이저는 HV_X64_MSR_TSC_EMULATION_STATUS MSR을 사용 하 여 TSC 액세스를 현재 에뮬레이트하는 경우 쿼리할 수 있습니다. 예를 들어 L1 하이퍼바이저는 [실시간 마이그레이션 알림을](#live-migration-notification) 구독 하 고 마이그레이션 인터럽트를 받은 후 TSC 상태를 쿼리할 수 있습니다. 이 MSR을 사용 하 여 TSC 에뮬레이션을 해제할 수도 있습니다 (TscScale 값을 업데이트 한 후).

```c
#define HV_X64_MSR_TSC_EMULATION_CONTROL (0x40000107)
#define HV_X64_MSR_TSC_EMULATION_STATUS (0x40000108)

typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Enabled :1;
        UINT64 RsvdZ :63;
    };
 } HV_TSC_EMULATION_CONTROL;

typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 InProgress : 1;
        UINT64 RsvdP1 : 63;
    };
} HV_TSC_EMULATION_STATUS;
```

## <a name="virtual-tlb"></a>가상 TLB

하이퍼바이저에서 노출 하는 가상 TLB를 확장 하 여 L2 Gpa에서 Gpa로의 변환을 캐시할 수 있습니다. 논리 프로세서의 TLB와 마찬가지로, 가상 TLB는 비 일관 된 캐시 이며, 이러한 비 일관성는 게스트에 게 표시 됩니다. 하이퍼바이저는 TLB를 관리 하는 작업을 노출 합니다.
Intel 플랫폼에서 L0 하이퍼바이저는 다음과 같은 추가 방법을 가상화 하 여 TLB를 관리 합니다.

- INVVPID 명령을 사용 하 여 캐시 된 GVA를 GPA 또는 SPA 매핑으로 무효화할 수 있습니다.
- INVEPT 명령을 사용 하 여 캐시 된 L2 GPA를 GPA 매핑으로 무효화할 수 있습니다.

### <a name="direct-virtual-flush"></a>직접 가상 플러시

하이퍼바이저는 운영 체제에서 가상 TLB를 보다 효율적으로 관리할 수 있도록 hypercall ([HvCallFlushVirtualAddressSpace](hypercalls/HvCallFlushVirtualAddressSpace.md), [HvCallFlushVirtualAddressSpaceEx](hypercalls/HvCallFlushVirtualAddressSpaceEx.md), [hvcallflushvirtualaddresslist](hypercalls/HvCallFlushVirtualAddressList.md)및 [Hvcallflushvirtualaddresslistex](hypercalls/HvCallFlushVirtualAddressListEx.md))를 노출 합니다. L1 하이퍼바이저는 게스트가 이러한 hypercall를 사용 하도록 허용 하 고 이러한 hypercall를 L0 하이퍼바이저로 처리 하는 책임을 위임 하도록 선택할 수 있습니다. 이렇게 하려면 지원 VMCSs 및 파티션 지원 페이지를 사용 해야 합니다.

지원 VMCSs를 사용 하는 경우 가상 TLB는 모든 캐시 된 매핑의 태그를 만든 지원 VMCSS의 식별자로 태그를 만듭니다. L2 게스트에서 직접 가상 플러시 hypercall에 대 한 응답으로 L0 하이퍼바이저는 지원 VMCSs에 의해 생성 된 모든 캐시 된 매핑을 무효화 합니다.

- VmId는 호출자의 VmId와 동일 합니다.
- VpId가 지정 된 ProcessorMask에 포함 되어 있거나 HV_FLUSH_ALL_PROCESSORS 지정 되어 있습니다.

#### <a name="configuration"></a>구성

가상 플러시 hypercall의 직접 처리는 다음을 통해 사용 하도록 설정 됩니다.

1. [가상 프로세서 지원 페이지](vp-properties.md#virtual-processor-assist-page) 의 NestedEnlightenmentsControl DirectHypercall 필드를 1로 설정 합니다.
2. 지원 VMCS의 NestedFlushVirtualHypercall 필드를 1로 설정 합니다.

L1 하이퍼바이저는이 기능을 사용 하도록 설정 하기 전에 지원 VMCS의 다음과 같은 추가 필드를 구성 해야 합니다.

- VpId: 지원 VMCS가 제어 하는 가상 프로세서의 ID입니다.
- VmId: 지원 VMCS가 속한 가상 컴퓨터의 ID입니다.
- PartitionAssistPage: 파티션 지원 페이지의 게스트 실제 주소입니다.

L1 하이퍼바이저는 또한 CPUID를 통해 게스트에 다음 기능을 노출 해야 합니다.

- UseHypercallForLocalFlush
- UseHypercallForRemoteFlush

#### <a name="partition-assist-page"></a>파티션 지원 페이지

파티션 지원 페이지는 L1 하이퍼바이저가 할당 해야 하는 메모리의 페이지 크기 정렬 된 페이지 크기 영역으로, 직접 플러시 hypercall를 사용 하기 전에 0입니다. GPA는 지원 VMCS의 해당 필드에 써야 합니다.

```c
struct
{
    UINT32 TlbLockCount;
} VM_PARTITION_ASSIST_PAGE;
```

#### <a name="synthetic-vm-exit"></a>가상 VM-Exit

호출자의 파티션 지원 페이지의 TlbLockCount가 0이 아닌 경우, L0 하이퍼바이저는 직접 가상 플러시 hypercall을 처리 한 후에 L1 하이퍼바이저에 대 한 가상 종료 이유가 포함 된 VM-Exit를 전달 합니다.

```c
#define HV_VMX_SYNTHETIC_EXIT_REASON_TRAP_AFTER_FLUSH 0x10000031
```

### <a name="second-level-address-translation"></a>두 번째 수준 주소 변환

게스트 파티션에 대해 중첩 된 가상화를 사용 하는 경우 파티션에 의해 노출 되는 MMU (메모리 관리 단위)는 두 번째 수준 주소 변환에 대 한 지원을 포함 합니다. 두 번째 수준 주소 변환은 L1 하이퍼바이저에서 실제 메모리를 가상화 하는 데 사용할 수 있는 기능입니다. 사용 중인 경우 게스트 물리적 주소 (Gpa)로 처리 되는 특정 주소는 l2 게스트 실제 주소 (L2 Gpa)로 처리 되 고 페이징 구조 집합을 순회 하 여 Gpa로 변환 됩니다.

L1 하이퍼바이저는 두 번째 수준 주소 공간을 사용 하는 방법 및 위치를 결정할 수 있습니다. 각 두 번째 수준 주소 공간은 게스트에서 정의한 64 비트 ID 값으로 식별 됩니다. Intel 플랫폼에서이 값은 EPT PML4 테이블의 주소와 동일 합니다.

#### <a name="compatibility"></a>호환성

Intel 플랫폼에서 하이퍼바이저에 의해 노출 되는 두 번째 수준 주소 변환 기능은 일반적으로 주소 변환에 대 한 .VMX 지원과 호환 됩니다. 그러나 다음과 같은 게스트 관찰 가능 차이점이 있습니다.

- 내부적으로 하이퍼바이저는 L2 Gpa를 SPAs로 변환 하는 섀도 페이지 테이블을 사용할 수 있습니다. 이러한 구현에서 이러한 섀도 페이지 테이블은 software to large TLBs로 표시 됩니다. 그러나 몇 가지 차이점은 관찰 가능 합니다. 첫 번째는 두 가상 프로세서 간에 섀도 페이지 테이블을 공유할 수 있는 반면 기존 TLBs는 프로세서 당 구조 이며 독립적입니다. 한 가상 프로세서의 페이지 액세스가 이후에 다른 가상 프로세서에서 사용 되는 섀도 페이지 테이블 항목을 채울 수 있으므로이 공유는 표시 될 수 있습니다.
- 일부 하이퍼바이저 구현은 게스트 페이지 테이블의 내부 쓰기 보호를 사용 하 여 내부 데이터 구조 (예: 섀도 페이지 테이블)에서 MMU 매핑을 지연 플러시할 수 있습니다. 이러한 테이블에 대 한 쓰기가 하이퍼바이저에 의해 투명 하 게 처리 되기 때문에이는 주로 게스트에 게 표시 되지 않습니다. 그러나 다른 파티션이나 장치에 의해 기본 GPA 페이지에 수행 된 쓰기는 적절 한 TLB 플러시를 트리거하지 않을 수 있습니다.
- 일부 하이퍼바이저 구현에서는 두 번째 수준 페이지 오류 ("EPT 위반")가 캐시 된 매핑을 무효화 하지 않을 수 있습니다.

#### <a name="enlightened-second-level-tlb-flushes"></a>지원 두 번째 수준 TLB 플러시

또한 하이퍼바이저는 게스트에서 두 번째 수준 TLB를 보다 효율적으로 관리할 수 있는 향상 된 기능 집합을 지원 합니다. 이러한 향상 된 작업은 레거시 TLB 관리 작업과 교대로 사용할 수 있습니다.

하이퍼바이저는 다음 hypercall를 지원 하 여 TLBs를 무효화 합니다.

| Hypercall                                                                           | Description                                     |
|-------------------------------------------------------------------------------------|-------------------------------------------------|
| [HvCallFlushGuestPhysicalAddressSpace](hypercalls/HvCallFlushGuestPhysicalAddressSpace.md)      | 캐시 된 L2 GPA를 두 번째 수준 주소 공간 내의 GPA 매핑으로 무효화 합니다.   |
| [HvCallFlushGuestPhysicalAddressList](hypercalls/HvCallFlushGuestPhysicalAddressList.md)  | 캐시 된 GVA/L2 GPA를 두 번째 수준 주소 공간의 일부에서 GPA 매핑으로 무효화 합니다.    |

## <a name="nested-virtualization-exceptions"></a>중첩 된 가상화 예외

L1 하이퍼바이저는 페이지 폴트 예외 클래스에서 가상화 예외를 결합 하도록 옵트인 (opt in) 할 수 있습니다. L0 하이퍼바이저는 하이퍼바이저 중첩 된 가상화 기능 CPUID 리프에서이 enlightment에 대 한 지원을 보급 합니다.

VirtualizationException를 "1"로 설정 하 여 가상 프로세서 지원 페이지의 [HV_NESTED_ENLIGHTENMENTS_CONTROL](datatypes/HV_NESTED_ENLIGHTENMENTS_CONTROL.md) 데이터 구조에서이 계몽을 사용 하도록 설정할 수 있습니다.

## <a name="nested-virtualization-msrs"></a>중첩 된 가상화 MSRs

### <a name="nested-vp-index-register"></a>중첩 된 VP 인덱스 레지스터

L1 하이퍼바이저는 현재 프로세서의 기본 VP 인덱스를 보고 하는 MSR을 노출 합니다.

| MSR 주소      | 이름 등록              | Description                                                                 |
|------------------|----------------------------|-----------------------------------------------------------------------------|
| 0x40001002       | HV_X64_MSR_NESTED_VP_INDEX | 중첩 된 루트 파티션에서는 현재 프로세서의 기본 VP 인덱스를 보고 합니다. |

### <a name="nested-synic-msrs"></a>중첩 된 Syic MSRs

중첩 된 루트 파티션에서 다음 MSRs를 사용 하면 기본 하이퍼바이저의 해당 [Syic MSRs](inter-partition-communication.md#synic-msrs) 에 액세스할 수 있습니다.

기본 프로세서의 인덱스를 찾으려면 호출자는 먼저 HV_X64_MSR_NESTED_VP_INDEX를 사용 해야 합니다.

| MSR 주소      | 이름 등록                 | 기본 MSR                                              |
|------------------|-------------------------------|--------------------------------------------------------------|
| 0x40001080       | HV_X64_MSR_NESTED_SCONTROL    | HV_X64_MSR_SCONTROL                                          |
| 0x40001081       | HV_X64_MSR_NESTED_SVERSION    | HV_X64_MSR_SVERSION                                          |
| 0x40001082       | HV_X64_MSR_NESTED_SIEFP       | HV_X64_MSR_SIEFP                                             |
| 0x40001083       | HV_X64_MSR_NESTED_SIMP        | HV_X64_MSR_SIMP                                              |
| 0x40001084       | HV_X64_MSR_NESTED_EOM         | HV_X64_MSR_EOM                                               |
| 0x40001090       | HV_X64_MSR_NESTED_SINT0       | HV_X64_MSR_SINT0                                             |
| 0x40001091       | HV_X64_MSR_NESTED_SINT1       | HV_X64_MSR_SINT1                                             |
| 0x40001092       | HV_X64_MSR_NESTED_SINT2       | HV_X64_MSR_SINT2                                             |
| 0x40001093       | HV_X64_MSR_NESTED_SINT3       | HV_X64_MSR_SINT3                                             |
| 0x40001094       | HV_X64_MSR_NESTED_SINT4       | HV_X64_MSR_SINT4                                             |
| 0x40001095       | HV_X64_MSR_NESTED_SINT5       | HV_X64_MSR_SINT5                                             |
| 0x40001096       | HV_X64_MSR_NESTED_SINT6       | HV_X64_MSR_SINT6                                             |
| 0x40001097       | HV_X64_MSR_NESTED_SINT7       | HV_X64_MSR_SINT7                                             |
| 0x40001098       | HV_X64_MSR_NESTED_SINT8       | HV_X64_MSR_SINT8                                             |
| 0x40001099       | HV_X64_MSR_NESTED_SINT9       | HV_X64_MSR_SINT9                                             |
| 0x4000109A       | HV_X64_MSR_NESTED_SINT10      | HV_X64_MSR_SINT10                                            |
| 0x4000109B       | HV_X64_MSR_NESTED_SINT11      | HV_X64_MSR_SINT11                                            |
| 0x4000109C       | HV_X64_MSR_NESTED_SINT12      | HV_X64_MSR_SINT12                                            |
| 0x4000109D       | HV_X64_MSR_NESTED_SINT13      | HV_X64_MSR_SINT13                                            |
| 0x4000109E       | HV_X64_MSR_NESTED_SINT14      | HV_X64_MSR_SINT14                                            |
| 0x4000109F       | HV_X64_MSR_NESTED_SINT15      | HV_X64_MSR_SINT15                                            |
