---
title: 가상 보안 모드
description: 가상 보안 모드
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: b858b24c8ddd672f8d109a1fbda4cb006437d68d
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120900"
---
# <a name="virtual-secure-mode"></a>가상 보안 모드

VSM (가상 보안 모드)은 호스트 및 게스트 파티션에 제공 되는 하이퍼바이저 기능 및 enlightenments의 집합으로, 운영 체제 소프트웨어 내에서 새로운 보안 경계를 만들고 관리할 수 있도록 합니다. VSM은 Device Guard, Credential Guard, virtual Tpm 및 차폐 Vm을 포함 하는 Windows 보안 기능을 기반으로 하는 하이퍼바이저 기능입니다. 이러한 보안 기능은 Windows 10 및 Windows Server 2016에서 도입 되었습니다.

VSM을 사용 하면 루트 및 게스트 파티션에서 운영 체제 소프트웨어를 사용 하 여 시스템 보안 자산을 저장 하 고 처리할 수 있도록 격리 된 메모리 영역을 만들 수 있습니다. 이러한 격리 된 영역에 대 한 액세스는 시스템의 신뢰할 수 있는 (TCB)의 매우 신뢰할 수 있는 매우 신뢰할 수 있는 부분인 하이퍼바이저를 통해서만 제어 되 고 부여 됩니다. 하이퍼바이저는 운영 체제 소프트웨어 보다 더 높은 권한 수준에서 실행 되 고 시스템 초기화 초기에 CPU MMU 및 IOMMU의 메모리 액세스 권한 제어와 같은 키 시스템 하드웨어 리소스를 단독으로 제어 하기 때문에, 하이퍼바이저는 감독자 모드 액세스를 사용 하는 운영 체제 소프트웨어 (예: OS 커널 및 장치 드라이버)에서와 같이 격리 된 영역을 무단 액세스 로부터 보호할 수 있습니다 (즉, CPL0 또는 "링 0").

이 아키텍처를 사용 하면 감독자 모드에서 실행 되는 정상적인 시스템 수준 소프트웨어 (예: 커널, 드라이버 등)가 악성 소프트웨어로 인해 손상 되는 경우에도 하이퍼바이저로 보호 되는 격리 된 지역의 자산은 보안을 유지할 수 있습니다.

## <a name="virtual-trust-level-vtl"></a>가상 신뢰 수준 (VTL)

VSM은 VTLs (가상 신뢰 수준)를 통해 격리를 달성 하 고 유지 관리 합니다. VTLs는 파티션당 및 가상 프로세서 별로 사용 하도록 설정 되 고 관리 됩니다.

가상 신뢰 수준은 계층 구조 이며 수준이 낮은 수준 보다 더 높은 수준의 권한입니다. VTL0는 최소 권한 수준으로, VTL1는 VTL0 보다 더 많은 권한을 보유 하 고 VTL2는 VTL1 보다 더 많은 권한이 있습니다.

구조적으로 최대 16 수준의 VTLs가 지원 됩니다. 그러나 하이퍼바이저는 VTL의 수를 16 개 미만으로 구현 하도록 선택할 수 있습니다. 현재는 두 개의 VTLs만 구현 됩니다.

 ```c
typedef UINT8 HV_VTL, *PHV_VTL;

#define HV_NUM_VTLS 2
#define HV_INVALID_VTL ((HV_VTL) -1)
#define HV_VTL_ALL 0xF
 ```

각 VTL에는 자체 메모리 액세스 보호 집합이 있습니다. 이러한 액세스 보호는 파티션의 실제 주소 공간에서 하이퍼바이저에 의해 관리 되므로 파티션에서 실행 되는 시스템 수준 소프트웨어로는 수정할 수 없습니다.

권한 있는 VTLs는 자체 메모리 보호를 적용할 수 있으므로 더 높은 VTLs는 더 낮은 VTLs에서 메모리 영역을 효과적으로 보호할 수 있습니다. 실제로이를 통해 더 낮은 VTL으로 보호 하 여 격리 된 메모리 영역을 VTL 수 있습니다. 예를 들어 VTL0는 VTL1에 비밀을 저장할 수 있으며,이 시점에서 VTL1만 액세스할 수 있습니다. VTL0가 손상 된 경우에도 보안은 안전 합니다.

### <a name="vtl-protections"></a>VTL 보호

VTLs 간에 격리를 달성 하는 여러 패싯이 있습니다.

- 메모리 액세스 보호: 각 VTL는 게스트의 실제 메모리 액세스 보호 집합을 유지 관리 합니다. 특정 VTL에서 실행 되는 소프트웨어는 이러한 보호에 따라 메모리에만 액세스할 수 있습니다.
- 가상 프로세서 상태: 가상 프로세서는 개별 VTL 상태를 유지 합니다. 예를 들어 각 VTL는 개인 VP 레지스터 집합을 정의 합니다. 낮은 VTL에서 실행 되는 소프트웨어는 상위 VTL의 개인 가상 프로세서의 등록 상태에 액세스할 수 없습니다.
- 인터럽트: 별도의 프로세서 상태와 함께 각 VTL에는 자체 인터럽트 하위 시스템 (로컬 APIC)도 있습니다. 이렇게 하면 더 높은 VTLs에서 낮은 VTL의 방해를 받지 않고 인터럽트를 처리할 수 있습니다.
- 오버레이 페이지: 더 높은 VTLs가 안정적으로 액세스할 수 있도록 특정 오버레이 페이지가 VTL에서 유지 관리 됩니다. 예를 들어 VTL 당 별도의 hypercall 오버레이 페이지가 있습니다.

## <a name="vsm-detection-and-status"></a>VSM 검색 및 상태

VSM 기능은 AccessVsm 파티션 권한 플래그를 통해 파티션에 보급 됩니다. 다음 권한이 모두 있는 파티션만 VSM: AccessVsm, AccessVpRegisters 및 Accesssyicregs를 활용할 수 있습니다.

### <a name="vsm-capability-detection"></a>VSM 기능 검색

게스트는 다음의 모델 관련 레지스터를 사용 하 여 VSM 기능에 대 한 보고서에 액세스 해야 합니다.

| MSR 주소      | 이름 등록                   | Description                                                    |
|------------------|---------------------------------|----------------------------------------------------------------|
| 0x000D0006       | HV_X64_REGISTER_VSM_CAPABILITIES| VSM 기능에 대해 보고 합니다.                                    |

