---
title: 기능 및 인터페이스 검색
description: 하이퍼바이저 기능 및 인터페이스 검색
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: b88cb0be23f4e3ea4ba1007d48bd11fe48749f00
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120907"
---
# <a name="feature-and-interface-discovery"></a>기능 및 인터페이스 검색

게스트 소프트웨어는 다양 한 메커니즘을 통해 하이퍼바이저와 상호 작용 합니다. 이러한 미러의 대부분은 소프트웨어에서 기본 프로세서와 상호 작용 하는 데 사용 하는 기존 메커니즘을 사용 합니다. 이러한 메커니즘은 아키텍처에 따라 다릅니다. X64 아키텍처에는 다음과 같은 메커니즘이 사용 됩니다.

- CPUID 명령 – 정적 기능 및 버전 정보에 사용 됩니다.
- MSRs (모델 관련 레지스터)-상태 및 제어 값에 사용 됩니다.
- 메모리 매핑된 레지스터 – 상태 및 컨트롤 값에 사용 됩니다.
- 프로세서 인터럽트 – 비동기 이벤트, 알림 및 메시지에 사용 됩니다.

이러한 아키텍처 관련 인터페이스 외에도 하이퍼바이저는 [hypercall](hypercall-interface.md)를 사용 하 여 구현 되는 간단한 절차적 인터페이스를 제공 합니다.

## <a name="hypervisor-discovery"></a>하이퍼바이저 검색

하이퍼바이저 인터페이스를 사용 하기 전에 먼저 소프트웨어에서 가상화 된 환경 내에서 실행 되 고 있는지 확인 해야 합니다. 이 사양에 맞는 x64 플랫폼에서 입력 (EAX) 값이 1 인 CPUID 명령을 실행 하 여이 작업을 수행 합니다. 실행 시 코드는 레지스터 ECX의 비트 31 ("하이퍼바이저 present 비트")을 확인 해야 합니다. 이 비트가 설정 되어 있는 경우 하이퍼바이저가 있습니다. 가상화 되지 않은 환경에서는 비트가 명확 하 게 됩니다.

 ```c
CPUID.01h.ECX:31 // if set, virtualization present
 ```

"하이퍼바이저 존재 비트"가 설정 된 경우 준수 하는 하이퍼바이저 및 해당 기능에 대 한 자세한 내용은 추가 CPUID leafs를 쿼리할 수 있습니다. 두 개의 리프를 사용할 수 있도록 보장 됩니다 `0x40000000` . `0x40000001` 이후에 번호가 매겨진 리프도 사용할 수 있습니다.

## <a name="standard-hypervisor-cpuid-leaves"></a>표준 하이퍼바이저 CPUID 리프

에서 리프를 `0x40000000` 쿼리하면 하이퍼바이저는 최대 하이퍼바이저 CPUID 리프 번호와 공급 업체 ID 서명을 제공 하는 정보를 반환 합니다.

| 등록  | 제공된 정보                                        |
|-----------|-------------------------------------------------------------|
| EAX       | 하이퍼바이저 CPUID 정보에 대 한 최대 입력 값    |
| EBX       | 하이퍼바이저 공급 업체 ID 서명                              |
| ECX       | 하이퍼바이저 공급 업체 ID 서명                              |
| EDX       | 하이퍼바이저 공급 업체 ID 서명                              |

에서 리프를 `0x40000001` 쿼리하면 공급 업체 중립 하이퍼바이저 인터페이스 id를 나타내는 값이 반환 됩니다. 이는에서의 리프에 대 한 의미를 결정 합니다 `0x4000002` `0x400000FF` .

| 등록  | 제공된 정보                                        |
|-----------|-------------------------------------------------------------|
| EAX       | 하이퍼바이저 인터페이스 서명입니다.                             |
| EBX       | 예약됨                                                    |
| ECX       | 예약됨                                                    |
| EDX       | 예약됨                                                    |

이러한 두 개의 리프는 게스트에서 하이퍼바이저 공급 업체 ID와 인터페이스를 독립적으로 쿼리할 수 있도록 합니다. 공급 업체 ID는 정보 제공 및 진단 목적 으로만 제공 됩니다. 소프트웨어 전용 기본 호환성은 리프를 통해 보고 되는 인터페이스 서명에 따라 결정 하는 것이 좋습니다 `0x40000001` .

