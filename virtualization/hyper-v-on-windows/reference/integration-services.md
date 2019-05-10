---
title: Hyper-V 통합 서비스
description: Hyper-V 통합 서비스 참조
keywords: windows 10, hyper-v, 통합 서비스, 통합 구성 요소
author: scooley
ms.date: 05/25/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 18930864-476a-40db-aa21-b03dfb4fda98
ms.openlocfilehash: 762b82f3714651ffb488f682581680c9526404a8
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621211"
---
# <a name="hyper-v-integration-services"></a>Hyper-V 통합 서비스

통합 서비스(통합 구성 요소라고도 함)는 가상 컴퓨터가 Hyper-V 호스트와 통신하도록 허용하는 서비스입니다. 이러한 서비스 중 상당수는 편리하며 나머지 서비스는 가상 컴퓨터의 기능이 제대로 작동하도록 하는 데 매우 중요할 수 있습니다.

이 문서는 Windows에서 사용할 수 있는 각 통합 서비스에 대한 참조입니다.  또한 특정 통합 서비스 또는 통합 서비스 기록과 관련된 정보에 대한 시작 지점으로 사용할 수 있습니다.

**사용자 가이드:**  
* [통합 서비스 관리](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Manage-Hyper-V-integration-services)


## <a name="quick-reference"></a>빠른 참조

