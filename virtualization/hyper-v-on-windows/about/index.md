---
title: "Windows 10의 Hyper-V 소개"
description: "Hyper-V, 가상화 및 관련 기술을 소개합니다."
keywords: windows 10, hyper-v
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: 918fe27f7aee74e3c1de3b7381e7ccea76f49c73
ms.sourcegitcommit: d5f30aa1bdfb34dd9e1909d73b5bd9f4153d6b46
ms.translationtype: HT
ms.contentlocale: ko-KR
---
# <a name="introduction-to-hyper-v-on-windows-10"></a>Windows 10의 Hyper-V 소개

> Hyper-V는 Microsoft Virtual PC를 대체합니다. 

많은 소프트웨어 개발자, IT 전문가 또는 기술 매니아들은 여러 운영 체제를 실행해야 합니다. Hyper-V를 사용하면 각 컴퓨터에 물리적 하드웨어를 개별적으로 지정하는 대신 운영 체제 또는 컴퓨터 시스템을 Windows에서 가상 컴퓨터로 실행할 수 있습니다.  

![](media/HyperVNesting.png)

Hyper-V는 특히 하드웨어 가상화를 제공합니다.  즉, 각 가상 컴퓨터가 가상 하드웨어에서 실행됩니다.  Hyper-V를 통해 가상 하드 드라이브, 가상 스위치 및 가상 컴퓨터에 추가할 수 있는 각종 가상 디바이스를 만들 수 있습니다.

## <a name="reasons-to-use-virtualization"></a>가상화를 사용하는 이유

가상화를 사용하면 다음과 같은 일을 할 수 있습니다.  
* 이전 버전의 Windows 또는 Windows가 아닌 다른 운영 체제가 필요한 소프트웨어를 실행합니다. 

* 다른 운영 체제를 시험해 봅니다. Hyper-V를 사용하면 다른 운영 체제를 아주 쉽게 만들고 제거할 수 있습니다.

* 여러 가상 컴퓨터를 사용하여 여러 운영 체제에서 소프트웨어를 테스트합니다. Hyper-V를 사용하면 모든 가상 컴퓨터를 하나의 데스크톱 또는 노트북 컴퓨터에서 실행할 수 있습니다. 이러한 가상 컴퓨터를 내보낸 다음 Azure를 비롯한 다른 Hyper-V 시스템으로 가져올 수 있습니다.

* 어느 Hyper-V 배포에서나 가상 컴퓨터 문제를 해결합니다. 프로덕션 환경에서 가상 컴퓨터를 내보내고 Hyper-V를 실행하는 데스크톱에서 열고 가상 컴퓨터 문제를 해결한 다음 프로덕션 환경으로 다시 내보낼 수 있습니다. 

* 가상 네트워킹을 사용하여 프로덕션 네트워크에는 영향을 미치지 않도록 하면서 테스트/개발/데모를 위한 다중 컴퓨터 환경을 만들 수 있습니다.

## <a name="system-requirements"></a>시스템 요구 사항
Hyper-V는 Windows 8 이상의 64비트 Windows Professional, Enterprise 및 Education 버전에서 사용할 수 있습니다.  Windows Home 버전에서는 사용할 수 없습니다.  

>  **설정** > **업데이트 및 보안** > **정품 인증**을 열어 Windows 10 Home 버전을 Windows 10 Professional로 업그레이드하세요. 여기서 스토어를 방문하고 업그레이드를 구입할 수 있습니다.

대부분의 컴퓨터에서 Hyper-V를 실행하게 되겠지만 가상 컴퓨터는 전체 운영 체제를 실행하므로 상당히 많은 리소스가 필요합니다.  일반적으로 4GB RAM이 설치된 컴퓨터에서 가상 컴퓨터를 하나 이상 실행할 수 있지만 가상 컴퓨터를 추가하거나 게임, 동영상 편집, 디자인 소프트웨어 엔지니어링처럼 리소스를 많이 사용하는 소프트웨어를 설치하고 실행하려면 더 많은 리소스가 필요합니다. 