## <a name="microsoft-hypervisor-cpuid-leaves"></a>Microsoft 하이퍼바이저 CPUID 리프

Microsoft 하이퍼바이저 CPUID 인터페이스에 맞는 하이퍼바이저에서 `0x40000000` 및 `0x40000001` 리프 레지스터에는 다음 값이 포함 됩니다.

### <a name="hypervisor-cpuid-leaf-range---0x40000000"></a>하이퍼바이저 CPUID 리프 범위-0x40000000

EAX는 최대 하이퍼바이저 CPUID 리프를 결정 합니다. EBX-EDX에는 하이퍼바이저 공급 업체 ID 서명이 포함 되어 있습니다. 공급 업체 ID 서명은 보고 및 진단 용도로만 사용 해야 합니다.

| 등록  | 제공된 정보                                        |
|-----------|-------------------------------------------------------------|
| EAX       | 하이퍼바이저 CPUID 정보에 대 한 최대 입력 값입니다. Microsoft 하이퍼바이저에서이는 최소 `0x40000005` 입니다.  |
| EBX       | 0X7263699d-"Micr"                                           |
| ECX       | 0x666F736F-"osof"                                           |
| EDX       | 0x76482074-"t Hv"                                           |

### <a name="hypervisor-vendor-neutral-interface-identification---0x40000001"></a>하이퍼바이저 공급 업체-중립 인터페이스 식별-0x40000001

EAX에는 하이퍼바이저 인터페이스 id 서명이 포함 되어 있습니다. 이는에서의 리프에 대 한 의미를 결정 합니다 `0x40000002` `0x400000FF` .

| 등록  | 제공된 정보                                        |
|-----------|-------------------------------------------------------------|
| EAX       | 0x31237648-"Hv # 1"                                           |
| EBX       | 예약됨                                                    |
| ECX       | 예약됨                                                    |
| EDX       | 예약됨                                                    |

"Hv # 1" 인터페이스에 맞는 하이퍼바이저는 최소한 다음 리프를 제공 합니다.

### <a name="hypervisor-system-identity---0x40000002"></a>하이퍼바이저 시스템 Id-0x40000002

<!---
+-----------+-------+------------------------+
| Register  | Bits  | Information Provided   |
+===========+=======+========================+
| EAX       |       | Build Number           |
+-----------+-------+------------------------+
| EBX       | 31-16 | Major Version          |
+           +-------+------------------------+
|           | 15-0  | Minor Version          |
+-----------+-------+------------------------+
| ECX       |       | Service Pack           |
+-----------+-------+------------------------+
| EDX       | 31-24 | Service Branch         |
+           +-------+------------------------+
|           | 23-0  | Service Number         |
+-----------+-------+------------------------+
-->

<table>
    <thead>
        <tr>
            <th>등록</th>
            <th>비트</th>
            <th>제공된 정보</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>EAX</td>
            <td></td>
            <td>빌드 번호</td>
        </tr>
        <tr>
            <td rowspan="2">EBX</td>
            <td>31-16</td>
            <td>주 버전</td>
        </tr>
         <tr>
            <td>15-0</td>
            <td>부 버전</td>
        </tr>
    </tbody>
</table>

### <a name="hypervisor-feature-identification---0x40000003"></a>하이퍼바이저 기능 식별-0x40000003

EAX 및 EBX는 현재 파티션 권한을 기반으로 파티션에 사용할 수 있는 기능을 표시 합니다.

