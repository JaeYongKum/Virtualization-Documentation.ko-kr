---
title: Windows 컨테이너의 GPU 가속
description: Windows 컨테이너의 GPU 가속 수준
keywords: docker, 컨테이너, 디바이스, 하드웨어
author: cwilhit
ms.topic: how-to
ms.openlocfilehash: 11ab6edb58c14c4532d69877533fb575f82d80cd
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192270"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 컨테이너의 GPU 가속

CPU 컴퓨팅 리소스는 여러 컨테이너화된 워크로드에 충분한 성능을 제공합니다. 그러나 특정 워크로드 클래스의 경우 GPU(그래픽 처리 장치)에서 제공하는 대규모 병렬 컴퓨팅 성능으로 작업 속도를 엄청나게 향상하고, 비용을 절감하고, 처리량을 대폭 높일 수 있습니다.

GPU는 전통적인 렌더링과 시뮬레이션부터 기계 학습 및 유추까지 여러 주요 워크로드에 이미 널리 사용되고 있는 도구입니다. Windows 컨테이너는 DirectX 및 DirectX 기반의 모든 프레임워크에서 GPU 가속을 지원합니다.

> [!NOTE]
> 이 기능은 Docker Desktop 버전 2.1 및 Docker 엔진 Enterprise 버전 19.03 이상에서 사용할 수 있습니다.

## <a name="requirements"></a>요구 사항

이 기능이 작동하려면 환경이 다음 요구 사항을 충족해야 합니다.

- 컨테이너 호스트에서 Windows Server 2019 또는 Windows 10 버전 1809 이상을 실행해야 합니다.
- 컨테이너 기본 이미지가 [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windows) 이상이어야 합니다. Windows Server Core 및 Nano Server 컨테이너 이미지는 현재 지원되지 않습니다.
- 컨테이너 호스트에서 Docker 엔진 19.03 이상 버전을 실행해야 합니다.
- 디스플레이 드라이버 버전 WDDM 2.5 이상을 실행하는 GPU가 컨테이너 호스트에 있어야 합니다.

디스플레이 드라이버의 WDDM 버전을 확인하려면 컨테이너 호스트에서 DirectX 진단 도구(dxdiag.exe)를 실행합니다. 도구의 “디스플레이” 탭에서 아래에 나와 있는 "드라이버" 섹션을 찾습니다.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>GPU 가속을 사용하여 컨테이너 실행

GPU 가속으로 컨테이너를 시작하려면 다음 명령을 실행합니다.

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX(및 DirectX 기반의 모든 프레임워크)는 현재 GPU로 가속화할 수 있는 유일한 API입니다. 타사 프레임워크는 지원되지 않습니다.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V 격리 Windows 컨테이너 지원

Hyper-V 격리 Windows 컨테이너의 워크로드에 대한 GPU 가속은 현재 지원되지 않습니다.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V 격리 Linux 컨테이너 지원

Hyper-V 격리 Linux 컨테이너의 워크로드에 대한 GPU 가속은 현재 지원되지 않습니다.

## <a name="more-information"></a>자세한 정보

GPU 가속을 활용하는 컨테이너화된 DirectX 앱의 완전한 예제는 [DirectX 컨테이너 샘플](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx)을 참조하세요.
