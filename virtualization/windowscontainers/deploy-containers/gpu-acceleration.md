---
title: Windows 컨테이너의 GPU 가속
description: Windows 컨테이너에 있는 GPU 가속 수준
keywords: docker, 컨테이너, 장치, 하드웨어
author: cwilhit
ms.openlocfilehash: c6746b45caece9802134831eb6cb3da885957ac5
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/20/2019
ms.locfileid: "10045043"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 컨테이너의 GPU 가속

많은 containerized 작업 부하에 대해 CPU 계산 리소스는 충분 한 성능을 제공 합니다. 그러나 특정 작업 부하의 경우 Gpu (그래픽 처리 단위)가 제공 하는 대규모 병렬 연산 전원을 통해 작업 속도를 높여 비용을 계산 하 고 처리량을 줄일 수 있습니다.

Gpu는 기존 렌더링 및 시뮬레이션에서 기계 학습 교육 및 유추에 이르기까지 인기 작업 부하에 대 한 일반적인 도구입니다. Windows 컨테이너는 DirectX 및 그 위에 빌드된 모든 프레임 워크에 대 한 GPU 가속을 지원 합니다.

> [!NOTE]
> 이 기능은 Docker 데스크탑, 버전 2.1 및 Docker 엔진-Enterprise 버전 19.03 이상에서 사용할 수 있습니다.

## <a name="requirements"></a>요구 사항

이 기능이 작동 하려면 환경이 다음 요구 사항을 충족 해야 합니다.

- 컨테이너 호스트는 Windows Server 2019 또는 Windows 10, 버전 1809 이상을 실행 중 이어야 합니다.
- 컨테이너 기본 이미지는 [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows) 또는 최신 이어야 합니다. Windows Server Core 및 Nano 서버 컨테이너 이미지는 현재 지원 되지 않습니다.
- 컨테이너 호스트는 Docker 엔진 19.03 이상 버전을 실행 중 이어야 합니다.
- 컨테이너 호스트에는 GPU가 실행 중인 디스플레이 드라이버 버전 WDDM 2.5 이상이 있어야 합니다.

디스플레이 드라이버의 WDDM 버전을 확인 하려면 컨테이너 호스트에서 DirectX 진단 도구 (dxdiag)를 실행 합니다. 도구의 "표시" 탭에서 아래에 표시 된 "드라이버" 섹션을 확인 하세요.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>GPU 가속을 사용 하 여 컨테이너 실행

GPU 가속을 사용 하 여 컨테이너를 시작 하려면 다음 명령을 실행 합니다.

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (및 그 위에 빌드된 모든 프레임 워크)는 현재 GPU를 사용 하 여 가속화 될 수 있는 유일한 Api입니다. 타사 프레임 워크는 지원 되지 않습니다.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-격리 된 Windows 컨테이너 지원

Hyper-v-격리 된 Windows 컨테이너의 작업 부하에 대 한 GPU 가속은 현재 지원 되지 않습니다.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-isolated Linux 컨테이너 지원

Hyper-v-isolated Linux 컨테이너의 작업 부하에 대 한 GPU 가속은 현재 지원 되지 않습니다.

## <a name="more-information"></a>자세한 정보

GPU 가속을 활용 하는 containerized DirectX 앱의 전체 예제는 [DirectX 컨테이너 샘플](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx)을 참조 하세요.