<!---
+-----------+-------+---------------------------------------------------------------------------------------+
| Register  | Bits  | Information Provided                                                                  |
+===========+=======+=======================================================================================+
| EAX       |       | Corresponds to bits 31-0 of HV_PARTITION_PRIVILEGE_MASK                               |
+-----------+-------+---------------------------------------------------------------------------------------+
| EBX       |       | Corresponds to bits 63-32 of HV_PARTITION_PRIVILEGE_MASK                              |
+-----------+-------+---------------------------------------------------------------------------------------+
| ECX       |       |                                                                                       |
+-----------+-------+---------------------------------------------------------------------------------------+
| EDX       | 0     | Deprecated (previously indicated availability of the MWAIT command).                  |
+           +-------+---------------------------------------------------------------------------------------+
|           | 1     | Guest debugging support is available                                                  |
+           +-------+---------------------------------------------------------------------------------------+
|           | 2     | Performance Monitor support is available                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 3     | Support for physical CPU dynamic partitioning events is available                     |
+           +-------+---------------------------------------------------------------------------------------+
|           | 4     | Support for passing hypercall input parameter block via XMM registers is available    |
+           +-------+---------------------------------------------------------------------------------------+
|           | 5     | Support for a virtual guest idle state is available                                   |
+           +-------+---------------------------------------------------------------------------------------+
|           | 6     | Support for hypervisor sleep state is available                                       |
+           +-------+---------------------------------------------------------------------------------------+
|           | 7     | Support for querying NUMA distances is available                                      |
+           +-------+---------------------------------------------------------------------------------------+
|           | 8     | Support for determining timer frequencies is available                                |
+           +-------+---------------------------------------------------------------------------------------+
|           | 9     | Support for injecting synthetic machine checks is available                           |
+           +-------+---------------------------------------------------------------------------------------+
|           | 10    | Support for guest crash MSRs is available                                             |
+           +-------+---------------------------------------------------------------------------------------+
|           | 11    | Support for debug MSRs is available                                                   |
+           +-------+---------------------------------------------------------------------------------------+
|           | 12    | Support for NPIEP is available                                                        |
+           +-------+---------------------------------------------------------------------------------------+
|           | 13    | DisableHypervisorAvailable                                                            |
+           +-------+---------------------------------------------------------------------------------------+
|           | 14    | ExtendedGvaRangesForFlushVirtualAddressListAvailable                                  |
+           +-------+---------------------------------------------------------------------------------------+
|           | 15    | Support for returning hypercall output via XMM registers is available                 |
+           +-------+---------------------------------------------------------------------------------------+
|           | 16    | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 17    | SintPollingModeAvailable                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 18    | HypercallMsrLockAvailable                                                             |
+           +-------+---------------------------------------------------------------------------------------+
|           | 19    | Use direct synthetic timers                                                           |
+           +-------+---------------------------------------------------------------------------------------+
|           | 31-20 | Reserved                                                                              |
-->

<table>
    <thead>
        <tr>
            <th>등록</th>
            <th>비트</th>
            <th>제공된 정보</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>EAX</td>
            <td></td>
            <td>HV_PARTITION_PRIVILEGE_MASK 비트 31-0에 해당 합니다.</td>
        </tr>
        <tr>
            <td>EBX</td>
            <td></td>
            <td>HV_PARTITION_PRIVILEGE_MASK 비트 63-32에 해당 합니다.</td>
        </tr>
        <tr>
            <td>ECX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td rowspan="27">EDX</td>
            <td>0</td>
            <td>사용 되지 않음 (이전에 MWAIT 명령의 사용 가능)</td>
        </tr>
        <tr>
            <td>1</td>
            <td>게스트 디버깅 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>2</td>
            <td>성능 모니터 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>3</td>
            <td>실제 CPU 동적 분할 이벤트에 대 한 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>4</td>
            <td>XMM 레지스터를 통해 hypercall 입력 매개 변수 블록을 전달 하는 기능을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>5</td>
            <td>가상 게스트 유휴 상태에 대 한 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>6</td>
            <td>하이퍼바이저 절전 모드에 대 한 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>7</td>
            <td>NUMA 거리 쿼리에 대 한 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>8</td>
            <td>타이머 빈도 결정에 대 한 지원이 제공 됩니다.</td>
        </tr>
        <tr>
            <td>9</td>
            <td>가상 컴퓨터 검사 삽입에 대 한 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>10</td>
            <td>게스트 크래시 MSRs에 대 한 지원이 제공 됩니다.</td>
        </tr>
        <tr>
            <td>11</td>
            <td>MSRs의 debug 지원 기능을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>12</td>
            <td>NPIEP에 대 한 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>13</td>
            <td>DisableHypervisorAvailable 가능</td>
        </tr>
        <tr>
            <td>14</td>
            <td>ExtendedGvaRangesForFlushVirtualAddressListAvailable</td>
        </tr>
        <tr>
            <td>15</td>
            <td>XMM 레지스터를 통한 hypercall 출력 반환에 대 한 지원을 사용할 수 있습니다.</td>
        </tr>
        <tr>
            <td>16</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>17</td>
            <td>SintPollingModeAvailable</td>
        </tr>
        <tr>
            <td>18</td>
            <td>HypercallMsrLockAvailable 가능</td>
        </tr>
        <tr>
            <td>19</td>
            <td>직접 가상 타이머 사용</td>
        </tr>
        <tr>
            <td>20</td>
            <td>VSM에 사용할 수 있는 PAT 레지스터 지원</td>
        </tr>
        <tr>
            <td>21</td>
            <td>VSM에 사용할 수 있는 bndcfgs 레지스터 지원</td>
        </tr>
        <tr>
            <td>22</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>23</td>
            <td>중지 되지 않은 가상 시간 사용 가능 타이머에 대 한 지원</td>
        </tr>
        <tr>
            <td>25-24</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>26</td>
            <td>Intel의 LBR (Last Branch Record) 기능이 지원 됨</td>
        </tr>
        <tr>
            <td>31-27</td>
            <td>예약됨</td>
        </tr>
    </tbody>
