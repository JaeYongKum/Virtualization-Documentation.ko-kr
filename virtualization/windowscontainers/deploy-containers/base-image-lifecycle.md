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
ms.openlocfilehash: d3a8240dbba8af3c74ce5d185620e129d103ef81
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883186"
---
# <a name="base-image-servicing-lifecycles"></a>기본 이미지 서비스 수명 주기

Windows 컨테이너 기반 이미지는 Windows Server의 반기 채널 릴리스 또는 장기 서비스 채널 릴리스를 기반으로 합니다. 이 문서에서는 두 채널의 여러 버전의 기본 이미지에 대 한 지원이 얼마나 오래 지속 되는지 설명 합니다.

반기 채널은 각 릴리스에 대 한 eighteen의 서비스 timeline을 사용 하는 연간 2 개월 기능 업데이트 릴리스입니다. 이를 통해 고객은 응용 프로그램 (특히 컨테이너 및 마이크로 서비스를 기반으로 하는 시스템) 및 소프트웨어 정의 된 하이브리드 데이터 센터에서 새로운 운영 체제 기능을 더욱 빠르게 활용할 수 있습니다. 자세한 내용은 [Windows Server 반기 채널 개요](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)를 참조 하세요.

서버 코어 이미지의 경우 고객은 2 ~ 3 년 마다 Windows Server의 새 주요 버전을 릴리스 하는 장기 서비스 채널을 사용할 수도 있습니다. 장기 서비스 채널 릴리스는 5 년 이상의 기본 지원 및 5 년 연장 지원을 받습니다. 이 채널은 더 긴 서비스 옵션 및 기능 안정성을 필요로 하는 시스템에서 작동 합니다.

다음 표에는 각 종류의 기본 이미지와 해당 서비스 채널이 나열 되어 있으며,이에 대 한 지원은 마지막으로 얼마나 오래 걸릴 수 있습니다.

|기본 이미지                       |서비스 채널|버전|OS 빌드|가용성|일반 지원 종료 날짜|연장 지원 날짜|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, Nano 서버, Windows|반기      |1903   |18362   |05/21/2019  |12/08/2020                 |해당 없음                  |
|Server Core                      |장기간        |1809   |17763   |2018/11/13  |2024/01/09                 |2029/01/09           |
|Server Core, Nano 서버, Windows|반기      |1809   |17763   |2018/11/13  |05/12/2020                 |해당 없음                  |
|Server Core, Nano 서버         |반기      |1803   |17134   |2018/04/30  |2019/11/12                 |해당 없음                  |
|Server Core, Nano 서버         |반기      |1709   |16299   |2017/10/17  |2019/04/09                 |해당 없음                  |
|Server Core                      |장기간        |1607   |14393   |2016/10/15  |2022/01/11                 |2027/01/11           |
|Nano 서버                      |반기      |1607   |14393   |2016/10/15  |10/09/2018                 |해당 없음                  |

서비스 요구 사항 및 기타 추가 정보는 [Windows 수명 주기 FAQ](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products), [windows Server 릴리스 정보](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)및 [windows 기반 OS 이미지 Docker 허브 리포지토리](https://hub.docker.com/_/microsoft-windows-base-os-images)를 참조 하세요.
