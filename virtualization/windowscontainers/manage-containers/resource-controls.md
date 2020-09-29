---
title: 리소스 컨트롤 구현
description: Windows 컨테이너용 리소스 컨트롤에 대한 세부 정보
keywords: Docker, 컨테이너, cpu, 메모리, 디스크, 리소스
author: taylorb-microsoft
ms.author: jgerend
ms.date: 08/12/2020
ms.topic: conceptual
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: 78a4dbcab22bf8a6b427f252f197929874e48a38
ms.sourcegitcommit: 160405a16d127892b6e2897efa95680f29f0496a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/22/2020
ms.locfileid: "90990706"
---
# <a name="implementing-resource-controls-for-windows-containers"></a>Windows 컨테이너용 리소스 컨트롤 구현

컨테이너별 및 리소스별로 구현할 수 있는 몇 가지 리소스 컨트롤이 있습니다.  기본적으로 컨테이너 실행은 대개 공평한 공유를 기반으로 하지만 이러한 컨트롤의 구성을 기반으로 하는 일반적인 Windows 리소스 관리에 따라 적용되지만 개발자 또는 관리자는 리소스 사용을 제한하거나 이에 영향을 미칠 수 있습니다.  제어할 수 있는 리소스는 CPU/프로세서, 메모리/RAM, 디스크/스토리지 및 네트워킹/처리량입니다.

Windows 컨테이너는 [작업 개체](/windows/desktop/ProcThread/job-objects)를 활용하여 각 컨테이너와 관련된 프로세스를 그룹화하고 추적합니다.  컨테이너와 관련된 상위 작업 개체에 리소스 컨트롤이 구현됩니다.

[Hyper-V 격리](./hyperv-container.md)의 경우 리소스 컨트롤은 가상 컴퓨터뿐만 아니라 가상 컴퓨터 내에서 자동으로 실행되는 컨테이너의 작업 개체에도 적용되며 이로 인해 프로세스가 작업 개체 컨트롤을 바이패스하거나 이스케이프한 컨테이너에서 실행되더라도 가상 컴퓨터에서 정의된 리소스 컨트롤을 초과하지 못하도록 합니다.

## <a name="resources"></a>리소스

각 리소스에 대해 이 섹션은 해당되는 Windows HCS(호스트 컴퓨팅 서비스) API에 대해 리소스 컨트롤을 어떻게 사용할 수 있는지(조정자 또는 다른 도구로 구성할 수 있음)와 일반적으로 리소스 컨트롤이 Windows에 의해 어떻게 구현되는지에 대한 예로서의 Docker 명령줄 인터페이스 간 매핑을 제공합니다(이 설명은 개괄적이며 기본 구현은 변경될 수 있음).

### <a name="memory"></a>메모리

| 리소스 | 위치 |
|-----|------|
| Docker 인터페이스 | [--memory](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| HCS 인터페이스 | [MemoryMaximumInMB](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOB_OBJECT_LIMIT_JOB_MEMORY](/windows/desktop/api/winnt/ns-winnt-_jobobject_basic_limit_information) |
| Hyper-V 격리 | 가상 컴퓨터 메모리 |

>[!NOTE]
>Windows Server 2016의 Hyper-V 격리의 경우 메모리 캡을 사용할 때 컨테이너가 처음에 메모리 캡 용량을 할당한 다음, 컨테이너 호스트로 반환하기 시작합니다. 이후 버전의 Windows Server(1709 이상)는 이 프로세스를 최적화했습니다.

### <a name="cpu-count"></a>CPU(수)

| 리소스 | 위치 |
|---|---|
| Docker 인터페이스 | [--cpus](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 인터페이스 | [ProcessorCount](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)로 시뮬레이트됨* |
| Hyper-V 격리 | 노출된 가상 프로세서 수 |

### <a name="cpu-percent"></a>CPU(백분율)

| 리소스 | 위치 |
|---|---|
| Docker 인터페이스 | [--cpu-percent](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 인터페이스 | [ProcessorMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Hyper-V 격리 | 가상 프로세서에서 하이퍼바이저 제한 |

### <a name="cpu-shares"></a>CPU(공유)

| 리소스 | 위치 |
|---|---|
| Docker 인터페이스 | [--cpu-shares](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 인터페이스 | [ProcessorWeight](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Hyper-V 격리 | 하이퍼바이저 가상 프로세서 가중치 |

### <a name="storage-image"></a>스토리지(이미지)

| 리소스 | 위치 |
|---|---|
| Docker 인터페이스 | [--io-maxbandwidth/--io-maxiops](https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| HCS 인터페이스 | [StorageIOPSMaximum 및 StorageBandwidthMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Hyper-V 격리 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |

### <a name="storage-volumes"></a>스토리지(볼륨)

| 리소스 | 위치 |
|---|---|
| Docker 인터페이스 | [--storage-opt size=](https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| HCS 인터페이스 | [StorageSandboxSize](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Hyper-V 격리 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |

## <a name="additional-notes-or-details"></a>추가 메모 및 세부 정보

### <a name="memory-requirements"></a>메모리 요구 사항

Windows 컨테이너는 일반적으로 사용자 관리, 네트워킹 등 컨테이너별 기능을 제공하는 각 컨테이너에서 일부 시스템 프로세스를 실행합니다. 이러한 프로세스에서 요구하는 메모리의 대부분은 컨테이너 간에 공유되며 메모리 용량은 이러한 메모리를 허용할 수 있도록 충분히 높아야 합니다.  Hyper-V 격리가 있거나 없는 각 기본 이미지 유형은 [시스템 요구 사항](../deploy-containers/system-requirements.md#memory-requirements) 문서에서 표로 제공됩니다.

### <a name="cpu-shares-without-hyper-v-isolation"></a>CPU 공유(Hyper-V 격리 없음)

CPU 공유를 사용할 때 기본 구현(Hyper-V 격리 사용하지 않음)은 [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)을 구성하며, 특히 JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED에 컨트롤 플래그를 설정하고 적절한 가중치를 제공합니다.  작업 개체의 유효한 가중치 범위는 1~9이며 기본값은 1~10000의 호스트 컴퓨팅 서비스 값보다 낮은 정확도인 5입니다.  예를 들면 공유 가중치가 7500이면 가중치는 7이고, 공유 가중치가 2500이면 값은 2입니다.