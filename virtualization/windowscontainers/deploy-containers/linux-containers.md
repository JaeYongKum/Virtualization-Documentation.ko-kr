---
title: Windows 10의 Linux 컨테이너
description: Hyper-V를 사용하여 마치 네이티브 컨테이너인 것처럼 Windows 10에서 Linux 컨테이너를 실행하는 여러 가지 방법을 알아봅니다.
keywords: linux 컨테이너, docker, 컨테이너, windows 10
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 6b737129692ec8e56bebf290ad8064f010f78ea3
ms.sourcegitcommit: 6a5c237bff2c953fec2ce1e09424375a7c615010
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/09/2020
ms.locfileid: "84632973"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10의 Linux 컨테이너

Linux 컨테이너는 전체 컨테이너 에코 시스템에서 매우 큰 비중을 차지하며 개발자 환경과 프로덕션 환경 모두에 필수적입니다.  컨테이너는 컨테이너 호스트와 커널을 공유하지만, Linux 컨테이너를 Windows에서 직접 실행할 수는 없습니다. 여기서 가상화가 등장합니다.

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM의 Linux 컨테이너

Linux VM에서 Linux 컨테이너를 실행하려면 [Docker 시작 가이드](https://docs.docker.com/docker-for-windows/)를 따르세요.

Docker는 Hyper-V에서 실행되는 [LinuxKit](https://github.com/linuxkit/linuxkit) 기반 가상 머신을 사용하여 2016년(Hyper-V 격리 또는 Windows의 Linux 컨테이너를 사용할 수 있게 되기 전)에 처음 릴리스될 때부터 Windows 데스크톱에서 Linux 컨테이너를 실행할 수 있었습니다.

이 모델에서는 Docker 클라이언트가 Windows 데스크톱에서 실행되지만, Linux VM에서 Docker 디먼을 호출합니다.

![컨테이너 호스트로 실행되는 Moby VM](media/MobyVM.png)

이 모델에서는 모든 Linux 컨테이너가 단일 Linux 기반 컨테이너 호스트를 공유하고, 모든 Linux 컨테이너가 다음과 같이 작동합니다.

* 서로 그리고 Moby VM과 커널을 공유하지만 Windows 호스트와는 공유하지 않습니다.
* Linux에서 실행되는 Linux 컨테이너와 일치하는 스토리지 및 네트워킹 속성을 갖습니다(Linux VM에서 실행되므로).

또한 Linux 컨테이너 호스트(Moby VM)는 Docker 디먼 및 모든 Docker 디먼 종속 요소를 실행해야 합니다.

Moby VM으로 실행 중인지 확인하려면 Hyper-V Manager UI를 사용하거나 관리자 권한 PowerShell 창에서 `Get-VM`을 실행하여 Moby VM용 Hyper-V Manager를 확인합니다.
