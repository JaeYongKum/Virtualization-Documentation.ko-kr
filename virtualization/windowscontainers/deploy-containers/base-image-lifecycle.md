---
title: 기본 이미지 서비스 수명 주기
description: Windows 컨테이너 기본 이미지 수명 주기에 대한 정보입니다.
keywords: windows 컨테이너, 컨테이너, 수명 주기, 릴리스 정보, 기본 이미지, 컨테이너 기본 이미지
author: Heidilohr
ms.author: helohr
ms.date: 05/12/2020
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: c6276db89f093b62a01cadc095f5357d2e5a8eba
ms.sourcegitcommit: dd80813679df2de89fe523a26600cfe58a2d39a2
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/27/2020
ms.locfileid: "84023148"
---
# <a name="base-image-servicing-lifecycles"></a>기본 이미지 서비스 수명 주기

> [!Note]  
> Microsoft는 사용자와 조직이 비즈니스 연속성을 유지하는 데 집중할 수 있도록 도와주기 위해 여러 제품의 예정된 지원 및 서비스 종료 날짜를 연기했습니다. 자세한 내용은 2020년 4월 14일부터 [지원 종료 및 서비스 날짜에 대한 수명 주기 변경](https://support.microsoft.com/en-us/help/4557164/lifecycle-changes-to-end-of-support-and-servicing-dates) 항목을 참조하세요.

Windows 컨테이너 기본 이미지는 Windows Server의 반기 채널 릴리스 또는 장기 서비스 채널 릴리스를 기반으로 합니다. 이 문서에서는 두 채널의 여러 기본 이미지 버전에 대한 지원이 얼마나 지속되는지 알려줍니다.

반기 채널은 1년에 두 차례 기능 업데이트를 릴리스하며, 각 릴리스의 서비스 타임라인은 18개월입니다. 따라서 고객은 애플리케이션(특히 컨테이너 및 마이크로서비스 기반의 애플리케이션)과 소프트웨어 정의 하이브리드 데이터 센터에서 새로운 운영 체제 기능을 더 빠르게 활용할 수 있습니다. 자세한 내용은 [Windows Server 반기 채널 개요](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)를 참조하세요.

Server Core 이미지의 경우 고객은 2~3년마다 Windows Server의 새로운 주 버전을 출시하는 장기 서비스 채널을 사용할 수도 있습니다. 장기 서비스 채널 릴리스에는 5년의 일반 지원과 5년의 연장 지원이 제공됩니다. 이 채널은 보다 장기적인 서비스 옵션과 기능적 안정성이 필요한 시스템에서 작동합니다.

다음 표에는 기본 이미지의 유형, 서비스 채널 및 지원 기간이 나열되어 있습니다.

|Base image                       |서비스 채널|Version|OS 빌드|가용성|일반 지원 종료 날짜|연장 지원 날짜|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, Nano Server, Windows|반기      |2004   |19041   |2020/05/27  |2021/12/14                 |해당 없음                  |
|Server Core, Nano Server, Windows|반기      |1909   |18363   |2019/11/12  |2021/05/11                 |해당 없음                  |
|Server Core, Nano Server, Windows|반기      |1903   |18362   |2019/05/21  |2020/12/08                 |해당 없음                  |
|Server Core                      |장기        |2019   |17763   |2018/11/13  |2024/01/09                 |2029/01/09           |
|Server Core, Nano Server, Windows|반기      |1809   |17763   |2018/11/13  |2020/11/10                 |해당 없음                  |
|Server Core, Nano Server         |반기      |1803   |17134   |2018/04/30  |2019/11/12                 |해당 없음                  |
|Server Core, Nano Server         |반기      |1709   |16299   |2017/10/17  |2019/04/09                 |해당 없음                  |
|Server Core                      |장기        |2016   |14393   |2016/10/15  |2022/01/11                 |2027/01/11           |
|Nano 서버                      |반기      |1607   |14393   |2016/10/15  |2018/10/09                 |해당 없음                  |

서비스 요구 사항 및 기타 추가 정보는 [Windows 수명 주기 FAQ](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products), [Windows Server 릴리스 정보](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info) 및 [Windows 기본 OS 이미지 Docker 허브 리포지토리](https://hub.docker.com/_/microsoft-windows-base-os-images)를 참조하세요.
