---
title: Windows 10 Hyper-V 시스템 요구 사항
description: Windows 10 Hyper-V 시스템 요구 사항
keywords: windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: ebc9be132f05c20eb8daf9b5e6713b9258012305
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853994"
---
# <a name="windows-10-hyper-v-system-requirements"></a>Windows 10 Hyper-V 시스템 요구 사항

Hyper-v는 64 비트 버전의 Windows 10 Pro, Enterprise 및 교육용에서 사용할 수 있습니다. Hyper-V는 Intel 및 AMD의 최신 세대 64비트 프로세서에서 제공하는 SLAT(두 번째 수준 주소 변환)가 필요합니다.

RAM이 4GB인 호스트에서 3~4개의 기본 가상 컴퓨터를 실행할 수 있고, 가상 컴퓨터 수가 늘면 더 많은 리소스가 필요합니다. 크게는 물리적 하드웨어에 따라 32개의 프로세서와 512GB RAM을 사용하는 대형 가상 컴퓨터를 만들 수도 있습니다.

## <a name="operating-system-requirements"></a>운영 체제 요구 사항

이러한 버전의 Windows 10에서 Hyper-V 역할을 활성화할 수 있습니다.

- Windows 10 Enterprise
- Windows 10 Pro
- Windows 10 Education

다음 버전에는 Hyper-V 역할을 설치할 수 **없습니다**.

- Windows 10 Home
- Windows 10 Mobile
- Windows 10 Mobile Enterprise

>Windows 10 Home edition은 Windows 10 Pro로 업그레이드할 수 있습니다. 그러려면 **설정** > **업데이트 및 보안** > **활성화**를 엽니다. 여기에서 스토어를 방문하고 업그레이드를 구입할 수 있습니다.

## <a name="hardware-requirements"></a>하드웨어 요구 사항

이 문서에서는 Hyper-V와 호환되는 하드웨어의 전체 목록을 제공하지 않지만 다음 항목은 필요합니다.

- 두 번째 수준 주소 변환(SLAT)을 사용하는 64비트 프로세서.
- VM 모니터 모드 확장에 대 한 CPU 지원 (Intel CPU의 VT-x)
- 최소 4GB의 메모리. 가상 컴퓨터는 Hyper-V 호스트와 메모리를 공유하지만 예상된 가상 워크로드를 처리하려면 충분한 메모리를 제공해야 합니다.

다음 항목을 시스템 BIOS에서 사용하도록 설정해야 합니다.
- 가상화 기술 - 마더보드 제조업체에 따라 다른 레이블이 있을 수 있습니다.
- 하드웨어 적용 데이터 실행 방지.

## <a name="verify-hardware-compatibility"></a>하드웨어 호환성 확인

위의 운영 체제 및 하드웨어 요구 사항을 확인 한 후 PowerShell 세션 또는 명령 프롬프트 (cmd.exe) 창을 열고, **systeminfo**를 입력 한 다음 Hyper-v 요구 사항 섹션을 확인 하 여 Windows에서 하드웨어 호환성을 확인 합니다. 나열된 모든 Hyper-V 요구 사항의 값이 **예**인 경우 시스템에서 Hyper-V 역할을 실행할 수 있습니다. 항목이 **아니요**를 반환하는 경우 이 문서에 나열된 요구 사항을 확인하고 가능한 경우 조정합니다.

![](media/SystemInfo-upd.png)

## <a name="final-check"></a>최종 확인

모든 OS, 하드웨어 및 호환성 요구 사항이 충족 되 면 제어판의 **hyper-v** 가 표시 됩니다 **. Windows 기능을 설정 하거나 해제** 하 고 두 가지 옵션을 사용할 수 있습니다.

1. Hyper-v 플랫폼
1. Hyper-V 관리 도구

![](media/hyper_v_feature_screenshot.png)

> [!NOTE] 제어판에서 **hyper-v** 대신 **windows 하이퍼바이저 플랫폼** 을 사용 하는 경우 **Windows 기능 사용 >/사용 안 함** 이 hyper-v와 호환 되지 않을 수 있습니다. 그런 다음, 위의 요구 사항을 교차 검사 합니다.
>
>기존 Hyper-V 호스트에서 **systeminfo**를 실행하는 경우 Hyper-V 요구 사항 섹션을 읽습니다.
>
>```
>Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V will not be displayed.
>```