VSM 기능 등록 MSR의 형식은 다음과 같습니다.

| 비트          | Description                         | 특성                                                  |
|---------------|-------------------------------------|-------------------------------------------------------------|
| 63            | Dr6Shared                           | 읽기                                                        |
| 62:47         | MbecVtlMask                         | 읽기                                                        |
| 46            | DenyLowerVtlStartup                 | 읽기                                                        |
| 45:0          | RsvdZ                               | 읽기                                                        |

Dr6Shared는 Dr6가 VTLs 간의 공유 레지스터 인지 여부를 게스트에 게 나타냅니다.

MvecVtlMask는 게스트에 Mbec을 사용할 수 있는 VTLs를 나타냅니다.

DenyLowerVtlStartup는 Vtl가 하위 VTL의 VP 재설정을 거부할 수 있는지 여부를 게스트에 게 나타냅니다.

### <a name="vsm-status-register"></a>VSM 상태 등록

파티션 권한 플래그 외에도 두 개의 가상 레지스터를 사용 하 여 VSM 상태 (및)에 대 한 추가 정보를 배울 수 있습니다. `HvRegisterVsmPartitionStatus` `HvRegisterVsmVpStatus`

#### <a name="hvregistervsmpartitionstatus"></a>HvRegisterVsmPartitionStatus

HvRegisterVsmPartitionStatus은 모든 VTLs에서 공유 되는 파티션당 읽기 전용 레지스터입니다. 이 등록은 파티션에 대해 사용 하도록 설정 된 VTLs에 대 한 정보를 제공 하며, VTLs에서 모드 기반 실행 컨트롤을 사용 하도록 설정 하 고, 허용 되는 최대 VTL에 대 한 정보를 제공 합니다.

```c
typedef union
{
    UINT64 AsUINT64;
    struct {
        UINT64 EnabledVtlSet : 16;
        UINT64 MaximumVtl : 4;
        UINT64 MbecEnabledVtlSet: 16;
        UINT64 ReservedZ : 28;
    };
} HV_REGISTER_VSM_PARTITION_STATUS;
```

#### <a name="hvregistervsmvpstatus"></a>HvRegisterVsmVpStatus

HvRegisterVsmVpStatus는 읽기 전용 레지스터 이며 모든 VTLs에서 공유 됩니다. 각 가상 프로세서는 고유한 인스턴스를 유지 관리 하는 VP 등록을 의미 합니다. 이 등록은 활성화 된 VTLs 및 VP에서 활성화 된 MBEC 모드에 대 한 정보를 제공 합니다.

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 ActiveVtl : 4;
        UINT64 ActiveMbecEnabled : 1;
        UINT64 ReservedZ0 : 11;
        UINT64 EnabledVtlSet : 16;
        UINT64 ReservedZ1 : 32;
    };
} HV_REGISTER_VSM_VP_STATUS;
```

ActiveVtl은 가상 프로세서에서 현재 활성 상태인 VTL 컨텍스트의 ID입니다.

ActiveMbecEnabled는 MBEC가 가상 프로세서에서 현재 활성 상태 인지 여부를 지정 합니다.

EnabledVtlSet은 가상 프로세서에서 사용 되는 VTL의 비트맵입니다.

### <a name="partition-vtl-initial-state"></a>Partition VTL 초기 상태

파티션이 시작 되거나 다시 설정 되 면 VTL0에서 실행 되기 시작 합니다. 다른 모든 VTLs는 파티션 생성 시 사용 하지 않도록 설정 됩니다.

## <a name="vtl-enablement"></a>VTL

VTL 사용을 시작 하려면 더 낮은 VTL가 다음을 시작 해야 합니다.

1. 파티션에 대 한 대상 VTL를 사용 하도록 설정 합니다. 이렇게 하면 VTL가 파티션에 일반적으로 사용할 수 있습니다.
2. 하나 이상의 가상 프로세서에서 대상 VTL를 사용 하도록 설정 합니다. 이렇게 하면 VTL를 VP에 사용할 수 있고 초기 컨텍스트를 설정 합니다. 모든 VPs에서 동일한 VTLs를 사용 하도록 설정 하는 것이 좋습니다. 일부 VPs (전체가 아닌)에서 VTL를 사용 하도록 설정 하면 예기치 않은 동작이 발생할 수 있습니다.
3. 파티션에 대해 VTL를 사용 하도록 설정 하 고 VP가 설정 되 면 EnableVtlProtection 플래그가 설정 되 면 액세스 보호를 설정 하기 시작할 수 있습니다.

VTLs는 연속 되지 않아도 됩니다.

### <a name="enabling-a-target-vtl-for-a-partition"></a>파티션에 대 한 대상 VTL 설정

[HvCallEnablePartitionVtl](hypercalls/HvCallEnablePartitionVtl.md) hypercall는 특정 파티션에 대 한 VTL를 설정 하는 데 사용 됩니다. 특정 VTL에서 소프트웨어를 실제로 실행 하기 전에 해당 VTL는 파티션의 가상 프로세서에서 사용 하도록 설정 해야 합니다.

### <a name="enabling-a-target-vtl-for-virtual-processors"></a>가상 프로세서에 대 한 대상 VTL 사용

파티션에 대해 VTL를 사용 하도록 설정 하면 파티션의 가상 프로세서에서 사용할 수 있습니다. [HvCallEnableVpVtl](hypercalls/HvCallEnableVpVtl.md) hypercall는 초기 컨텍스트를 설정 하는 가상 프로세서에 대해 vtls를 사용 하도록 설정 하는 데 사용할 수 있습니다.

가상 프로세서는 VTL 당 하나의 "context"를 가집니다. VTL 전환 되 면 VTL의 [개인 상태도](#private-state) 전환 됩니다.

## <a name="vtl-configuration"></a>VTL 구성

VTL 사용 하도록 설정 된 후에는 동일한 VTL 이상에서 실행 되는 VP에 의해 해당 구성을 변경할 수 있습니다.

### <a name="partition-configuration"></a>파티션 구성

HvRegisterVsmPartitionConfig register를 사용 하 여 파티션 차원의 특성을 구성할 수 있습니다. 모든 파티션에서 각 VTL (0 보다 큼)에 대해이 레지스터의 인스턴스가 하나씩 있습니다.

모든 VTL은 해당 HV_REGISTER_VSM_PARTITION_CONFIG의 고유한 인스턴스 뿐만 아니라 더 낮은 VTLs의 인스턴스도 수정할 수 있습니다. VTLs는 더 높은 VTLs에 대해이 등록을 수정할 수 없습니다.

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 EnableVtlProtection : 1;
        UINT64 DefaultVtlProtectionMask : 4;
        UINT64 ZeroMemoryOnReset : 1;
        UINT64 DenyLowerVtlStartup : 1;
        UINT64 ReservedZ : 2;
        UINT64 InterceptVpStartup : 1;
        UINT64 ReservedZ : 54; };
} HV_REGISTER_VSM_PARTITION_CONFIG;
```