</table>

### <a name="implementation-recommendations---0x40000004"></a>구현 권장 사항-0x40000004

하이퍼바이저가 최적의 성능을 위해 OS에서 구현 하도록 권장 하는 동작을 나타냅니다.

<!---
+-----------+-------+---------------------------------------------------------------------------------------+
| Register  | Bits  | Information Provided                                                                  |
+===========+=======+=======================================================================================+
| EAX       | 0     | Recommend using hypercall for address space switches rather than MOV to CR3 instruction
+           +-------+---------------------------------------------------------------------------------------+
|           | 1     | Recommend using hypercall for local TLB flushes rather than INVLPG or MOV to CR3 instructions
+           +-------+---------------------------------------------------------------------------------------+
|           | 2     | Recommend using hypercall for remote TLB flushes rather than inter-processor interrupts
+           +-------+---------------------------------------------------------------------------------------+
|           | 3     | Recommend using MSRs for accessing APIC registers EOI, ICR and TPR rather than their memory-mapped counterparts
+           +-------+---------------------------------------------------------------------------------------+
|           | 4     | Recommend using the hypervisor-provided MSR to initiate a system RESET                |
+           +-------+---------------------------------------------------------------------------------------+
|           | 5     | Recommend using relaxed timing for this partition. If used, the VM should disable any watchdog timeouts that rely on the timely delivery of external interrupts.
+           +-------+---------------------------------------------------------------------------------------+
|           | 6     | Recommend using DMA remapping                                                         |
+           +-------+---------------------------------------------------------------------------------------+
|           | 7     | Recommend using interrupt remapping                                                   |
+           +-------+---------------------------------------------------------------------------------------+
|           | 8     | Recommend using x2APIC MSRs                                                           |
+           +-------+---------------------------------------------------------------------------------------+
|           | 9     | Recommend deprecating AutoEOI                                                         |
+           +-------+---------------------------------------------------------------------------------------+
|           | 10    | Recommend using SyntheticClusterIpi hypercall                                         |
+           +-------+---------------------------------------------------------------------------------------+
|           | 11    | Recommend using the newer ExProcessorMasks interface                                  |
+           +-------+---------------------------------------------------------------------------------------+
|           | 12    |  Indicates that the hypervisor is nested within a Hyper-V partition.                  |
+           +-------+---------------------------------------------------------------------------------------+
|           | 13    | Recommend using INT for MBEC system calls                                             |
+           +-------+---------------------------------------------------------------------------------------+
|           | 14    | Recommend a nested hypervisor using the enlightened VMCS interface. Also indicates that additional nested enlightenments may be available (see leaf `0x4000000A`)
+           +-------+---------------------------------------------------------------------------------------+
|           | 15    | UseSyncedTimeline – Indicates the partition should consume the QueryPerformanceCounter bias provided by the root partition.
+           +-------+---------------------------------------------------------------------------------------+
|           | 16    | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 17    | UseDirectLocalFlushEntire – Indicates the guest should toggle CR4.PGE to flush the entire TLB, as this is more performant than making a hypercall.
+           +-------+---------------------------------------------------------------------------------------+
|           | 18    | NoNonArchitecturalCoreSharing - Indicates that a virtual processor will never share a physical core with another virtual processor, except for virtual processors that are reported as sibling SMT threads. This can be used as an optimization to avoid the performance overhead of STIBP.
+           +-------+---------------------------------------------------------------------------------------+
|           | 31-19 | Reserved                                                                              |
+-----------+-------+---------------------------------------------------------------------------------------+
| EBX       |       | Recommended number of attempts to retry a spinlock failure before notifying the hypervisor about the failures. 0xFFFFFFFF indicates never to retry.
+-----------+-------+---------------------------------------------------------------------------------------+
| ECX       | 6-0   | ImplementedPhysicalAddressBits – Reports the physical address width (MAXPHYADDR) reported by the system’s physical processors. If all bits contain 0, the feature is not supported. Note that the value reported is the actual number of physical address bits, and not the bit position used to represent that number.
+           +-------+---------------------------------------------------------------------------------------+
|           | 31-7  | Reserved                                                                              |
+-----------+-------+---------------------------------------------------------------------------------------+
| EDX       |       |   Reserved                                                                            |
-->

