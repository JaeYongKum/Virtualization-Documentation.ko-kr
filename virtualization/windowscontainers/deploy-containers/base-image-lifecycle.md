---
title: 기본 이미지 서비스 수명 주기
description: Windows 컨테이너 기본 이미지 수명 주기에 대 한 정보입니다.
keywords: windows 컨테이너, 컨테이너, 수명 주기, 릴리스 정보, 기본 이미지, 컨테이너 기본 이미지
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 2dcd228af0984b55162894555fa21f9e02dd1934
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/15/2020
ms.locfileid: "81395746"
---
# <a name="base-image-servicing-lifecycles"></a>기본 이미지 서비스 수명 주기

> [!Note]  
> Microsoft는 사용자와 조직이 비즈니스 연속성을 유지 하는 데 집중 하는 데 도움이 되는 다양 한 제품에 대 한 지원 및 서비스 날짜의 예약 된 종료를 지연 시켰습니다. 자세한 내용은 2020 년 4 월 14 일의 [지원 종료 및 서비스 날짜 항목의 수명 주기 변경을](https://support.microsoft.com/en-us/help/4557164/lifecycle-changes-to-end-of-support-and-servicing-dates) 참조 하세요.

Windows 컨테이너 기본 이미지는 Windows Server의 반기 채널 릴리스 또는 장기 서비스 채널 릴리스를 기반으로 합니다. 이 문서에서는 두 채널의 기본 이미지에 대 한 다양 한 버전에 대 한 지원이 얼마나 지속 될 지를 알려 줍니다.

반기 채널은 각 릴리스에 대 한 18 월간 서비스 타임 라인이 포함 된 두 번의 연간 기능 업데이트 릴리스입니다. 이렇게 하면 고객이 응용 프로그램 (특히 컨테이너 및 마이크로 서비스를 기반으로 하는 응용 프로그램) 및 소프트웨어 정의 하이브리드 데이터 센터에서 새 운영 체제 기능을 더 빠르게 활용할 수 있습니다. 자세한 내용은 [Windows Server 반기 채널 개요](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)를 참조 하세요.

Server Core 이미지의 경우 고객은 Windows Server의 새로운 주 버전을 2 ~ 3 년 마다 출시 하는 장기 서비스 채널을 사용할 수도 있습니다. 장기 서비스 채널 릴리스는 5 년간의 기본 지원과 5 년의 추가 지원 기능을 받습니다. 이 채널은 더 긴 서비스 옵션과 기능 안정성이 필요한 시스템에서 작동 합니다.

다음 표에서는 각 유형의 기본 이미지, 해당 서비스 채널 및 해당 지원의 지속 시간을 보여 줍니다.

|Base image                       |서비스 채널|버전|OS 빌드|가용성|일반 지원 종료 날짜|지원 날짜 연장|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, Nano Server, Windows|반기      |1909   |18363   |2019/11/12  |2021/05/11                 |N/A                  |
|Server Core, Nano Server, Windows|반기      |1903   |18362   |05/21/2019  |2020/12/08                 |N/A                  |
|Server Core                      |장기        |2019   |17763   |2018/11/13  |2024/01/09                 |2029/01/09           |
|Server Core, Nano Server, Windows|반기      |1809   |17763   |2018/11/13  |11/10/2020                 |N/A                  |
|Server Core, Nano Server         |반기      |1803   |17134   |2018/04/30  |2019/11/12                 |N/A                  |
|Server Core, Nano Server         |반기      |1709   |16299   |2017/10/17  |2019/04/09                 |N/A                  |
|Server Core                      |장기        |1607   |14393   |2016/10/15  |2022/01/11                 |2027/01/11           |
|Nano 서버                      |반기      |1607   |14393   |2016/10/15  |10/09/2018                 |N/A                  |

서비스 요구 사항 및 기타 추가 정보는 [Windows 수명 주기 FAQ](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products), [windows Server 릴리스 정보](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)및 [windows 기본 OS 이미지 Docker 허브 리포지토리](https://hub.docker.com/_/microsoft-windows-base-os-images)를 참조 하세요.
