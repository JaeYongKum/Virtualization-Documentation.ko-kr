---
title: Windows 10의 Hyper-V 소개
description: Hyper-V, 가상화 및 관련 기술을 소개합니다.
keywords: windows 10, hyper-v
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: 78991d0b6d8b27ea20365fed74f35cee64eb089f
ms.sourcegitcommit: 64c8d5d6f068d385b94db4637259bb3852666efe
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/23/2018
ms.locfileid: "1797679"
---
# <a name="introduction-to-hyper-v-on-windows-10"></a>Windows 10의 Hyper-V 소개

> Hyper-V는 Microsoft Virtual PC를 대체합니다.

소프트웨어 개발자, IT 전문가, 기술 매니아 등 여러 운영 체제를 실행해야 하는 사용자가 많습니다. Hyper-V를 사용하면 Windows에서 가상 컴퓨터로 여러 운영 체제를 실행할 수 있습니다.

![Windows를 실행 중인 가상 컴퓨터](media/HyperVNesting.png)

Hyper-V는 특히 하드웨어 가상화를 제공합니다.  즉, 각 가상 컴퓨터가 가상 하드웨어에서 실행됩니다.  Hyper-V를 통해 가상 하드 드라이브, 가상 스위치 및 가상 컴퓨터에 추가할 수 있는 각종 가상 디바이스를 만들 수 있습니다.

## <a name="reasons-to-use-virtualization"></a>가상화를 사용하는 이유

가상화를 사용하면 다음과 같은 일을 할 수 있습니다.

* 이전 버전의 Windows 또는 Windows가 아닌 다른 운영 체제가 필요한 소프트웨어를 실행합니다.

* 다른 운영 체제를 시험해 봅니다. Hyper-V를 사용하면 다른 운영 체제를 아주 쉽게 만들고 제거할 수 있습니다.

* 여러 가상 컴퓨터를 사용하여 여러 운영 체제에서 소프트웨어를 테스트합니다. Hyper-V를 사용하면 모든 가상 컴퓨터를 하나의 데스크톱 또는 노트북 컴퓨터에서 실행할 수 있습니다. 이러한 가상 컴퓨터를 내보낸 다음 Azure를 비롯한 다른 Hyper-V 시스템으로 가져올 수 있습니다.

## <a name="system-requirements"></a>시스템 요구 사항

Hyper-V는 Windows 8 이상의 64비트 Windows Professional, Enterprise 및 Education 버전에서 사용할 수 있습니다.  Windows Home 버전에서는 사용할 수 없습니다.

> **설정** > **업데이트 및 보안** > **정품 인증**을 열어 Windows 10 Home 버전을 Windows 10 Professional로 업그레이드하세요. 여기에서 스토어를 방문하여 업그레이드를 구입할 수 있습니다.

대부분의 컴퓨터는 Hyper-V를 실행하지만, 각 가상 컴퓨터는 완전히 별도 운영 체제를 기반으로 합니다.  일반적으로 4GB RAM이 설치된 컴퓨터에서 가상 컴퓨터를 하나 이상 실행할 수 있지만 가상 컴퓨터를 추가하거나 게임, 동영상 편집, 디자인 소프트웨어 엔지니어링처럼 리소스를 많이 사용하는 소프트웨어를 설치하고 실행하려면 더 많은 리소스가 필요합니다.

Hyper-V 시스템 요구 사항 및 Hyper-V가 컴퓨터에서 실행되는지 확인하는 방법에 대한 자세한 내용은 [Hyper-V 요구 사항 참조](..\reference\hyper-v-requirements.md)을 살펴보세요.

## <a name="operating-systems-you-can-run-in-a-virtual-machine"></a>가상 컴퓨터에서 실행할 수 있는 운영 체제

Windows의 Hyper-V는 다양한 버전의 Linux, FreeBSD 및 Windows를 포함하여 가상 컴퓨터에서 여러 운영 체제를 지원합니다.

참고로 VM에서 사용하는 모든 운영 체제에는 유효한 라이선스가 필요합니다.

Windows의 Hyper-V에서 게스트로 지원되는 운영 체제에 대한 자세한 내용은 [지원되는 Windows 게스트 운영 체제](supported-guest-os.md) 및 [지원되는 Linux 게스트 운영 체제](https://technet.microsoft.com/library/dn531030.aspx)를 참조하세요.

## <a name="differences-between-hyper-v-on-windows-and-hyper-v-on-windows-server"></a>Windows의 Hyper-V와 Windows Server의 Hyper-V 간 차이점

Windows에서 Hyper-V를 실행할 때 몇 가지 기능은 Windows Server에서 Hyper-V를 실행할 때와는 다르게 작동합니다.

Windows Server에서만 사용할 수 있는 Hyper-V 기능:

* RemoteFX를 사용하여 GPU 가상화
* 가상 컴퓨터를 실시간으로 한 호스트에서 다른 호스트로 마이그레이션
* Hyper-V 복제본
* 가상 파이버 채널
* SR-IOV 네트워킹
* 공유 .VHDX

Windows 10에서만 사용할 수 있는 Hyper-V 기능:

* 빨리 만들기 및 VM 갤러리
* 기본 네트워크(NAT 스위치)

메모리 관리 모델이 Windows의 Hyper-V에서 다릅니다. 서버에서 Hyper-V 메모리는 해당 서버에서 관리 컴퓨터만 실행된다는 가정 하에 관리됩니다. Windows의 Hyper-V에서 메모리는 대부분의 클라이언트 컴퓨터가 가상 컴퓨터 실행 외에도 호스트의 소프트웨어를 실행한다는 예상에 따라 관리됩니다.

## <a name="limitations"></a>제한 사항

특정 하드웨어에 종속된 프로그램은 가상 컴퓨터에서 제대로 작동하지 않습니다. 예를 들어 GPU를 통한 처리가 필요한 게임이나 응용 프로그램은 제대로 작동하지 않을 수 있습니다. 또한 라이브 뮤직 믹싱 응용 프로그램이나 고정밀 시간과 같이 10밀리초 미만의 타이머를 사용하는 응용 프로그램은 가상 컴퓨터에서 실행될 때 문제가 있을 수 있습니다.

또한 Hyper-V를 사용하도록 설정한 경우 대기 시간에 민감한 고정밀 응용 프로그램은 호스트에서 실행될 때도 문제가 있을 수 있습니다.  이는 가상화를 사용하도록 설정하면 호스트 OS도 게스트 운영 체제처럼 Hyper-V 가상화 계층 위에서 실행되기 때문입니다. 그러나 게스트와 달리 호스트 OS는 모든 하드웨어에 직접 액세스할 수 있다는 점에서 특별합니다. 즉, 하드웨어 요구 사항이 특별한 응용 프로그램도 호스트 OS에서 문제 없이 실행될 수 있다는 의미입니다.

## <a name="next-step"></a>다음 단계

[Windows 10에 Hyper-V 설치](..\quick-start\enable-hyper-v.md)
