---
title: 지원되는 Windows 게스트
description: 지원되는 Windows 게스트입니다.
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: e3255d236a3fbb5ac4d908143750b84e3db82ceb
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621101"
---
# <a name="supported-windows-guests"></a>지원되는 Windows 게스트

이 문서는 Windows의 Hyper-V에서 지원되는 운영 체제 조합을 나열합니다.  지원되는 통합 서비스 및 기타 요인에 대해서도 소개합니다.

Microsoft는 이러한 호스트/게스트 조합을 테스트했습니다.  이러한 조합과 관련된 문제는 제품 지원 서비스의 도움을 받을 수 있습니다.

Microsoft는 다음과 같은 방식으로 지원을 제공합니다.

* Microsoft 운영 체제 및 통합 서비스에서 발견된 문제는 Microsoft 지원에서 지원됩니다.

* Hyper-V에서 실행할 운영 체제 공급업체의 인증을 받은 다른 운영 체제에서 발견된 문제는 해당 공급업체가 지원 서비스를 제공합니다.

* 기타 운영 체제에서 발견된 문제에 대해 Microsoft는 다중 공급업체 지원 커뮤니티인 [TSANet](http://www.tsanet.org/)에 문제를 제출합니다.

지원을 받으려면 모든 운영 체제(게스트와 호스트)를 업데이트해야 합니다.  Windows 업데이트의 중요 업데이트를 확인합니다.

## <a name="supported-guest-operating-systems"></a>지원되는 게스트 운영 체제

| 게스트 운영 체제 |  최대 가상 프로세서 수 | 참고 |
|:-----|:-----|:-----|
| Windows 10 | 32 |고급 세션 모드는 Windows 10 Home 버전에서 작동하지 않음 |
| Windows8.1 | 32 | |
| Windows8 | 32 ||
| Windows 7 SP 1(서비스 팩 1) | 4 | Ultimate/Enterprise/Professional Edition(32비트/64비트) |
| Windows 7 | 4 | Ultimate/Enterprise/Professional Edition(32비트/64비트) |
| Windows Vista SP2(서비스 팩 2) | 2 | Business/Enterprise/Ultimate(N 및 KN 버전 포함) |
| - | | |
| [Windows Server 반기 채널](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| WindowsServer 2019 | 64 | |
| WindowsServer 2016 | 64 | |
| WindowsServer 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 SP 1(서비스 팩 1) | 64 | Datacenter/Enterprise/Standard/Web Edition. |
| Windows Server 2008 SP 2(서비스 팩 2) | 4 | Datacenter/Enterprise/Standard/Web Edition(32비트/64비트). |
| Windows Home Server2011 | 4 | |
| Windows Small Business Server 2011 | Essentials edition - 2, Standard edition - 4 | |

> Windows 10은 Windows 8.1 및 Windows Server 2012 R2 Hyper-V 호스트에서 게스트 운영 체제로 실행할 수 있습니다.

## <a name="supported-linux-and-free-bsd"></a>지원되는 Linux 및 무료 BSD

| 게스트 운영 체제 |  |
|:-----|:------|
| [CentOS 및 Red Hat Enterprise Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-CentOS-and-Red-Hat-Enterprise-Linux-virtual-machines-on-Hyper-V) | |
| [Hyper-V의 Debian 가상 컴퓨터](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Debian-virtual-machines-on-Hyper-V) | |
| [SUSE](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-SUSE-virtual-machines-on-Hyper-V) | |
| [Oracle Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Oracle-Linux-virtual-machines-on-Hyper-V)  | |
| [Ubuntu](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Ubuntu-virtual-machines-on-Hyper-V) | |
| [FreeBSD](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-FreeBSD-virtual-machines-on-Hyper-V) | |

이전 버전의 Hyper-V에 대한 지원 정보를 포함한 자세한 내용은 [Hyper-V의 Linux 및 FreeBSD 가상 컴퓨터](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows)를 참조하세요.
