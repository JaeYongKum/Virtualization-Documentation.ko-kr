---
title: 가상 MMU
description: 하이퍼바이저 가상 MMU 인터페이스
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: 7bfb7ba5426df0192bf142a2fc4e492da2fa21ef
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120761"
---
# <a name="virtual-mmu"></a>가상 MMU

각 파티션에 의해 노출 되는 가상 컴퓨터 인터페이스에는 MMU (메모리 관리 단위)가 포함 됩니다. 하이퍼바이저 파티션에 의해 노출 되는 가상 MMU는 일반적으로 기존 MMUs와 호환 됩니다.

## <a name="virtual-mmu-overview"></a>가상 MMU 개요

가상 프로세서는 가상 메모리와 가상 TLB (변환 조회 버퍼)를 노출 하며,이는 가상 주소에서 (게스트) 실제 주소로의 변환을 캐시 합니다. 논리 프로세서의 TLB와 마찬가지로, 가상 TLB는 비 일관 된 캐시 이며, 이러한 비 일관성는 게스트에 게 표시 됩니다. 하이퍼바이저는 TLB를 플러시하는 작업을 노출 합니다. 게스트는 이러한 작업을 사용 하 여 잠재적으로 일치 하지 않는 항목을 제거 하 고 가상 주소 번역을 예측할 수 있습니다.

### <a name="compatibility"></a>호환성

하이퍼바이저에 의해 노출 되는 가상 MMU 일반적으로 x64 프로세서 내에서 발견 된 실제 MMU와 호환 됩니다. 다음과 같은 게스트 관찰 가능 차이점이 있습니다.

- CR3입니다. PWT 및 CR3. PCD bits는 일부 하이퍼바이저 구현에서 지원 되지 않을 수 있습니다. 이러한 구현에서 게스트가 CR3 명령 또는 작업 게이트 스위치를 통해 이러한 플래그를 설정 하는 시도는 무시 됩니다. HvSetVpRegisters 또는 HvSwitchVirtualAddressSpace를 통해 프로그래밍 방식으로 이러한 비트를 설정 하려고 하면 오류가 발생할 수 있습니다.
- 리프 페이지 테이블 항목 내의 PWT 및 PCD 비트 (예: 4-K 페이지의 경우 PTE 및 대용량 페이지의 경우 PDE)는 매핑되는 페이지의 캐시 가능성을 지정 합니다. 리프가 아닌 페이지 테이블 항목 내에서 PAT, PWT 및 PCD 비트는 계층 구조에 있는 다음 페이지 테이블의 캐시 가능성을 표시 합니다. 일부 하이퍼바이저 구현은 이러한 플래그를 지원 하지 않을 수 있습니다. 이러한 구현에서 하이퍼바이저에서 수행 하는 모든 페이지 테이블 액세스는 나중 쓰기 캐시 특성을 사용 하 여 수행 됩니다. 이는 특히 페이지 테이블 항목에 기록 된, 액세스 및 더티 비트에 영향을 줍니다. 게스트가 리프가 아닌 페이지 테이블 항목 내에서 PAT, PWT 또는 PCD 비트를 설정 하는 경우 가상 프로세서가 해당 페이지 테이블에 의해 매핑되는 페이지에 액세스 하면 "지원 되지 않는 기능" 메시지가 생성 될 수 있습니다.
- CR0 레지스터입니다. 일부 하이퍼바이저 구현에서는 CD (캐시 사용 안 함) 비트가 지원 되지 않을 수 있습니다. 이러한 구현에서는 CR0 레지스터입니다. CD 비트는 0으로 설정 해야 합니다. 게스트가 CR0 레지스터 명령을 통해이 플래그를 설정 하는 시도는 무시 됩니다. HvSetVpRegisters를 통해 프로그래밍 방식으로이 비트를 설정 하려고 하면 오류가 발생 합니다.
- PAT (페이지 주소 유형) MSR은 VP 별 레지스터입니다. 그러나 파티션의 모든 가상 프로세서가 PAT MSR을 동일한 값으로 설정 하면 새 효과는 파티션 전체에 적용 됩니다.
- 보안 및 격리를 위해 INVD 명령이 몇 가지 차이점이 있지만 WBINVD 명령 처럼 작동 하도록 가상화 됩니다. 보안을 위해 대신 CLFLUSH를 사용 해야 합니다.

### <a name="legacy-tlb-management-operations"></a>레거시 TLB 관리 작업

X64 아키텍처는 프로세서의 TLBs을 관리 하는 여러 가지 방법을 제공 합니다. 하이퍼바이저에서 가상화 하는 메커니즘은 다음과 같습니다.

- INVLPG 명령은 프로세서의 TLB에서 단일 페이지에 대 한 번역을 무효화 합니다. 지정 된 가상 주소를 원래 4kb 페이지로 매핑한 경우이 페이지에 대 한 번역이 TLB에서 제거 됩니다. 지정 된 가상 주소를 원래 "large page" (MMU 모드에 따라 2mb 또는 4mb)로 매핑한 경우 전체 크기의 페이지에 대 한 번역이 TLB에서 제거 됩니다. INVLPG 명령은 전역 및 비 전역 번역을 모두 플러시합니다. 전역 번역은 페이지 테이블 항목 내에 "전역" 비트가 설정 된 것으로 정의 됩니다.
- MOV to CR3 명령 및 작업 스위치는 프로세서의 TLB 내 모든 비 전역 페이지에 대 한 번역을 수정 CR3 무효화 합니다.
- CR4를 수정 하는 CR4 명령입니다. PGE (전역 페이지 사용) 비트 (CR4)입니다. PSE (페이지 크기 확장) 비트 또는 CR4. PAE (페이지 주소 확장) 비트는 프로세서의 TLB 내에서 모든 번역 (전역 및 비 전역)을 무효화 합니다.

