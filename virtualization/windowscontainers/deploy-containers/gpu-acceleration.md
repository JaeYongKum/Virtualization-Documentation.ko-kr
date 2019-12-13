---
title: Windows 컨테이너의 GPU 가속
description: Windows 컨테이너에 있는 GPU 가속 수준
keywords: docker, 컨테이너, 장치, 하드웨어
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909913"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 컨테이너의 GPU 가속

많은 컨테이너 화 된 워크 로드의 경우 CPU 계산 리소스는 충분 한 성능을 제공 합니다. 그러나 특정 워크 로드 클래스의 경우 Gpu (그래픽 처리 장치)에서 제공 하는 대규모 병렬 계산 능력은 규모의 주문 별로 작업 속도를 높이고 비용을 절감 하 고 처리량을 상당히 높일 수 있습니다.

Gpu는 기존 렌더링과 시뮬레이션부터 기계 학습 학습 및 유추까지 널리 사용 되는 다양 한 워크 로드에 대 한 공통 도구입니다. Windows 컨테이너는 DirectX에 대 한 GPU 가속 및 그 위에 빌드되는 모든 프레임 워크를 지원 합니다.

> [!NOTE]
> 이 기능은 Docker Desktop, 버전 2.1 및 Docker 엔진-Enterprise, 버전 19.03 이상에서 사용할 수 있습니다.

## <a name="requirements"></a>요구 사항

이 기능이 작동 하려면 환경이 다음 요구 사항을 충족 해야 합니다.

- 컨테이너 호스트에서 Windows Server 2019 또는 Windows 10 버전 1809 이상이 실행 되 고 있어야 합니다.
- 컨테이너 기본 이미지는 [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windows) 이상 이어야 합니다. Windows Server Core 및 Nano Server 컨테이너 이미지는 현재 지원 되지 않습니다.
- 컨테이너 호스트에서 Docker 엔진 19.03 이상 버전을 실행 해야 합니다.
- 컨테이너 호스트에는 GPU를 실행 하는 디스플레이 드라이버 버전 WDDM 2.5 이상이 있어야 합니다.

디스플레이 드라이버의 WDDM 버전을 확인 하려면 컨테이너 호스트에서 DirectX 진단 도구 (dxdiag)를 실행 합니다. 도구의 "디스플레이" 탭에서 아래에 나와 있는 "드라이버" 섹션을 찾습니다.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>GPU 가속을 사용 하 여 컨테이너 실행

GPU 가속으로 컨테이너를 시작 하려면 다음 명령을 실행 합니다.

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (및 그 위에 빌드된 모든 프레임 워크)는 현재 GPU로 가속화 될 수 있는 유일한 Api입니다. 타사 프레임 워크는 지원 되지 않습니다.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v 격리 Windows 컨테이너 지원

Hyper-v 격리 Windows 컨테이너의 워크 로드에 대 한 GPU 가속은 현재 지원 되지 않습니다.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v 격리 Linux 컨테이너 지원

Hyper-v 격리 Linux 컨테이너의 워크 로드에 대 한 GPU 가속은 현재 지원 되지 않습니다.

## <a name="more-information"></a>추가 정보

GPU 가속을 활용 하는 컨테이너 화 된 DirectX 앱에 대 한 전체 예제는 [directx 컨테이너 샘플](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx)을 참조 하세요.
