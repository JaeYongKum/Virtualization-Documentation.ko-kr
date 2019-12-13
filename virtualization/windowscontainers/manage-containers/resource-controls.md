---
title: 리소스 컨트롤 구현
description: Windows 컨테이너용 리소스 컨트롤에 대한 세부 정보
keywords: Docker, 컨테이너, cpu, 메모리, 디스크, 리소스
author: taylorb-microsoft
ms.date: 11/21/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: 3e9f7e3208222cd6c0f512c5f892453ac6e6980c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910173"
---
# <a name="implementing-resource-controls-for-windows-containers"></a>Windows 컨테이너용 리소스 컨트롤 구현
컨테이너별 및 리소스별로 구현할 수 있는 몇 가지 리소스 컨트롤이 있습니다.  기본적으로 컨테이너 실행은 대개 공평한 공유를 기반으로 하지만 이러한 컨트롤의 구성을 기반으로 하는 일반적인 Windows 리소스 관리에 따라 적용되지만 개발자 또는 관리자는 리소스 사용을 제한하거나 이에 영향을 미칠 수 있습니다.  제어할 수 있는 리소스에는 CPU/프로세서, 메모리/RAM, 디스크/저장소 및 네트워킹/처리량이 포함됩니다.

Windows 컨테이너는 [작업 개체](https://docs.microsoft.com/windows/desktop/ProcThread/job-objects)를 활용하여 각 컨테이너와 관련된 프로세스를 그룹화하고 추적합니다.  컨테이너와 관련된 상위 작업 개체에 리소스 컨트롤이 구현됩니다. 

[Hyper-V 격리](./hyperv-container.md)의 경우 리소스 컨트롤은 가상 컴퓨터뿐만 아니라 가상 컴퓨터 내에서 자동으로 실행되는 컨테이너의 작업 개체에도 적용되며 이로 인해 프로세스가 작업 개체 컨트롤을 바이패스하거나 이스케이프한 컨테이너에서 실행되더라도 가상 컴퓨터에서 정의된 리소스 컨트롤을 초과하지 못하도록 합니다.

## <a name="resources"></a>리소스
각 리소스에 대해 이 섹션은 해당되는 Windows HCS(호스트 계산 서비스) API에 대해 리소스 컨트롤을 어떻게 사용할 수 있는지(조정자 또는 다른 도구로 구성할 수 있음)와 일반적으로 리소스 컨트롤이 Windows에 의해 어떻게 구현되는지에 대한 예로서의 Docker 명령줄 인터페이스 간 매핑을 제공합니다(이 설명은 개괄적이며 기본 구현은 변경될 수 있음).

|  | |
| ----- | ------|
| *메모리* ||
| Docker 인터페이스 | [--메모리](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| HCS 인터페이스 | [MemoryMaximumInMB](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOB_OBJECT_LIMIT_JOB_MEMORY](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_basic_limit_information) |
| Hyper-V 격리 | 가상 컴퓨터 메모리 |
| _Windows Server 2016의 Hyper-v 격리와 관련 된 참고 사항: 메모리 한도를 사용 하는 경우 컨테이너는 처음에 cap 크기를 할당 하 고 컨테이너 호스트로 다시 반환 하기 시작 합니다.  이후 버전 (1709 이상)에서이는 최적화 되었습니다._ |
| ||
| *CPU (개수)* ||
| Docker 인터페이스 | [--cpu](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 인터페이스 | [ProcessorCount](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)로 시뮬레이트됨* |
| Hyper-V 격리 | 노출된 가상 프로세서 수 |
| ||
| *CPU (백분율)* ||
| Docker 인터페이스 | [--cpu-%](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 인터페이스 | [ProcessorMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Hyper-V 격리 | 가상 프로세서에서 하이퍼바이저 제한 |
| ||
| *CPU (공유)* ||
| Docker 인터페이스 | [--cpu-공유](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 인터페이스 | [ProcessorWeight](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Hyper-V 격리 | 하이퍼바이저 가상 프로세서 가중치 |
| ||
| *저장소 (이미지)* ||
| Docker 인터페이스 | [--io-maxbandwidth/--io-maxbandwidth](https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| HCS 인터페이스 | [StorageIOPSMaximum 및 StorageBandwidthMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Hyper-V 격리 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| ||
| *저장소 (볼륨)* ||
| Docker 인터페이스 | [--저장소-최적 크기 =](https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| HCS 인터페이스 | [StorageSandboxSize](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 공유 커널 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Hyper-V 격리 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |

## <a name="additional-notes-or-details"></a>추가 메모 및 세부 정보

### <a name="memory"></a>메모리

Windows 컨테이너는 일반적으로 사용자 관리, 네트워킹 등 컨테이너별 기능을 제공하는 각 컨테이너에서 일부 시스템 프로세스를 실행합니다. 이러한 프로세스에서 요구하는 메모리의 대부분은 컨테이너 간에 공유되며 메모리 용량은 이러한 메모리를 허용할 수 있도록 충분히 높아야 합니다.  Hyper-V 격리가 있거나 없는 각 기본 이미지 유형은 [시스템 요구 사항](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/system-requirements#memory-requirments) 문서에서 표로 제공됩니다.

### <a name="cpu-shares-without-hyper-v-isolation"></a>CPU 공유(Hyper-V 격리 없음)

CPU 공유를 사용할 때 기본 구현(Hyper-V 격리 사용하지 않음)은 [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)을 구성하며, 특히 JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED에 컨트롤 플래그를 설정하고 적절한 가중치를 제공합니다.  작업 개체의 유효한 가중치 범위는 1~9이며 기본값은 1~10000의 호스트 컴퓨팅 서비스 값보다 낮은 정확도인 5입니다.  예를 들면 공유 가중치가 7500이면 가중치는 7이고, 공유 가중치가 2500이면 값은 2입니다.