이러한 모든 무효화 작업은 하나의 프로세서에만 영향을 줍니다. 다른 프로세서에서 번역을 무효화 하려면 소프트웨어에서 소프트웨어 기반 "TLB 다운" 메커니즘 (일반적으로 프로세스 간 인터럽트를 사용 하 여 구현)을 사용 해야 합니다.

### <a name="virtual-tlb-enhancements"></a>가상 TLB의 향상 된 기능

하이퍼바이저는 앞에서 설명한 레거시 TLB 관리 메커니즘을 지 원하는 것 외에도, 게스트에서 가상 TLB를 보다 효율적으로 관리할 수 있는 향상 된 기능 집합을 지원 합니다. 이러한 향상 된 작업은 레거시 TLB 관리 작업과 교대로 사용할 수 있습니다.

하이퍼바이저는 다음 hypercall를 지원 하 여 TLBs를 무효화 합니다.

| Hypercall                                                                           | Description                                     |
|-------------------------------------------------------------------------------------|-------------------------------------------------|
| [HvCallFlushVirtualAddressSpace](hypercalls/HvCallFlushVirtualAddressSpace.md)      | 지정 된 주소 공간에 속하는 모든 가상 TLB 항목을 무효화 합니다.    |
| [HvCallFlushVirtualAddressSpaceEx](hypercalls/HvCallFlushVirtualAddressSpaceEx.md)  | HvCallFlushVirtualAddressSpace와 마찬가지로는 스파스 VP를 입력으로 사용 합니다.    |
| [HvCallFlushVirtualAddressList](hypercalls/HvCallFlushVirtualAddressList.md)        | 지정 된 주소 공간의 일부를 무효화 합니다.    |
| [HvCallFlushVirtualAddressListEx](hypercalls/HvCallFlushVirtualAddressListEx.md)    | HvCallFlushVirtualAddressList와 마찬가지로, 스파스 VP를 입력으로 사용 합니다.    |

하드웨어에 대 한 가상화 지원이 충분 한 일부 시스템에서 레거시 TLB 관리 지침은 로컬 또는 원격 (프로세서 간) TLB 무효화의 경우 더 빠를 수 있습니다. 최적의 성능에 관심이 있는 게스트는 CPUID 리프 0x40000004를 사용 하 여 hypercall를 사용 하 여 구현할 동작을 결정 해야 합니다.

- UseHypercallForAddressSpaceSwitch:이 플래그가 설정 된 경우 호출자는 HvCallSwitchAddressSpace를 사용 하 여 주소 공간 간을 전환 하는 것이 더 빠른 것으로 가정 합니다. 이 플래그를 명확 하 게 하는 경우 MOV to CR3 명령이 권장 됩니다.
- UseHypercallForLocalFlush:이 플래그가 설정 된 경우 호출자는 가상 TLB에서 하나 이상의 페이지를 플러시하는 것과 달리 hypercall (INVLPG 또는 MOV to CR3)를 사용 하는 것이 더 빠른 것으로 가정 합니다.
- Usehypercallforremoteflus수동 Localflush전체:이 플래그가 설정 된 경우 호출자는 게스트 생성 프로세서 인터럽트를 사용 하는 것과는 반대로, hypercall를 사용 하 여 가상 TLB에서 하나 이상의 페이지를 플러시하는 것으로 가정 합니다.

## <a name="memory-cache-control-overview"></a>메모리 캐시 컨트롤 개요

하이퍼바이저는 게스트의 GVA 공간 내에 매핑된 페이지에 대해 게스트 정의 캐시 가능성 설정을 지원 합니다. 사용 가능한 캐시 설정 및 해당 의미에 대 한 자세한 내용은 Intel 또는 AMD 설명서를 참조 하세요.

가상 프로세서가 GVA 공간을 통해 페이지에 액세스 하는 경우 하이퍼바이저는 페이지를 매핑하는 데 사용 되는 게스트 페이지 테이블 항목 내에서 캐시 특성 비트 (PAT, PWT 및 PCD)를 적용 합니다. 이러한 3 비트는 파티션의 PAT (페이지 주소 유형) 레지스터에 인덱스를 사용 하 여 페이지에 대 한 최종 캐시 가능성 설정을 조회 합니다.

GPA 공간을 통해 직접 액세스 한 페이지 (예: CR0 레지스터가 페이징을 사용 하지 않도록 설정 된 경우) PG가 지워진 경우 MTRRs에서 정의 된 캐시를 사용 합니다. 하이퍼바이저 구현에서 가상 MTRRs을 지원 하지 않는 경우 AMR-WB 캐시 가능성을 가정 합니다.

### <a name="mixing-cache-types-between-a-partition-and-the-hypervisor"></a>파티션과 하이퍼바이저 간에 캐시 유형 혼합

게스트가 GPA 공간 내의 일부 페이지를 하이퍼바이저에서 액세스할 수 있음을 인식 해야 합니다. 다음 목록에는 여러 예제가 나와 있습니다.

- hypercall에 대 한 입력 또는 출력 매개 변수를 포함 하는 페이지입니다.
- Hypercall 페이지, Syic SIEF 및 SIM 페이지, 통계 페이지를 비롯 한 모든 오버레이 페이지

하이퍼바이저는 항상 AMR-WB 캐시 가능성 설정을 사용 하 여 hypercall 매개 변수 및 오버레이 페이지에 대 한 액세스를 수행 합니다.