---
title: Windows 컨테이너에서 GPU 가속
description: Windows 컨테이너에 있는 GPU 가속 수준
keywords: docker, 컨테이너, 장치, 하드웨어
author: cwilhit
ms.openlocfilehash: 281241e07e4bc184e73c4e74a117b44253a775be
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380057"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 컨테이너에서 GPU 가속

많은 컨테이너 화 된 워크 로드에 대 한 CPU 계산 리소스 충분 한 성능을 제공합니다. 그러나 워크 로드의 특정 클래스에 대 한 Gpu (그래픽 처리 단위)에서 제공에서 대규모 병렬 계산 전원 속도를 높일 수 작업의 크기가, 비용 다운 하 고 매우 처리량 향상.

Gpu에서 기존의 렌더링 및 시뮬레이션 컴퓨터 학습 교육 및 유추에 많은 인기 있는 워크 로드에 대 한 일반적인 도구 이미 있습니다. Windows 컨테이너는 DirectX 및이 기반으로 모든 프레임 워크에 대 한 GPU 가속을 지원 합니다.

> [!IMPORTANT]
> 이 기능을 지 원하는 Docker 버전이 필요 합니다 `--device` Windows 컨테이너에 대 한 명령줄 옵션입니다. 공식 Docker 지원 예정 된 다음 Docker EE 엔진 19.03 릴리스에 대 한 합니다. 그 때까지 Docker에 대 한 [업스트림 소스](https://master.dockerproject.org/) 에 필요한 비트 포함 되어 있습니다.

## <a name="requirements"></a>요구 사항

이 기능이 제대로 작동 하도록 환경에는 다음 요구 사항을 충족 해야 합니다.

- Windows 10, 버전 1809 이상 또는 Windows Server 2019 컨테이너 호스트를 실행 해야 합니다.
- 컨테이너 기본 이미지 [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows) 이어야 합니다. 또는 최신 합니다. Windows Server Core와 Nano 서버 컨테이너 이미지는 현재 지원 되지 않습니다.
- 컨테이너 호스트 19.03 또는 최신 Docker 엔진을 실행 되어야 합니다.
- 컨테이너 호스트 GPU 실행 디스플레이 드라이버 버전 WDDM 2.5 이상 있어야 합니다.

디스플레이 드라이버 WDDM 버전을 확인 하려면 컨테이너 호스트에 DirectX 진단 도구 (dxdiag.exe)를 실행 합니다. 도구의 "디스플레이" 탭에서 아래와 같이 "드라이버" 섹션을 찾습니다.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>GPU 가속을 사용 하 여 컨테이너를 실행 합니다.

GPU 가속을 사용 하 여 컨테이너를 시작 하려면 다음 명령을 실행 합니다.

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (및이 기반으로 모든 프레임 워크)는 GPU 사용 하 여 가속 될 수 있는 유일한 Api를 현재는 합니다. 타사 프레임 워크가 지원 되지 않습니다.

## <a name="hyper-v-isolated-windows-container-support"></a>V-격리 하이퍼 Windows 컨테이너 지원

Windows 하이퍼 V 격리 된 컨테이너에서 워크 로드에 대 한 GPU 가속 현재 지원 되지 않습니다.

## <a name="hyper-v-isolated-linux-container-support"></a>V-격리 하이퍼 Linux 컨테이너 지원

Linux 하이퍼 V 격리 된 컨테이너에서 워크 로드에 대 한 GPU 가속 현재 지원 되지 않습니다.
