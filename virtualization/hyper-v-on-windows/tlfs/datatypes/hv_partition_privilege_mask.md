---
title: HV_PARTITION_PRIVILEGE_MASK
description: HV_PARTITION_PRIVILEGE_MASK
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 2efabf3eedba77d9b3a51e8ee130ba3a0b975ccd
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121135"
---
# <a name="hv_partition_privilege_mask"></a>HV_PARTITION_PRIVILEGE_MASK

파티션은 "하이퍼바이저 기능 식별" CPUID 리프 (0x40000003)를 통해 해당 권한 마스크를 쿼리할 수 있습니다.

## <a name="syntax"></a>구문

```c
typedef struct
{
    // Access to virtual MSRs
    UINT64 AccessVpRunTimeReg:1;
    UINT64 AccessPartitionReferenceCounter:1;
    UINT64 AccessSynicRegs:1;
    UINT64 AccessSyntheticTimerRegs:1;
    UINT64 AccessIntrCtrlRegs:1;
    UINT64 AccessHypercallMsrs:1;
    UINT64 AccessVpIndex:1;
    UINT64 AccessResetReg:1;
    UINT64 AccessStatsReg:1;
    UINT64 AccessPartitionReferenceTsc:1;
    UINT64 AccessGuestIdleReg:1;
    UINT64 AccessFrequencyRegs:1;
    UINT64 AccessDebugRegs:1;
    UINT64 AccessReenlightenmentControls:1
    UINT64 Reserved1:18;

    // Access to hypercalls
    UINT64 CreatePartitions:1;
    UINT64 AccessPartitionId:1;
    UINT64 AccessMemoryPool:1;
    UINT64 Reserved:1;
    UINT64 PostMessages:1;
    UINT64 SignalEvents:1;
    UINT64 CreatePort:1;
    UINT64 ConnectPort:1;
    UINT64 AccessStats:1;
    UINT64 Reserved2:2;
    UINT64 Debugging:1;
    UINT64 CpuManagement:1;
    UINT64 Reserved:1
    UINT64 Reserved:1;
    UINT64 Reserved:1;
    UINT64 AccessVSM:1;
    UINT64 AccessVpRegisters:1;
    UINT64 Reserved:1;
    UINT64 Reserved:1;
    UINT64 EnableExtendedHypercalls:1;
    UINT64 StartVirtualProcessor:1;
    UINT64 Reserved3:10;
} HV_PARTITION_PRIVILEGE_MASK;
 ```

각 권한은 가상 MSRs 또는 hypercall 집합에 대 한 액세스를 제어 합니다.