이 레지스터의 필드는 아래에 설명 되어 있습니다.

#### <a name="enable-vtl-protections"></a>VTL 보호 사용

VTL를 사용 하도록 설정한 후에는 EnableVtlProtection 플래그를 설정 해야 메모리 보호 적용을 시작할 수 있습니다.
이 플래그는 한 번만 작성 됩니다. 즉, 설정 된 후에는 수정할 수 없습니다.

#### <a name="default-protection-mask"></a>기본 보호 마스크

기본적으로 시스템은 현재 매핑된 모든 페이지에 RWX 보호를 적용 하 고 향후 모든 "핫 추가" 페이지를 적용 합니다. 핫 추가 된 페이지는 크기 조정 작업을 수행 하는 동안 파티션에 추가 된 모든 메모리를 나타냅니다.

VTL에서 DefaultVtlProtectionMask HV_REGISTER_VSM_PARTITION_CONFIG를 지정 하 여 다른 기본 메모리 보호 정책을 설정할 수 있습니다. 이 마스크는 VTL 사용 하도록 설정할 때 설정 해야 합니다. 설정 된 후에는 변경할 수 없으며, 파티션을 다시 설정 해야만 지워집니다.

| bit       | Description                                                                          |
|-----------|--------------------------------------------------------------------------------------|
| 0         | 읽기                                                                                 |
| 1         | 쓰기                                                                                |
| 2         | 사용자 모드 실행 (UMX)                                                              |
| 3         | 커널 모드 실행 (KMX)                                                            |

#### <a name="zero-memory-on-reset"></a>다시 설정 시 메모리 0 개

ZeroMemOnReset는 파티션이 다시 설정 되기 전에 메모리를 0으로 설정 했는지 여부를 제어 하는 비트입니다. 이 구성은 기본적으로 설정 되어 있습니다. 비트가 설정 된 경우 더 높은 VTL의 메모리를 낮은 VTL으로 손상 시킬 수 없도록 파티션의 메모리가 다시 설정 될 때 0이 됩니다.
이 비트를 지우면 파티션의 메모리가 다시 설정 될 때 0으로 설정 되지 않습니다.

#### <a name="denylowervtlstartup"></a>DenyLowerVtlStartup

DenyLowerVtlStartup 플래그는 가상 프로세서를 더 낮은 VTLs로 시작 하거나 다시 설정할 수 있는지 여부를 제어 합니다. 여기에는 가상 프로세서를 다시 설정 하는 아키텍처 방법 (예: x 64의 SIPI) 뿐만 아니라 [Hvcallstartvirtualprocessor](hypercalls/HvCallStartVirtualProcessor.md) hypercall가 포함 됩니다.

#### <a name="interceptvpstartup"></a>InterceptVpStartup

InterceptVpStartup 플래그가 설정 된 경우 가상 프로세서를 시작 하거나 다시 설정 하면 더 높은 VTL로 가로채기를 생성 합니다.

### <a name="configuring-lower-vtls"></a>하위 VTLs 구성

