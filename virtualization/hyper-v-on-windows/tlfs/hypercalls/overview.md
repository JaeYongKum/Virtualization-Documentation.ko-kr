---
title: Hypercall 참조
description: 지원 되는 hypercall 목록
keywords: hyper-v
author: alexgrest
ms.author: alegre
ms.date: 10/15/2020
ms.topic: reference
ms.prod: windows-10-hyperv
ms.openlocfilehash: d40fc621374d7c3dacf616c01f1ea1efb3f08364
ms.sourcegitcommit: d43632e3dc22e2b7a256b5c15fbb665c047de286
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92120937"
---
# <a name="hypercall-reference"></a>Hypercall 참조

다음 표에서는 호출 코드에 의해 지원 되는 hypercall를 나열 합니다.

| 호출 코드 | 형식    | Hypercall                                                                           |
|-----------|---------|-------------------------------------------------------------------------------------|
| 0x0001    | 단순  | HvCallSwitchVirtualAddressSpace                                                     |
| 0x0002    | 단순  | [HvCallFlushVirtualAddressSpace](HvCallFlushVirtualAddressSpace.md)                 |
| 0x0003    | Rep     | [HvCallFlushVirtualAddressList](HvCallFlushVirtualAddressList.md)                   |
| 0x0008    | 단순  | [HvCallNotifyLongSpinWait](HvCallNotifyLongSpinWait.md)                             |
| 0x000b    | 단순  | [HvCallSendSyntheticClusterIpi](HvCallSendSyntheticClusterIpi.md)                   |
| 0x000c    | Rep     | [HvCallModifyVtlProtectionMask](HvCallModifyVtlProtectionMask.md)                   |
| 0x000d    | 단순  | [HvCallEnablePartitionVtl](HvCallEnablePartitionVtl.md)                             |
| 0x000f    | 단순  | [HvCallEnableVpVtl](HvCallEnableVpVtl.md)                                           |
| 0x0011    | 단순  | [HvCallVtlCall](HvCallVtlCall.md)                                                   |
| 0x0012    | 단순  | [HvCallVtlReturn](HvCallVtlReturn.md)                                               |
| 0x0013    | 단순  | [HvCallFlushVirtualAddressSpaceEx](HvCallFlushVirtualAddressSpaceEx.md)             |
| 0x0014    | Rep     | [HvCallFlushVirtualAddressListEx](HvCallFlushVirtualAddressListEx.md)               |
| 0x0015    | 단순  | [HvCallSendSyntheticClusterIpiEx](HvCallSendSyntheticClusterIpiEx.md)               |
| 0x0050    | Rep     | [HvCallGetVpRegisters](HvCallGetVpRegisters.md)                                     |
| 0x0051    | Rep     | [HvCallSetVpRegisters](HvCallSetVpRegisters.md)                                     |
| 0x005C    | 단순  | [HvCallPostMessage](HvCallPostMessage.md)                                           |
| 0x005D    | 단순  | [HvCallSignalEvent](HvCallSignalEvent.md)                                           |
| 0x007e    | 단순  | [HvCallRetargetDeviceInterrupt](HvCallRetargetDeviceInterrupt.md)                   |
| 0x0099    | 단순  | [HvCallStartVirtualProcessor](HvCallStartVirtualProcessor.md)                       |
| 0x009A    | Rep     | [HvCallGetVpIndexFromApicId](HvCallGetVpIndexFromApicId.md)                         |
| 0x00AF    | 단순  | [HvCallFlushGuestPhysicalAddressSpace](HvCallFlushGuestPhysicalAddressSpace.md)     |
| 0x00B0    | Rep     | [HvCallFlushGuestPhysicalAddressList](HvCallFlushGuestPhysicalAddressList.md)       |

다음 표에서는 호출 코드에 의해 지원 되는 확장 된 hypercall를 나열 합니다.

| 호출 코드 | 형식    | Hypercall                                                                           |
|-----------|---------|-------------------------------------------------------------------------------------|
| 0x8001    | 단순  | [HvExtCallQueryCapabilities](HvExtCallQueryCapabilities.md)                         |
| 0x8002    | 단순  | HvExtCallGetBootZeroedMemory                                                        |
| 0x8003    | 단순  | HvExtCallMemoryHeatHint                                                             |
| 0x8004    | 단순  | HvExtCallEpfSetup                                                                   |
| 0x8006    | 단순  | HvExtCallMemoryHeatHintAsync                                                        |