컴퓨터에 SLAT(두 번째 수준 주소 변환)가 필요하며, SLAT는 Intel 및 AMD의 최신 세대 64비트 프로세서에서 제공합니다.  64비트 버전의 Windows도 필요합니다.

Hyper-V 시스템 요구 사항 및 Hyper-V가 컴퓨터에서 실행되는지 확인하는 방법에 대한 자세한 내용은 [Hyper-V 요구 사항 참조](..\reference\hyper-v-requirements.md)을 살펴보세요.

## <a name="operating-systems-you-can-run-in-a-virtual-machine"></a>가상 컴퓨터에서 실행할 수 있는 운영 체제
"게스트"라는 말은 가상 컴퓨터를, "호스트"는 가상 컴퓨터를 실행하는 컴퓨터를 의미합니다. Windows의 Hyper-V는 다양한 Linux, FreeBSD 및 Windows 버전을 포함하여 여러 게스트 운영 체제를 지원합니다. 

참고로 VM에서 사용하는 모든 운영 체제에는 유효한 라이선스가 필요합니다. 

Windows의 Hyper-V에서 게스트로 지원되는 운영 체제에 대한 자세한 내용은 [지원되는 Windows 게스트 운영 체제](supported-guest-os.md) 및 [지원되는 Linux 게스트 운영 체제](https://technet.microsoft.com/library/dn531030.aspx)를 참조하세요. 


## <a name="differences-between-hyper-v-on-windows-and-hyper-v-on-windows-server"></a>Windows의 Hyper-V와 Windows Server의 Hyper-V 간 차이점
Windows에서 Hyper-V를 실행할 때 몇 가지 기능은 Windows Server에서 Hyper-V를 실행할 때와는 다르게 작동합니다. 

메모리 관리 모델이 Windows의 Hyper-V에서 다릅니다. 서버에서 Hyper-V 메모리는 해당 서버에서 관리 컴퓨터만 실행된다는 가정 하에 관리됩니다. Windows의 Hyper-V에서 메모리는 대부분의 클라이언트 컴퓨터가 가상 컴퓨터 실행 외에도 호스트의 소프트웨어를 실행한다는 예상에 따라 관리됩니다. 예를 들어 개발자가 동일한 컴퓨터에서 여러 가상 컴퓨터와 Visual Studio를 실행할 수 있습니다.

Windows Server의 Hyper-V에는 Windows의 Hyper-V에 없는 기능이 몇 가지 포함되어 있습니다. 확인할 수 있습니다.

* RemoteFX를 사용하여 GPU 가상화 
* 가상 컴퓨터를 실시간으로 한 호스트에서 다른 호스트로 마이그레이션
* Hyper-V 복제본
* 가상 파이버 채널
* SR-IOV 네트워킹
* 공유 .VHDX

## <a name="limitations"></a>제한 사항
가상화 사용에는 제한이 있습니다. 특정 하드웨어에 종속된 기능이나 응용 프로그램은 가상 컴퓨터에서 제대로 작동하지 않습니다. 예를 들어 GPU를 통한 처리가 필요한 게임이나 응용 프로그램은 제대로 작동하지 않을 수 있습니다. 또한 라이브 뮤직 믹싱 응용 프로그램이나 고정밀 시간과 같이 10밀리초 미만의 타이머를 사용하는 응용 프로그램은 가상 컴퓨터에서 실행될 때 문제가 있을 수 있습니다.

또한 Hyper-V를 사용하도록 설정한 경우 대기 시간에 민감한 고정밀 응용 프로그램은 호스트에서 실행될 때도 문제가 있을 수 있습니다.  이는 가상화를 사용하도록 설정하면 호스트 OS도 게스트 운영 체제처럼 Hyper-V 가상화 계층 위에서 실행되기 때문입니다. 그러나 게스트와 달리 호스트 OS는 모든 하드웨어에 직접 액세스할 수 있다는 점에서 특별합니다. 즉, 하드웨어 요구 사항이 특별한 응용 프로그램도 호스트 OS에서 문제 없이 실행될 수 있다는 의미입니다.

## <a name="next-step"></a>다음 단계
[Windows 10에 Hyper-V 설치](..\quick-start\enable-hyper-v.md) 