더 높은 VTLs에서 다음 레지스터를 사용 하 여 더 낮은 VTLs의 동작을 구성할 수 있습니다.

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 MbecEnabled : 1;
        UINT64 TlbLocked : 1;
        UINT64 ReservedZ : 62;
    };
} HV_REGISTER_VSM_VP_SECURE_VTL_CONFIG;
```

각 VTL (0 보다 큼)에는 모든 VTL에 대 한이 레지스터의 인스턴스가 포함 되어 있습니다. 예를 들어 VTL2에는 VTL1에 대 한 레지스터와 VTL0에 대 한 두 개의 인스턴스가 있습니다.

이 레지스터의 필드는 아래에 설명 되어 있습니다.

#### <a name="mbecenabled"></a>MbecEnabled

이 필드는 하위 VTL에 대해 MBEC을 사용할 수 있는지 여부를 구성 합니다.

#### <a name="tlblocked"></a>TlbLocked 됨

이 필드는 더 낮은 VTL의 TLB를 잠급니다. 이 기능을 사용 하 여 더 낮은 VTL을 방해할 수 있는 TLB 무효화가 발생 하지 않도록 할 수 있습니다. 이 비트가 설정 되 면 낮은 VTL의 모든 주소 공간 플러시 요청은 잠금이 리프트 될 때까지 차단 됩니다.

TLB의 잠금을 해제 하기 위해 더 높은 VTL는이 비트를 지울 수 있습니다. 또한 VP가 더 낮은 VTL로 반환 되 면 해당 시점에 보유 하는 모든 TLB 잠금을 해제 합니다.

## <a name="vtl-entry"></a>VTL 항목

VTL가 낮은 VTL에서 더 높은 수준으로 전환 되 면 "입력" 됩니다. 이 문제는 다음과 같은 이유로 발생할 수 있습니다.

1. VTL call: 소프트웨어에서 더 높은 VTL의 코드를 명시적으로 호출 하려고 하는 경우입니다.
2. 보안 인터럽트: 더 높은 VTL에 대해 인터럽트를 수신 하는 경우 VP가 더 높은 VTL를 입력 합니다.
3. 보안 가로채기: 특정 작업은 보안 인터럽트 (예: 특정 MSRs에 액세스)를 트리거합니다.

VTL를 입력 한 후에는 자발적으로 종료 해야 합니다. 더 높은 VTL는 더 낮은 VTL으로 선점할 수 없습니다.

### <a name="identifying-vtl-entry-reason"></a>VTL 항목 확인 이유

항목에 적절 하 게 반응 하기 위해 더 높은 VTL는 입력 된 이유를 알아야 할 수 있습니다. 항목의 이유를 파악 하기 위해 VTL 항목은 [HV_VP_VTL_CONTROL](datatypes/HV_VP_VTL_CONTROL.md) 구조에 포함 됩니다.

### <a name="vtl-call"></a>VTL 호출

"VTL call"은 [Hvcallvtlcall](hypercalls/HvCallVtlCall.md) hypercall를 통해 더 낮은 VTL가 더 높은 VTL (예: 더 높은 VTL의 메모리 영역을 보호 하는 경우)에 대 한 항목을 시작 하는 경우입니다.

VTL 호출은 VTL 스위치를 통해 공유 레지스터의 상태를 유지 합니다. 개인 레지스터는 VTL 수준에서 유지 됩니다. 이러한 제한에 대 한 예외는 VTL 호출 시퀀스에 필요한 레지스터입니다. VTL 호출에는 다음 레지스터가 필요 합니다.

| X64     | x86     | Description                                                 |
|---------|---------|-------------------------------------------------------------|
| RCX     | EDX: EAX | 하이퍼바이저에 대 한 VTL 호출 제어 입력을 지정 합니다.        |
| RAX     | ECX     | 예약됨                                                    |

VTL call 컨트롤 입력의 모든 비트가 현재 예약 되어 있습니다.

#### <a name="vtl-call-restrictions"></a>VTL 호출 제한

VTL 호출은 가장 권한 있는 프로세서 모드 에서만 시작할 수 있습니다. 예를 들어 x64 시스템에서 VTL 호출은 CPL0 에서만 가져올 수 있습니다. 프로세서 모드에서 시작 된 VTL 호출은 모든 항목 이지만 시스템에서 가장 많은 권한이 있는 호출은 하이퍼바이저가 가상 프로세서에 #UD 예외를 주입 합니다.

VTL 호출은 다음으로 가장 높은 VTL로만 전환할 수 있습니다. 즉, 여러 VTLs를 사용 하도록 설정 된 경우에는 호출에서 VTL를 "skip" 할 수 없습니다.
다음 작업을 수행 하면 #UD 예외가 발생 합니다.

- 프로세서 모드에서 시작 되는 VTL 호출은 시스템에서 가장 높은 권한 (아키텍처 관련)입니다.
- 실제 모드에서 VTL 호출 (x86/x64)
- 대상 VTL 사용 하지 않도록 설정 되었거나 아직 사용 하도록 설정 되지 않은 가상 프로세서의 VTL 호출입니다.
- 잘못 된 컨트롤 입력 값을 포함 하는 VTL 호출

## <a name="vtl-exit"></a>VTL 끝내기

낮은 VTL로 전환 하는 것을 "return" 이라고 합니다. VTL 처리가 완료 되 면 VTL 반환을 시작 하 여 하위 VTL로 전환할 수 있습니다. VTL 반환은 상위 VTL 자발적으로 하나를 시작 하는 경우에만 발생할 수 있습니다. 더 낮은 VTL는 더 높은 항목을 선점할 수 없습니다.

### <a name="vtl-return"></a>VTL 반환

"VTL return"은 VTL가 [Hvcallvtlreturn](hypercalls/HvCallVtlReturn.md) hypercall를 통해 더 낮은 VTL으로 스위치를 시작 하는 경우입니다. VTL 호출과 마찬가지로 전용 프로세서 상태는 전환 되며 공유 상태는 그대로 유지 됩니다. 낮은 VTL가 더 높은 VTL로 명시적으로 호출 된 경우 반환이 완료 되기 전에 하이퍼바이저는 더 높은 VTL의 명령 포인터를 증가 시켜 VTL 호출 후에도 계속 됩니다.

VTL 반환 코드 시퀀스를 사용 하려면 다음 레지스터를 사용 해야 합니다.

| X64     | x86     | Description                                                 |
|---------|---------|-------------------------------------------------------------|
| RCX     | EDX: EAX | 하이퍼바이저에 대 한 VTL 반환 컨트롤 입력을 지정 합니다.      |
| RAX     | ECX     | 예약됨                                                    |

VTL 반환 컨트롤 입력의 형식은 다음과 같습니다.

| 비트      | 필드           | Description                                                                 |
|-----------|-----------------|-----------------------------------------------------------------------------|
| 63:1      | RsvdZ           |                                                                             |
| 0         | 빠른 반환     | 레지스터가 복원 되지 않습니다.                                                  |

다음 작업을 수행 하면 #UD 예외가 생성 됩니다.

- 가장 낮은 VTL가 현재 활성 상태인 경우 VTL을 반환 하려고 합니다.
- 잘못 된 컨트롤 입력 값을 사용 하 여 VTL을 반환 하려고 합니다.
- VTL을 시도 하면 시스템에서 가장 많은 권한이 있는 프로세서 모드에서 반환 됩니다 (아키텍처 관련).

#### <a name="fast-return"></a>빠른 반환

반환을 처리 하는 과정에서 하이퍼바이저는 [HV_VP_VTL_CONTROL](datatypes/HV_VP_VTL_CONTROL.md) 구조에서 낮은 VTL의 레지스터 상태를 복원할 수 있습니다. 예를 들어 보안 인터럽트를 처리 한 후 더 높은 VTL는 하위 VTL의 상태를 방해 하지 않고 반환 하려고 할 수 있습니다. 따라서 하이퍼바이저는 낮은 VTL의 레지스터를 VTL 컨트롤 구조에 저장 된 사전 호출 값으로 복원 하는 메커니즘을 제공 합니다.

이 동작이 필요 하지 않은 경우 더 높은 VTL는 "빠른 반환"을 사용할 수 있습니다. 빠른 반환은 하이퍼바이저가 컨트롤 구조에서 등록 상태를 복원 하지 않는 경우입니다. 불필요 한 처리를 방지 하려면 가능한 경우 항상 사용 해야 합니다.

이 필드는 VTL 반환 입력의 비트 0을 사용 하 여 설정할 수 있습니다. 0으로 설정 되 면 레지스터는 HV_VP_VTL_CONTROL 구조에서 복원 됩니다. 이 비트가 1로 설정 된 경우 레지스터는 복원 되지 않습니다 (빠른 반환).

## <a name="hypercall-page-assist"></a>Hypercall 페이지 지원

하이퍼바이저는 VTL 호출을 지원 하 고 [hypercall 페이지](hypercall-interface.md#establishing-the-hypercall-interface)를 통해 반환 하는 메커니즘을 제공 합니다. 이 페이지에서는 VTLs를 전환 하는 데 필요한 특정 코드 시퀀스를 요약 합니다.

VTL 호출을 실행 하 고 반환 되는 코드 시퀀스는 hypercall 페이지의 특정 지침을 실행 하 여 액세스할 수 있습니다. 호출/반환 청크는 HvRegisterVsmCodePageOffset 가상 레지스터에 의해 결정 되는 hypercall 페이지의 오프셋에 있습니다. 이는 읽기 전용 이며 파티션 전체 레지스터 이며 별도의 인스턴스를 VTL 합니다.

VTL는 CALL 명령을 사용 하 여 VTL call/return을 실행할 수 있습니다. Hypercall 페이지에서 올바른 위치를 호출 하면 VTL 호출/반환을 시작 합니다.

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 VtlCallOffset : 12;
        UINT64 VtlReturnOffset : 12;
        UINT64 ReservedZ : 40;
    };
} HV_REGISTER_VSM_CODE_PAGE_OFFSETS;
```