<table>
    <thead>
        <tr>
            <th>등록</th>
            <th>비트</th>
            <th>제공된 정보</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="20">EAX</td>
            <td>0</td>
            <td>Hypercall 명령을 사용 하는 대신 주소 공간 스위치에 대해 MOV를 사용 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>1</td>
            <td>INVLPG 또는 MOV가 아닌 로컬 TLB 플러시에 hypercall를 사용 하 여 명령을 CR3 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>2</td>
            <td>프로세서 간 인터럽트가 아닌 원격 TLB 플러시에 hypercall를 사용 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>3</td>
            <td>메모리 매핑된 상응 하는 EOI, ICR 및 TPR에 액세스 하는 데 MSRs를 사용 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>4</td>
            <td>하이퍼바이저 제공 MSR을 사용 하 여 시스템 재설정을 시작 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>5</td>
            <td>이 파티션에 대해 완화 시간을 사용 하는 것이 좋습니다. 사용 되는 경우 VM은 적시에 외부 인터럽트를 배달 하는 데 사용 되는 모든 watchdog 시간 제한을 사용 하지 않도록 설정 해야 합니다.</td>
        </tr>
        <tr>
            <td>6</td>
            <td>DMA 다시 매핑을 사용 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>7</td>
            <td>인터럽트 다시 매핑을 사용 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>8</td>
            <td>예약되어 있습니다.</td>
        </tr>
        <tr>
            <td>9</td>
            <td>사용 중단 AutoEOI를 권장 합니다.</td>
        </tr>
        <tr>
            <td>10</td>
            <td>SyntheticClusterIpi hypercall를 사용 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>11</td>
            <td>최신 ExProcessorMasks 인터페이스를 사용 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>12</td>
            <td>하이퍼바이저가 Hyper-v 파티션 내에 중첩 되어 있음을 나타냅니다.</td>
        </tr>
        <tr>
            <td>13</td>
            <td>MBEC 시스템 호출에 INT를 사용 하는 것이 좋습니다.</td>
        </tr>
        <tr>
            <td>14</td>
            <td>지원 VMCS 인터페이스를 사용 하 여 중첩 된 하이퍼바이저를 권장 합니다. 또한 중첩 된 추가 enlightenments를 사용할 수 있음을 나타냅니다 (리프 0x4000000A 참조).</td>
        </tr>
        <tr>
            <td>15</td>
            <td>UseSyncedTimeline – 파티션이 루트 파티션에서 제공 된 QueryPerformanceCounter 바이어스를 사용 해야 함을 나타냅니다.</td>
        </tr>
        <tr>
            <td>16</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>17</td>
            <td>UseDirectLocalFlushEntire – 게스트가 CR4를 전환 해야 함을 나타냅니다. Hypercall를 만드는 것 보다 성능이 더 우수 하기 때문에 전체 TLB를 플러시하는 PGE.</td>
        </tr>
        <tr>
            <td>18</td>
            <td>NoNonArchitecturalCoreSharing-코어 공유를 사용할 수 없음을 나타냅니다. 이는 STIBP의 성능 오버 헤드를 방지 하기 위해 최적화로 사용할 수 있습니다.</td>
        </tr>
       <tr>
            <td>31-19</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>EBX</td>
            <td></td>
            <td>하이퍼바이저에 오류에 대해 알리기 전에 spinlock 실패를 재시도 하는 데 권장 되는 횟수입니다. 0xFFFFFFFF는 알리지 않음을 나타냅니다.</td>
        </tr>
        <tr>
            <td rowspan="2">ECX</td>
            <td>6-0</td>
            <td>구현 된 Edphaddressbits – 시스템의 실제 프로세서에서 보고 한 실제 주소 너비 (MAXPHYADDR)를 보고 합니다. 모든 비트에 0이 포함 된 경우이 기능은 지원 되지 않습니다. 보고 된 값은 실제 주소 비트의 실제 수 이며 해당 숫자를 나타내는 데 사용 되는 비트 위치가 아닙니다.</td>
        </tr>
        <tr>
            <td>31-7</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>EDX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
    </tbody>