| 권한 플래그                        | 의미                                       |
|---------------------------------------|-----------------------------------------------|
|`AccessVpRunTimeReg`                   | 파티션이 가상 MSR HV_X64_MSR_VP_RUNTIME에 액세스할 수 있습니다. |
|`AccessPartitionReferenceCounter`      | 파티션은 파티션 전체 참조 수 MSR, HV_X64_MSR_TIME_REF_COUNT에 액세스할 수 있습니다. |
|`AccessSynicRegs`                      | 이 파티션에는 Syic와 연결 된 가상 MSRs (HV_X64_MSR_SCONTROL HV_X64_MSR_EOM를 통해 HV_X64_MSR_SINT0 HV_X64_MSR_SINT15)에 액세스할 수 있습니다.|
|`AccessSyntheticTimerMsrs`             | 파티션은 Syic (HV_X64_MSR_STIMER0_CONFIG HV_X64_MSR_STIMER3_COUNT)와 연결 된 가상 MSRs에 액세스할 수 있습니다. |
|`AccessIntrCtrlRegs`                   | 파티션은 APIC (HV_X64_MSR_EOI, HV_X64_MSR_ICR 및 HV_X64_MSR_TPR)와 연결 된 가상 MSRs에 액세스할 수 있습니다. |
|`AccessHypercallMsrs`                  | 파티션은 hypercall 인터페이스 (HV_X64_MSR_GUEST_OS_ID 및 HV_X64_MSR_HYPERCALL)와 관련 된 가상 MSRs에 액세스할 수 있습니다. |
|`AccessVpIndex`                        | 파티션은 가상 프로세서 인덱스를 반환 하는 가상 MSR에 액세스할 수 있습니다. |
|`AccessResetReg`                       | 이 파티션은 시스템을 다시 설정 하는 가상 MSR에 액세스할 수 있습니다. |
|`AccessStatsReg`                       | 이 파티션은 게스트에서 자체 통계 페이지를 매핑하고 매핑을 해제할 수 있는 가상 MSRs에 액세스할 수 있습니다. |
|`AccessPartitionReferenceTsc`          | 파티션은 참조 TSC 액세스할 수 있습니다. |
|`AccessGuestIdleReg`                   | 파티션이 게스트 유휴 상태로 전환 될 수 있는 가상 MSR에 액세스할 수 있습니다. |
|`AccessFrequencyRegs`                  | 파티션은 지원 되는 경우 TSC 및 APIC 주파수를 제공 하는 가상 MSRs에 액세스할 수 있습니다. |
|`AccessDebugRegs`                      | 파티션은 일부 형식의 게스트 디버깅에 사용 되는 가상 MSRs에 액세스할 수 있습니다. |
|`AccessReenlightenmentControls`        | 파티션은 reenlightenment 컨트롤에 액세스할 수 있습니다. |
|`CreatePartitions`                     | 파티션이 hypercall HvCallCreatePartition를 호출할 수 있습니다. 또한 파티션은 자식에서 작동 하도록 제한 된 다른 hypercall 만들 수 있습니다. |
|`AccessPartitionId`                    | 파티션이 hypercall HvCallGetPartitionId를 호출 하 여 자체 파티션 ID를 가져올 수 있습니다. |
|`AccessMemoryPool`                     | 파티션은 hypercall HvCallDepositMemory, HvCallWithdrawMemory 및 HvCallGetMemoryBalance를 호출할 수 있습니다. |
|`PostMessages`                         | 파티션이 hypercall [HvCallPostMessage](../hypercalls/HvCallPostMessage.md)를 호출할 수 있습니다. |
|`SignalEvents`                         | 파티션이 hypercall [HvCallSignalEvent](../hypercalls/HvCallSignalEvent.md)를 호출할 수 있습니다. |
|`CreatePort`                           | 파티션이 hypercall HvCallCreatePort를 호출할 수 있습니다.  |
|`PostMessages`                         | 파티션이 hypercall HvCallPostMessage를 호출할 수 있습니다. |
|`ConnectPort`                          | 파티션이 hypercall HvCallConnectPort를 호출할 수 있습니다. |
|`AccessStats`                          | 파티션은 hypercall HvCallMapStatsPage 및 HvCallUnmapStatsPage를 호출할 수 있습니다. |
|`Debugging`                            | 파티션은 hypercall HvCallPostDebugData, HvCallRetrieveDebugData 및 Hvcallpostdebugdata을 호출할 수 있습니다. |
|`CpuManagement`                        | 파티션은 CPU 관리를 위해 다양 한 hypercall를 호출할 수 있습니다. |
|`AccessVSM`                            | 파티션은 [VSM](../vsm.md)을 사용할 수 있습니다. |
|`AccessVpRegisters`                    | 파티션은 hypercall HvCallSetVpRegisters 및 HvCallGetVpRegisters를 호출할 수 있습니다. |
|`EnableExtendedHypercalls`             | 파티션은 [확장 된 hypercall 인터페이스](../hypercall-interface.md#extended-hypercall-interface)를 사용할 수 있습니다. |
|`StartVirtualProcessor`                | 파티션은 [Hvcallstartvirtualprocessor](../hypercalls/HvCallStartVirtualProcessor.md) 를 사용 하 여 가상 프로세서를 초기화할 수 있습니다. |