요약 하자면, hypercall 페이지를 사용 하 여 코드 시퀀스를 호출 하는 단계는 다음과 같습니다.

1. Hypercall 페이지를 VTL의 GPA 공간에 매핑
2. 코드 시퀀스의 올바른 오프셋을 확인 합니다 (VTL 호출 또는 반환).
3. 호출을 사용 하 여 코드 시퀀스를 실행 합니다.

## <a name="memory-access-protections"></a>메모리 액세스 보호

VSM에서 제공 하는 필수 보호 중 하나는 메모리 액세스를 격리 하는 기능입니다.

더 높은 VTLs는 더 낮은 VTLs에서 허용 하는 메모리 액세스 유형에 대해 높은 수준의 제어를 가집니다. 특정 GPA 페이지 (읽기, 쓰기 및 실행)에 대해 더 높은 VTL에서 지정할 수 있는 기본 보호 유형은 세 가지입니다. 이러한 정의는 다음 표에 정의 되어 있습니다.

| Name                       | Description                                                         |
|----------------------------|---------------------------------------------------------------------|
| 읽기                       | 메모리 페이지에 대 한 읽기 권한이 허용 되는지 여부를 제어 합니다.            |
| 쓰기                      | 메모리 페이지에 대해 쓰기 액세스를 허용 하는지 여부를 제어 합니다.              |
| Execute                    | 메모리 페이지에 대해 명령 반입이 허용 되는지 여부를 제어 합니다. |

이러한 세 가지 유형의 메모리 보호는 다음과 같습니다.

1. 액세스 권한 없음
2. 읽기 전용, 실행 안 함
3. 읽기 전용, 실행
4. 읽기/쓰기, 실행 안 함
5. 읽기/쓰기, 실행

"Mode based execution control (MBEC))"가 enabed 인 경우 사용자 및 커널 모드 실행 보호를 별도로 설정할 수 있습니다.

더 높은 VTLs는 [HvCallModifyVtlProtectionMask](hypercalls/HvCallModifyVtlProtectionMask.md) hypercall를 통해 GPA에 대 한 메모리 보호를 설정할 수 있습니다.

### <a name="memory-protection-hierarchy"></a>메모리 보호 계층 구조

메모리 액세스 권한은 특정 VTL의 여러 원본으로 설정할 수 있습니다. 각 VTL의 사용 권한은 호스트 파티션 뿐만 아니라 다른 여러 VTLs에 의해 제한 될 수 있습니다. 보호를 적용 하는 순서는 다음과 같습니다.

1. 호스트에서 설정 하는 메모리 보호
2. 더 높은 VTLs로 설정 된 메모리 보호

즉, VTL 보호는 호스트 보호를 대체 합니다. 상위 VTLs는 하위 수준 VTLs를 대체 합니다. VTL는 자체에 대 한 메모리 액세스 권한을 설정 하지 않을 수 있습니다.

호환 되는 인터페이스는 RAM이 아닌 RAM 형식을 오버레이 하지 않아야 합니다.

### <a name="memory-access-violations"></a>메모리 액세스 위반

더 낮은 VTL에서 실행 되는 VP가 상위 VTL에서 설정한 메모리 보호를 위반 하려고 시도 하면 가로채기가 생성 됩니다. 이 가로채기는 보호를 설정 하는 더 높은 VTL에서 수신 합니다. 이렇게 하면 더 높은 VTLs가 사례별로 위반을 처리할 수 있습니다. 예를 들어 더 높은 VTL는 오류를 반환 하거나 액세스를 에뮬레이트할 수 있습니다.

### <a name="mode-based-execute-control-mbec"></a>모드 기반 실행 제어 (MBEC)

VTL가 낮은 VTL에 메모리 제한을 배치 하는 경우 "실행" 권한을 부여할 때 사용자와 커널 모드를 구분 해야 할 수 있습니다. 예를 들어 더 높은 VTL에서 코드 무결성 검사가 수행 되는 경우 사용자 모드와 커널 모드를 구분 하는 기능은 VTL 응용 프로그램에 대해서만 코드 무결성을 적용할 수 있다는 의미입니다.

3 개의 메모리 보호 (읽기, 쓰기, 실행)와는 별도로 MBEC는 실행 보호를 위한 사용자 모드 및 커널 모드의 차이를 소개 합니다. 따라서 MBEC를 사용 하는 경우 VTL는 4 가지 유형의 메모리 보호를 설정할 수 있습니다.

| Name                       | Description                                                         |
|----------------------------|---------------------------------------------------------------------|
| 읽기                       | 메모리 페이지에 대 한 읽기 권한이 허용 되는지 여부를 제어 합니다.            |
| 쓰기                      | 메모리 페이지에 대해 쓰기 액세스를 허용 하는지 여부를 제어 합니다.              |
| 사용자 모드 실행 (UMX)    | 사용자 모드에서 생성 된 명령 페치를 메모리 페이지에 사용할 수 있는지 여부를 제어 합니다. 참고: MBEC가 사용 하지 않도록 설정 된 경우이 설정은 무시 됩니다. |
| 커널 모드 실행 (UMX)  | 커널 모드에서 생성 된 명령 페치를 메모리 페이지에 사용할 수 있는지 여부를 제어 합니다. 참고: MBEC를 사용 하지 않도록 설정 하면이 설정은 사용자 모드 및 커널 모드 실행 액세스를 모두 제어 합니다. |