</table>

### <a name="hypervisor-implementation-limits---0x40000005"></a>하이퍼바이저 구현 제한-0x40000005

현재 하이퍼바이저 구현에서 지원 되는 크기 제한을 설명 합니다. 값이 0 이면 하이퍼바이저는 해당 정보를 노출 하지 않습니다. 그렇지 않으면 이러한 의미를 갖습니다.

| 등록  | 제공된 정보                                                                 |
|-----------|--------------------------------------------------------------------------------------|
| EAX       | 지원 되는 최대 가상 프로세서 수                                   |
| EBX       | 지원 되는 최대 논리적 프로세서 수                                   |
| ECX       | 인터럽트 다시 매핑에 사용할 수 있는 실제 인터럽트 벡터의 최대 수입니다.  |
| EDX       | 예약됨                                                                             |

### <a name="implementation-hardware-features---0x40000006"></a>구현 하드웨어 기능-0x40000006

현재 하이퍼바이저에서 사용 중인 하드웨어 관련 기능을 나타냅니다.

<!---
+-----------+-------+---------------------------------------------------------------------------------------+
| Register  | Bits  | Information Provided                                                                  |
+===========+=======+=======================================================================================+
| EAX       | 0     | Support for APIC overlay assist is detected and in use                                |
+           +-------+---------------------------------------------------------------------------------------+
|           | 1     | Support for MSR bitmaps is detected and in use                                        |
+           +-------+---------------------------------------------------------------------------------------+
|           | 2     | Support for architectural performance counters is detected and in use                 |
+           +-------+---------------------------------------------------------------------------------------+
|           | 3     | Support for second level address translation is detected and in use                   |
+           +-------+---------------------------------------------------------------------------------------+
|           | 4     | Support for DMA remapping is detected and in use                                      |
+           +-------+---------------------------------------------------------------------------------------+
|           | 5     | Support for interrupt remapping is detected and in use                                |
+           +-------+---------------------------------------------------------------------------------------+
|           | 6     | Indicates that a memory patrol scrubber is present in the hardware                    |
+           +-------+---------------------------------------------------------------------------------------+
|           | 7     | DMA protection is in use                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 8     | HPET is requested                                                                     |
+           +-------+---------------------------------------------------------------------------------------+
|           | 9     | Synthetic timers are volatile                                                         |
+           +-------+---------------------------------------------------------------------------------------+
|           | 31-10 | Reserved                                                                              |
+-----------+-------+---------------------------------------------------------------------------------------+
| EBX       | Reserved                                                                                      |
+-----------+-----------------------------------------------------------------------------------------------+
| ECX       | Reserved                                                                                      |
+-----------+-----------------------------------------------------------------------------------------------+
| EDX       | Reserved                                                                                      |
-->