| Name | Windows 서비스 이름 | Linux 디먼 이름 |  설명 | 사용하지 않을 때 VM에 미치는 영향 |
|:---------|:---------|:---------|:---------|:---------|
| [Hyper-V 하트비트 서비스](#hyper-v-heartbeat-service) |  vmicheartbeat | hv_utils | 가상 컴퓨터가 제대로 실행되고 있는지 보고합니다. | 상황에 따라 다름 |
| [Hyper-V 게스트 종료 서비스](#hyper-v-guest-shutdown-service) | vmicshutdown | hv_utils |  호스트에서 가상 컴퓨터 종료를 트리거하도록 허용합니다. | **높은** |
| [Hyper-V 시간 동기화 서비스](#hyper-v-time-synchronization-service) | vmictimesync | hv_utils | 가상 컴퓨터의 시계를 호스트 컴퓨터의 시계와 동기화합니다. | **높은** |
| [Hyper-V 데이터 교환 서비스(KVP)](#hyper-v-data-exchange-service-kvp) | vmickvpexchange | hv_kvp_daemon | 가상 컴퓨터와 호스트 간에 기본 메타데이터를 교환하는 방법을 제공합니다. | 중형 |
| [Hyper-V 볼륨 섀도 복사본 요청자](#hyper-v-volume-shadow-copy-requestor) | vmicvss | hv_vss_daemon | 볼륨 섀도 복사본 서비스에서 종료하지 않고 가상 컴퓨터를 백업하도록 허용합니다. | 상황에 따라 다름 |
| [Hyper-V 게스트 서비스 인터페이스](#hyper-v-powershell-direct-service) | vmicguestinterface | hv_fcopy_daemon | 가상 컴퓨터에 파일을 복사하거나 가상 컴퓨터의 파일을 복사할 수 있는 Hyper-V 호스트에 대한 인터페이스를 제공합니다. | 낮음 |
| [Hyper-V PowerShell Direct 서비스](#hyper-v-powershell-direct-service) | vmicvmsession | 사용할 수 없음 | 네트워크 연결 없이 PowerShell을 사용하여 가상 컴퓨터를 관리하는 방법을 제공합니다. | 낮음 |  


## <a name="hyper-v-heartbeat-service"></a>Hyper-V 하트비트 서비스

**Windows 서비스 이름:** vmicheartbeat  
**Linux 디먼 이름:** hv_utils  
**설명:** 가상 컴퓨터에 운영 체제가 설치되어 있으며 올바르게 부팅되었음을 Hyper-V 호스트에 알립니다.  
**추가됨:** Windows Server 2012, Windows 8  
**영향:** 사용하지 않도록 설정하면 가상 컴퓨터에서 가상 컴퓨터 내의 운영 체제가 제대로 작동하고 있는지 보고할 수 없습니다.  일부 유형의 모니터링 및 호스트 쪽 진단에 영향을 미칠 수 있습니다.  

하트비트 서비스는 “가상 컴퓨터가 부팅되었나요?” 같은 기본적인 질문에 대답할 수 있도록 합니다.  

Hyper-V에서 가상 컴퓨터 상태가 “실행 중”인 것으로 보고하면(아래 예제 참조) Hyper-V에서 가상 컴퓨터에 대해 리소스를 예약해 두었다는 의미이며, 설치되었거나 작동 중인 운영 체제가 있다는 의미가 아닙니다.  이 경우 하트비트가 유용합니다.  하트비트 서비스는 가상 컴퓨터 내의 운영 체제가 부팅되었음을 Hyper-V에 알립니다.  

### <a name="check-heartbeat-with-powershell"></a>PowerShell을 사용하여 하트비트 확인

관리자는 [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps)을 실행하여 가상 컴퓨터의 하트비트를 확인합니다.
``` PowerShell
Get-VM -VMName $VMName | select Name, State, Status
```

출력은 다음과 같이 표시됩니다.
```
Name    State    Status
----    -----    ------
DemoVM  Running  Operating normally
```

`Status` 필드는 하트비트 서비스에 의해 결정됩니다.



## <a name="hyper-v-guest-shutdown-service"></a>Hyper-V 게스트 종료 서비스

**Windows 서비스 이름:** vmicshutdown  
**Linux 디먼 이름:** hv_utils  
**설명:** Hyper-V 호스트에서 가상 컴퓨터 종료를 요청할 수 있도록 허용합니다.  호스트는 언제나 가상 컴퓨터를 강제로 끌 수 있지만, 이는 종료를 선택하는 것이 아니라 전원 스위치를 누르는 것과 같습니다.  
**추가됨:** Windows Server 2012, Windows 8  
**영향:** **강력한 영향**  사용하지 않도록 설정하면 호스트는 가상 컴퓨터 내에서 종료를 트리거할 수 없습니다.  하드 전원 오프 데이터 손실이 나 데이터 손상을 일으킬 수 있는 모든 끄기가 됩니다.  


## <a name="hyper-v-time-synchronization-service"></a>Hyper-V 시간 동기화 서비스

**Windows 서비스 이름:** vmictimesync  
**Linux 디먼 이름:** hv_utils  
**설명:** 물리적 컴퓨터의 시스템 시계와 가상 컴퓨터의 시스템 시계를 동기화합니다.  
**추가됨:** Windows Server 2012, Windows 8  
**영향:** **강력한 영향**  사용하지 않도록 설정하면 가상 컴퓨터의 시계는 이상하게 작동합니다.  


## <a name="hyper-v-data-exchange-service-kvp"></a>Hyper-V 데이터 교환 서비스(KVP)

**Windows 서비스 이름:** vmickvpexchange  
**Linux 디먼 이름:** hv_kvp_daemon  
**설명:** 가상 컴퓨터와 호스트 간에 기본 메타데이터를 교환하는 방법을 제공합니다.  
**추가됨:** Windows Server 2012, Windows 8  
**영향:** 사용하지 않도록 설정하면 Windows 8 또는 Windows Server 2012 이전 버전을 실행하는 가상 컴퓨터에서 Hyper-V 통합 서비스에 대한 업데이트를 받지 않습니다.  데이터 교환을 사용하지 않도록 설정하면 일부 유형의 모니터링 및 호스트 쪽 진단에 영향을 미칠 수도 있습니다.  

데이터 교환 서비스(때때로 KVP라고 함)는 Windows 레지스트리를 통해 키-값 쌍(KVP)을 사용하여 가상 컴퓨터와 Hyper-V 호스트 간에 소량의 컴퓨터 정보를 공유합니다.  또한 가상 컴퓨터와 호스트 간에 사용자 지정된 데이터를 공유하는 데에도 동일한 메커니즘이 사용될 수 있습니다.

키-값 쌍은 "키"와 "값"으로 구성됩니다. 키와 값 모두 문자열이며, 다른 데이터 형식은 지원되지 않습니다. 키-값 쌍을 만들거나 변경할 때 게스트와 호스트에 표시됩니다. 키-값 쌍 정보는 Hyper-V VMbus를 통해 전송되며 게스트와 Hyper-V 간에 어떤 종류의 네트워크 연결도 필요하지 않습니다. 

데이터 교환 서비스는 가상 컴퓨터에 대한 정보를 유지하기 위해 좋은 도구입니다. 대화형 데이터 공유 또는 데이터 전송의 경우 [PowerShell Direct](#hyper-v-powershell-direct-service)를 사용하세요. 


**사용자 가이드:**  
* [키-값 쌍을 사용하여 Hyper-V의 호스트 및 게스트 간에 정보를 공유합니다](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn798287(v=ws.11)).  


## <a name="hyper-v-volume-shadow-copy-requestor"></a>Hyper-V 볼륨 섀도 복사본 요청자

**Windows 서비스 이름:** vmicvss  
**Linux 디먼 이름:** hv_vss_daemon  
**설명:** 볼륨 섀도 복사본 서비스에서 가상 컴퓨터에 응용 프로그램 및 데이터를 백업할 수 있도록 허용합니다.  
**추가됨:** Windows Server 2012, Windows 8  
**영향:** 사용하도록 설정하면 가상 컴퓨터를 실행하는 중 백업할 수 없습니다(VSS 사용).  

볼륨 섀도 복사본 요청자 통합 서비스는 볼륨 섀도 복사본 서비스([VSS](https://docs.microsoft.com/windows/desktop/VSS/overview-of-processing-a-backup-under-vss))에 대해 필요합니다.  VSS(볼륨 섀도 복사본 서비스)는 제공하는 서비스의 성능 및 안정성을 지나치게 저하시키지 않으면서 실행 중인 시스템(특히 서버)에서 백업에 대한 이미지를 캡처하고 복사합니다.  이 통합 서비스는 가상 컴퓨터의 작업과 호스트의 백업 프로세스를 조정하여 그러한 작업을 가능하게 만듭니다.

볼륨 섀도 복사본에 대한 자세한 내용은 [여기](https://docs.microsoft.com/previous-versions/windows/desktop/virtual/backing-up-and-restoring-virtual-machines)에서 확인하세요.


## <a name="hyper-v-guest-service-interface"></a>Hyper-V 게스트 서비스 인터페이스

**Windows 서비스 이름:** vmicguestinterface  
**Linux 디먼 이름:** hv_fcopy_daemon  
**설명:** 가상 컴퓨터에 파일을 복사하거나 가상 컴퓨터의 파일을 복사할 수 있는 Hyper-V 호스트에 대한 인터페이스를 제공합니다.  
**추가됨:** Windows Server 2012 R2, Windows 8.1  
**영향:** 사용하도록 설정하면 호스트에서 `Copy-VMFile`을 사용하여 게스트에 파일을 복사하거나 게스트의 파일을 복사할 수 없습니다.  [Copy-VMFile cmdlet](https://docs.microsoft.com/powershell/module/hyper-v/copy-vmfile?view=win10-ps)에 대해 자세히 알아보세요.  

**참고:**  
기본적으로 사용할 수 없게 설정되어 있습니다.  [Copy-Item을 사용하는 PowerShell Direct](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)를 참조하세요. 


## <a name="hyper-v-powershell-direct-service"></a>Hyper-V PowerShell Direct 서비스

**Windows 서비스 이름:** vmicvmsession  
**Linux 디먼 이름:** 해당 없음  
**설명:** 가상 네트워크 없이 VM 세션을 통해 PowerShell이 포함된 가상 컴퓨터를 관리하는 메커니즘을 제공합니다.    
**추가됨:** Windows 서버 TP3, Windows 10  
**영향:** 이 서비스를 사용하지 않도록 설정하면 PowerShell Direct를 사용하여 가상 컴퓨터에 연결할 수 없습니다.  

**참고:**  
서비스 이름은 원래 Hyper-V VM 세션 서비스였습니다.  
PowerShell Direct는 개발 중이며 Windows 10/Windows Server Technical Preview 3 이상 호스트/게스트에서만 사용할 수 있습니다.

PowerShell Direct는 Hyper-V 호스트 또는 가상 컴퓨터에서 네트워크 구성 또는 원격 관리 설정에 관계없이 Hyper-V 호스트의 가상 컴퓨터 내에서 PowerShell 관리를 허용합니다. 이를 통해 Hyper-V 관리자는 관리 및 구성 작업을 더욱 쉽게 자동화하고 스크립트할 수 있습니다.

[PowerShell Direct에 대해 자세히 알아보세요](../user-guide/powershell-direct.md).  

**사용자 가이드:**  
* [가상 컴퓨터에서 실행되는 스크립트](../user-guide/powershell-direct.md#run-a-script-or-command-with-invoke-command)
* [가상 컴퓨터에 파일 복사 및 가상 컴퓨터의 파일 복사](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)