"사용자 모드 실행" 보호로 표시 된 메모리는 가상 프로세서가 사용자 모드에서 실행 되는 경우에만 실행 가능 합니다. 마찬가지로, "커널 모드 실행" 메모리는 가상 프로세서가 커널 모드에서 실행 되는 경우에만 실행 가능 합니다.

KMX 및 UMX는 개별적으로 설정 하 여 사용자와 커널 모드 간에 실행 권한이 다르게 적용 되도록 할 수 있습니다. KMX = 1, UMX = 0을 제외 하 고 UMX 및 KMX의 모든 조합이 지원 됩니다. 이 조합의 동작은 정의 되지 않습니다.

MBEC은 모든 VTLs 및 가상 프로세서에 대해 기본적으로 사용 하지 않도록 설정 되어 있습니다. MBEC를 사용 하지 않도록 설정 하면 커널 모드 실행 비트에서 메모리 액세스 제한을 결정 합니다. 따라서 MBEC를 사용 하지 않도록 설정 하는 경우 KMX = 1 코드는 커널 및 사용자 모드에서 실행 가능 합니다.

#### <a name="descriptor-tables"></a>설명자 테이블

설명자 테이블에 액세스 하는 모든 사용자 모드 코드는 KMX = UMX = 1로 표시 된 GPA 페이지에 있어야 합니다. KMX = 0으로 표시 된 GPA 페이지에서 설명자 테이블에 액세스 하는 사용자 모드 소프트웨어는 지원 되지 않으며 일반적인 보호 오류가 발생 합니다.

#### <a name="mbec-configuration"></a>MBEC 구성

모드 기반 실행 제어를 사용 하려면 다음 두 수준에서 사용 하도록 설정 해야 합니다.

1. 파티션에 대해 VTL를 사용 하도록 설정한 경우 HvCallEnablePartitionVtl를 사용 하 여 MBEC를 사용 하도록 설정 해야 합니다.
2. MBEC는 HvRegisterVsmVpSecureVtlConfig을 사용 하 여 VP 및 VTL 기준으로 구성 해야 합니다.

#### <a name="mbec-interaction-with-supervisor-mode-execution-prevention-smep"></a>MBEC SMEP (감독자 모드 실행 방지)와의 상호 작용

SMEP (Supervisor-Mode 실행 방지)는 일부 플랫폼에서 지원 되는 프로세서 기능입니다. SMEP는 메모리 페이지에 대 한 감독자 액세스 제한으로 인해 MBEC 작업에 영향을 줄 수 있습니다. 하이퍼바이저는 SMEP와 관련 된 다음 정책을 준수 합니다.

- SMEP를 게스트 OS에서 사용할 수 없는 경우 (하드웨어 기능이 나 프로세서 호환성 모드 때문 인지 여부에 관계 없이) MBEC는 영향을 받지 않습니다.
- SMEP를 사용할 수 있고 사용 하도록 설정 된 경우 MBEC는 영향을 받지 않습니다.
- SMEP를 사용할 수 있고 사용 하지 않도록 설정 된 경우 모든 실행 제한은 KMX 컨트롤에 의해 제어 됩니다. 따라서 KMX = 1로 표시 된 코드만 실행할 수 있습니다.

## <a name="virtual-processor-state-isolation"></a>가상 프로세서 상태 격리

가상 프로세서는 각 활성 VTL 별도의 상태를 유지 합니다. 그러나이 상태 중 일부는 특정 VTL 전용 이며 나머지 상태는 모든 VTLs 간에 공유 됩니다.

VTL 당 유지 되는 상태 ( 개인 상태)가 VTL 전환 간에 하이퍼바이저에 의해 저장 됩니다. VTL 스위치가 시작 되 면 하이퍼바이저는 활성 VTL의 현재 개인 상태를 저장 한 다음 대상 VTL의 개인 상태로 전환 합니다. 공유 상태는 VTL 스위치에 관계 없이 활성 상태로 유지 됩니다.

### <a name="private-state"></a>개인 상태

일반적으로 각 VTL에는 고유한 컨트롤 레지스터, RIP register, RSP register 및 MSRs가 있습니다. 다음은 각 VTL에 전용으로 적용 되는 특정 레지스터 및 MSRs의 목록입니다.

개인 MSRs:

- SYSENTER_CS, SYSENTER_ESP, SYSENTER_EIP, STAR, LSTAR, CSTAR, SFMASK, EFER, PAT, KERNEL_GSBASE, FS입니다. 기본, GS. 기본, TSC_AUX
- HV_X64_MSR_HYPERCALL
- HV_X64_MSR_GUEST_OS_ID
- HV_X64_MSR_REFERENCE_TSC
- HV_X64_MSR_APIC_FREQUENCY
- HV_X64_MSR_EOI
- HV_X64_MSR_ICR
- HV_X64_MSR_TPR
- HV_X64_MSR_APIC_ASSIST_PAGE
- HV_X64_MSR_NPIEP_CONFIG
- HV_X64_MSR_SIRBP
- HV_X64_MSR_SCONTROL
- HV_X64_MSR_SVERSION
- HV_X64_MSR_SIEFP
- HV_X64_MSR_SIMP
- HV_X64_MSR_EOM
- HV_X64_MSR_SINT0 – HV_X64_MSR_SINT15
- HV_X64_MSR_STIMER0_CONFIG – HV_X64_MSR_STIMER3_CONFIG
- HV_X64_MSR_STIMER0_COUNT – HV_X64_MSR_STIMER3_COUNT
- 로컬 APIC 레지스터 (CR8/TPR 포함)

개인 레지스터:

- 립, RSP
- RFLAGS
- CR0 레지스터, CR3, CR4
- DR7
- IDTR, GDTR
- CS, DS, ES, FS, GS, SS, TR, LDTR
- TSC
- DR6 (* 프로세서 유형에 따라 달라 집니다. HvRegisterVsmCapabilities 가상 레지스터를 읽고 공유/개인 상태를 확인 합니다.

### <a name="shared-state"></a>공유 상태

VTLs는 컨텍스트 전환의 오버 헤드를 줄이기 위해 상태를 공유 합니다. 또한 상태를 공유 하면 VTLs 간에 필요한 몇 가지 통신이 가능 합니다. 대부분의 일반적인 용도와 부동 소수점 레지스터는 대부분의 아키텍처 MSRs와 마찬가지로 공유 됩니다. 다음은 모든 VTLs에서 공유 되는 특정 MSRs 및 레지스터의 목록입니다.

공유 MSRs:

- HV_X64_MSR_TSC_FREQUENCY
- HV_X64_MSR_VP_INDEX
- HV_X64_MSR_VP_RUNTIME
- HV_X64_MSR_RESET
- HV_X64_MSR_TIME_REF_COUNT
- HV_X64_MSR_GUEST_IDLE
- HV_X64_MSR_DEBUG_DEVICE_OPTIONS
- MTRRs
- MCG_CAP
- MCG_STATUS

공유 레지스터:

- Rax, Rbx, Rcx, Rdx, Rsi, Rsi, Rsi
- CR2
- R8 – R15
- DR0 – DR5
- X87 부동 소수점 상태
- XMM 상태
- AVX 상태
- XCR0 (XFEM)
- DR6 (* 프로세서 유형에 따라 달라 집니다. HvRegisterVsmCapabilities 가상 레지스터를 읽고 공유/개인 상태를 확인 합니다.

### <a name="real-mode"></a>실제 모드

실제 모드는 0 보다 큰 VTL 지원 되지 않습니다. VTLs를 0 보다 크게 32 비트 또는 64 비트 모드로 실행할 수 있습니다.

## <a name="vtl-interrupt-management"></a>VTL 인터럽트 관리

가상 신뢰 수준 간에 높은 수준의 격리를 위해 가상 보안 모드는 가상 프로세서에서 사용 하도록 설정 된 각 VTL에 대 한 별도의 인터럽트 하위 시스템을 제공 합니다. 이렇게 하면 VTL는 보안 수준이 낮은 VTL의 간섭 없이 인터럽트를 보내고 받을 수 있습니다.

각 VTL에는 자체 인터럽트 컨트롤러가 있습니다 .이 컨트롤러는 해당 특정 VTL에서 가상 프로세서를 실행 하는 경우에만 활성화 됩니다. 가상 프로세서가 VTL 상태를 전환 하는 경우 프로세서에서 활성화 된 인터럽트 컨트롤러도 전환 됩니다.

활성 VTL 보다 높은 VTL를 대상으로 하는 인터럽트는 즉각적인 VTL 스위치를 발생 시킵니다. 그런 다음 더 높은 VTL는 인터럽트를 받을 수 있습니다. TPR/CR8 값 때문에 더 높은 VTL이 인터럽트를 받을 수 없는 경우 인터럽트가 "보류 중"으로 유지 되 고 VTL가 전환 되지 않습니다. 보류 중인 인터럽트가 있는 VTLs가 여러 개 있는 경우 가장 높은 VTL가 우선 적용 됩니다 (VTL에 대 한 통지 없이).

인터럽트가 더 낮은 VTL를 대상으로 하는 경우 다음 번에 가상 프로세서가 대상 VTL로 전환 될 때까지 인터럽트가 배달 되지 않습니다. 더 낮은 VTL를 대상으로 하는 초기화 및 시작 IPIs는 더 낮은 VTL를 사용 하는 가상 프로세서에서 삭제 됩니다. INIT/SIPI가 차단 되므로 [Hvcallstartvirtualprocessor](hypercalls/HvCallStartVirtualProcessor.md) hypercall를 사용 하 여 프로세서를 시작 해야 합니다.

### <a name="rflagsif"></a>RFLAGS. 때

VTLs를 전환 하려면 RFLAGS를 사용 합니다. 에서 보안 인터럽트가 VTL 스위치를 트리거 하는지 여부에 영향을 주지 않는 경우 RFLAGS 인 경우 가 인터럽트를 마스킹 하도록 선택 되지 않은 경우 더 높은 VTLs로의 인터럽트가 여전히 더 높은 VTL로 VTL 전환 됩니다. 즉시 중단 여부를 결정할 때 상위 VTL의 TPR/CR8 값만 고려 됩니다.

이 동작은 VTL 반환 시 보류 중인 인터럽트에도 영향을 줍니다. RFLAGS 인 경우입니다. 지정 된 VTL에서 인터럽트를 마스킹 하도록 비트를 지우면 VTL가 (더 낮은 VTL로) 반환 되 면 하이퍼바이저는 보류 중인 인터럽트를 다시 계산 합니다. 이렇게 하면 더 높은 VTL 다시 호출 됩니다.

### <a name="virtual-interrupt-notification-assist"></a>가상 인터럽트 알림 지원

더 높은 VTLs는 동일한 가상 프로세서의 더 낮은 VTL로 인터럽트를 즉시 전달 하는 것을 차단 하는 경우 알림을 받도록 등록할 수 있습니다. 더 높은 VTLs는 가상 레지스터 HvRegisterVsmVina를 통해 VINA (가상 인터럽트 알림 지원)를 사용 하도록 설정할 수 있습니다.

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Vector : 8;
        UINT64 Enabled : 1;
        UINT64 AutoReset : 1;
        UINT64 AutoEoi : 1;
        UINT64 ReservedP : 53;
    };
} HV_REGISTER_VSM_VINA;
```

각 VP의 각 VTL에는 자체 VINA 인스턴스와 자체의 HvRegisterVsmVina 버전이 있습니다. VINA 기능은 하위 VTL에 대 한 인터럽트가 즉시 배달 될 준비가 되 면 현재 활성화 된 상위 VTL에 대 한 edge 트리거된 인터럽트를 생성 합니다.

이 기능을 사용할 때 발생 하는 인터럽트가 발생 하지 않도록 하기 위해 VINA 기능에는 몇 가지 제한 된 상태가 포함 됩니다. VINA 인터럽트가 생성 되 면 VINA 시설의 상태가 "어설션된"로 변경 됩니다. VINA 기능에 연결 된 대상으로 인터럽트 종료를 보내면 "어설션된" 상태가 지워지지 않습니다. 다음 두 가지 방법 중 하나로 어설션된 상태를 지울 수 있습니다.

1. [HV_VP_VTL_CONTROL](datatypes/HV_VP_VTL_CONTROL.md) 구조의 VinaAsserted 필드에 작성 하 여 상태를 수동으로 지울 수 있습니다.
2. HvRegisterVsmVina register에서 "VTL entry에서 자동 다시 설정" 옵션을 사용 하는 경우 VTL에 대 한 다음 항목에서 상태가 자동으로 지워집니다.

이를 통해 보안 VTL에서 실행 되는 코드가 낮은 VTL에 대해 받은 첫 번째 인터럽트에 대 한 알림만 받을 수 있습니다. 보안 VTL에서 추가 인터럽트를 통보 받으려면 VP 지원 페이지의 VinaAsserted 필드를 지우면 다음 새 인터럽트가 통보 됩니다.

## <a name="secure-intercepts"></a>보안 가로채기

하이퍼바이저를 사용 하면 더 높은 VTL 더 낮은 VTL 컨텍스트에서 발생 하는 이벤트에 대 한 가로채기를 설치할 수 있습니다. 이렇게 하면 더 높은 VTLs가 VTL 리소스에 대해 높은 수준의 제어를 제공 합니다. 보안 가로채기는 시스템에 중요 한 리소스를 보호 하 고, 더 낮은 VTLs의 공격을 방지 하는 데 사용할 수 있습니다.

보안 가로채기는 더 높은 VTL 대기 하 고 해당 VTL은 VP에서 실행 가능 합니다.

### <a name="secure-intercept-types"></a>보안 가로채기 형식

| 가로채기 형식             | 가로채기 적용 대상                                                |
|----------------------------|---------------------------------------------------------------------|
| 메모리 액세스              | 상위 VTL에 의해 설정 된 GPA 보호에 액세스 하려고 합니다.   |
| 등록 액세스 제어    | 더 높은 VTL으로 지정 된 컨트롤 레지스터 집합에 액세스 하려고 합니다. |

### <a name="nested-intercepts"></a>중첩 된 가로채기

여러 VTLs는 더 낮은 VTL 동일한 이벤트에 대 한 보안 가로채기를 설치할 수 있습니다. 따라서 중첩 된 가로채기의 수신 위치를 결정 하기 위해 계층이 설정 됩니다. 다음 목록에는 가로채기를 알리는 순서가 나와 있습니다.

1. 낮은 VTL
2. 높은 VTL

### <a name="handling-secure-intercepts"></a>보안 가로채기 처리

VTL 보안 가로채기에 대 한 통지를 받은 후에는 하위 VTL을 계속 사용할 수 있도록 조치를 취해야 합니다.
높은 VTL는 예외를 삽입 하거나 액세스를 에뮬레이트 하거나 액세스에 프록시를 제공 하는 등 여러 가지 방법으로 가로채기를 처리할 수 있습니다. 어떤 경우 든 낮은 VTL VP의 개인 상태를 수정 해야 하는 경우에는 [HvCallSetVpRegisters](hypercalls/HvCallSetVpRegisters.md) 를 사용 해야 합니다.

### <a name="secure-register-intercepts"></a>보안 등록 가로채기

상위 VTL는 특정 컨트롤 레지스터에 대 한 액세스를 가로챌 수 있습니다. 이렇게 하려면 [HvCallSetVpRegisters](hypercalls/HvCallSetVpRegisters.md) hypercall를 사용 하 여 HvX64RegisterCrInterceptControl를 설정 합니다. HvX64RegisterCrInterceptControl에서 컨트롤 비트를 설정 하면 해당 컨트롤 레지스터의 모든 액세스에 대해 가로채기가 트리거됩니다.

```c
typedef union
{
    UINT64 AsUINT64;
    struct
    {
        UINT64 Cr0Write : 1;
        UINT64 Cr4Write : 1;
        UINT64 XCr0Write : 1;
        UINT64 IA32MiscEnableRead : 1;
        UINT64 IA32MiscEnableWrite : 1;
        UINT64 MsrLstarRead : 1;
        UINT64 MsrLstarWrite : 1;
        UINT64 MsrStarRead : 1;
        UINT64 MsrStarWrite : 1;
        UINT64 MsrCstarRead : 1;
        UINT64 MsrCstarWrite : 1;
        UINT64 ApicBaseMsrRead : 1;
        UINT64 ApicBaseMsrWrite : 1;
        UINT64 MsrEferRead : 1;
        UINT64 MsrEferWrite : 1;
        UINT64 GdtrWrite : 1;
        UINT64 IdtrWrite : 1;
        UINT64 LdtrWrite : 1;
        UINT64 TrWrite : 1;
        UINT64 MsrSysenterCsWrite : 1;
        UINT64 MsrSysenterEipWrite : 1;
        UINT64 MsrSysenterEspWrite : 1;
        UINT64 MsrSfmaskWrite : 1;
        UINT64 MsrTscAuxWrite : 1;
        UINT64 MsrSgxLaunchControlWrite : 1;
        UINT64 RsvdZ : 39;
    };
} HV_REGISTER_CR_INTERCEPT_CONTROL;
```

#### <a name="mask-registers"></a>레지스터 마스크

더 세밀 하 게 제어할 수 있도록 컨트롤 레지스터의 하위 집합에도 해당 하는 마스크 레지스터가 있습니다. Mask 레지스터를 사용 하 여 해당 컨트롤 레지스터의 하위 집합에 가로채기를 설치할 수 있습니다. Mask 레지스터가 정의 되지 않은 경우 HvX64RegisterCrInterceptControl에 의해 정의 된 모든 액세스는 가로채기를 트리거합니다.

하이퍼바이저는 HvX64RegisterCrInterceptCr0Mask, HvX64RegisterCrInterceptCr4Mask 및 HvX64RegisterCrInterceptIa32MiscEnableMask와 같은 마스크 레지스터를 지원 합니다.

## <a name="dma-and-devices"></a>DMA 및 장치

장치는 효과적으로 VTL0와 동일한 권한 수준을 갖습니다. VSM을 사용 하도록 설정 하면 모든 장치 할당 메모리가 VTL0로 표시 됩니다. 모든 DMA 액세스에는 VTL0와 동일한 권한이 있습니다.