<table>
    <thead>
        <tr>
            <th>등록</th>
            <th>비트</th>
            <th>제공된 정보</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="11">EAX</td>
            <td>0</td>
            <td>APIC 오버레이 지원에 대 한 지원이 검색 되어 사용 되 고 있습니다.</td>
        </tr>
        <tr>
            <td>1</td>
            <td>MSR 비트맵에 대 한 지원이 검색 되어 사용 되 고 있습니다.</td>
        </tr>
        <tr>
            <td>2</td>
            <td>아키텍처 성능 카운터에 대 한 지원이 검색 되어 사용 되 고 있습니다.</td>
        </tr>
        <tr>
            <td>3</td>
            <td>두 번째 수준 주소 변환에 대 한 지원이 검색 되어 사용 되 고 있습니다.</td>
        </tr>
        <tr>
            <td>4</td>
            <td>DMA 다시 매핑에 대 한 지원이 검색 되어 사용 되 고 있습니다.</td>
        </tr>
        <tr>
            <td>5</td>
            <td>인터럽트 다시 매핑에 대 한 지원이 검색 되어 사용 되 고 있습니다.</td>
        </tr>
        <tr>
            <td>6</td>
            <td>하드웨어에 메모리 순찰 스크러버가 있음을 나타냅니다.</td>
        </tr>
        <tr>
            <td>7</td>
            <td>DMA 보호를 사용 하 고 있습니다.</td>
        </tr>
        <tr>
            <td>8</td>
            <td>HPET가 요청 되었습니다.</td>
        </tr>
        <tr>
            <td>9</td>
            <td>가상 타이머가 일시적입니다.</td>
        </tr>
        <tr>
            <td>31-10</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>EBX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>ECX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>EDX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
    </tbody>
</table>

### <a name="nested-hypervisor-feature-identification---0x40000009"></a>중첩 된 하이퍼바이저 기능 Id-0x40000009

중첩 된를 실행할 때 하이퍼바이저에 의해 파티션에 노출 되는 기능을 설명 합니다. EAX는 가상 MSRs에 대 한 액세스를 설명 합니다. EDX는 hypercall에 대 한 액세스를 설명 합니다.

<!---
+-----------+-------+---------------------------------------------------------------------------------------+
| Register  | Bits  | Information Provided                                                                  |
+===========+=======+=======================================================================================+
| EAX       | 1-0   | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 2     | AccessSynicRegs                                                                       |
+           +-------+---------------------------------------------------------------------------------------+
|           | 3     | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 4     | AccessIntrCtrlRegs                                                                    |
+           +-------+---------------------------------------------------------------------------------------+
|           | 5     | AccessHypercallMsrs                                                                   |
+           +-------+---------------------------------------------------------------------------------------+
|           | 6     | AccessVpIndex                                                                         |
+           +-------+---------------------------------------------------------------------------------------+
|           | 11-7  | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 12    | AccessReenlightenmentControls                                                         |
+           +-------+---------------------------------------------------------------------------------------+
|           | 31-13 | Reserved                                                                              |
+-----------+-------+---------------------------------------------------------------------------------------+
| EBX       | Reserved                                                                                      |
+-----------+-----------------------------------------------------------------------------------------------+
| ECX       | Reserved                                                                                      |
+-----------+-----------------------------------------------------------------------------------------------+
| EDX       | 3-0   | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 4     | XmmRegistersForFastHypercallAvailable                                                 |
+           +-------+---------------------------------------------------------------------------------------+
|           | 14-5  | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 15    | FastHypercallOutputAvailable                                                          |
+           +-------+---------------------------------------------------------------------------------------+
|           | 16    | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 17    | SintPollingModeAvailable                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 31-18 | Reserved                                                                              |
-->

<table>
    <thead>
        <tr>
            <th>등록</th>
            <th>비트</th>
            <th>제공된 정보</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="9">EAX</td>
            <td>1-0</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>2</td>
            <td>AccessSynicRegs</td>
        </tr>
        <tr>
            <td>3</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>4</td>
            <td>AccessIntrCtrlRegs</td>
        </tr>
        <tr>
            <td>5</td>
            <td>AccessHypercallMsrs</td>
        </tr>
        <tr>
            <td>6</td>
            <td>AccessVpIndex</td>
        </tr>
        <tr>
            <td>11-7</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>12</td>
            <td>AccessReenlightenmentControls</td>
        </tr>
        <tr>
            <td>31-13</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>EBX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>ECX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td rowspan="7">EDX</td>
            <td>3-0</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>4</td>
            <td>XmmRegistersForFastHypercallAvailable</td>
        </tr>
        <tr>
            <td>14-5</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>15</td>
            <td>FastHypercallOutputAvailable</td>
        </tr>
        <tr>
            <td>16</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>17</td>
            <td>SintPollingModeAvailable</td>
        </tr>
        <tr>
            <td>31-18</td>
            <td>예약됨</td>
        </tr>
    </tbody>
</table>

### <a name="hypervisor-nested-virtualization-features---0x4000000a"></a>하이퍼바이저 중첩 가상화 기능-0x4000000A

중첩 된 하이퍼바이저에서 사용할 수 있는 중첩 된 가상화 최적화를 나타냅니다.

<!---
+-----------+-------+---------------------------------------------------------------------------------------+
| Register  | Bits  | Information Provided                                                                  |
+===========+=======+=======================================================================================+
| EAX       | 7-0   | Enlightened VMCS version (low)                                                        |
+           +-------+---------------------------------------------------------------------------------------+
|           | 15-8  | Enlightened VMCS version (high)                                                       |
+           +-------+---------------------------------------------------------------------------------------+
|           | 16    | Reserved                                                                              |
+           +-------+---------------------------------------------------------------------------------------+
|           | 17    | Indicates support for direct virtual flush hypercalls                                 |
+           +-------+---------------------------------------------------------------------------------------+
|           | 18    | Indicates support for the HvFlushGuestPhysicalAddressSpace and HvFlushGuestPhysicalAddressList hypercalls
+           +-------+---------------------------------------------------------------------------------------+
|           | 19    | Indicates support for using an enlightened MSR bitmap.                                |
+           +-------+---------------------------------------------------------------------------------------+
|           | 31-20 | Reserved                                                                              |
+-----------+-------+---------------------------------------------------------------------------------------+
| EBX       | Reserved                                                                                      |
+-----------+-----------------------------------------------------------------------------------------------+
| ECX       | Reserved                                                                                      |
+-----------+-----------------------------------------------------------------------------------------------+
| EDX       | Reserved                                                                                      |
-->

<table>
    <thead>
        <tr>
            <th>등록</th>
            <th>비트</th>
            <th>제공된 정보</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="8">EAX</td>
            <td>7-0</td>
            <td>지원 VMCS 버전 (낮음)</td>
        </tr>
        <tr>
            <td>15-8</td>
            <td>지원 VMCS 버전 (높음)</td>
        </tr>
        <tr>
            <td>16</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>17</td>
            <td>직접 가상 플러시 hypercall에 대 한 지원을 나타냅니다.</td>
        </tr>
        <tr>
            <td>18</td>
            <td>HvFlushGuestPhysicalAddressSpace 및 HvFlushGuestPhysicalAddressList hypercall에 대 한 지원을 나타냅니다.</td>
        </tr>
        <tr>
            <td>19</td>
            <td>지원 MSR 비트맵 사용에 대 한 지원을 나타냅니다.</td>
        </tr>
        <tr>
            <td>20</td>
            <td>페이지 폴트 예외 클래스에서 가상화 예외를 결합 하기 위한 지원을 나타냅니다.</td>
        </tr>
        <tr>
            <td>31-21</td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>EBX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>ECX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
        <tr>
            <td>EDX</td>
            <td></td>
            <td>예약됨</td>
        </tr>
    </tbody>
</table>

## <a name="versioning"></a>버전 관리

하이퍼바이저 버전 정보는 리프로 인코딩됩니다 `0x40000002` . 주 버전 및 서비스 버전 이라는 두 가지 버전 번호가 제공 됩니다.

주 버전에는 주 버전 번호와 부 버전 번호 및 빌드 번호가 포함 됩니다. 이러한 값은 Microsoft Windows 릴리스 번호에 해당 합니다. 서비스 버전은 주 버전에 대 한 변경 내용을 설명 합니다.

클라이언트는 `0x40000003` `0x40000005` 버전 범위와 비교 하는 대신 CPUID를 사용 하 여 하이퍼바이저 기능을 확인 하는 것이 좋습니다.