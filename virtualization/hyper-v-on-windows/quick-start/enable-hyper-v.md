---
title: Windows 10에서 Hyper-V를 사용하도록 설정
description: Windows 10에 Hyper-V 설치
keywords: windows 10, hyper-v
author: scooley
ms.date: 02/15/2019
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: e1b6b55b2e17ac4f0883078748d75f6d4b9fcafa
ms.sourcegitcommit: 3c06d6b5cebeacc4c0185dca5c20680ad67655bd
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681904"
---
# <a name="install-hyper-v-on-windows-10"></a>Windows 10에 Hyper-V 설치

Windows 10에서 가상 컴퓨터를 만들 수 있도록 Hyper-V를 사용하도록 설정합니다.  
Windows 10 제어판, PowerShell을 사용 하거나 DISM (배포 이미징 서비스 및 관리) 도구를 사용 하는 등 다양 한 방법으로 hyper-v를 사용할 수 있습니다. 이 문서에서는 각 옵션을 안내합니다.

> **참고:**  Hyper-V는 선택적 기능으로 Windows에서 기본 제공되기 때문에 Hyper-V 다운로드가 필요하지 않습니다.

## <a name="check-requirements"></a>요구 사항 확인

* Windows 10 Enterprise, Pro 또는 교육
* 두 번째 수준 주소 변환(SLAT)을 사용하는 64비트 프로세서.
* VM 모니터 모드 확장에 대 한 CPU 지원 (Intel Cpu의 VT-c).
* 최소 4GB의 메모리.

Hyper-V 역할은 Windows 10 Home에는 **설치할 수 없습니다**.

**설정** > **업데이트 및 보안** > **활성화**를 열어 windows 10 Home edition에서 windows 10 Pro로 업그레이드 합니다.

자세한 내용과 문제 해결은 [Windows 10 Hyper-V 시스템 요구 사항](../reference/hyper-v-requirements.md)을 참조하세요.

## <a name="enable-hyper-v-using-powershell"></a>PowerShell을 사용하여 Hyper-V를 사용하도록 설정

1. 관리자 권한으로 PowerShell 콘솔을 엽니다.

2. 다음 명령을 실행합니다.

  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
  ```

  명령을 찾을 수 없을 경우 관리자 권한으로 PowerShell을 실행하고 있는지 확인합니다.

설치가 완료되면 컴퓨터를 다시 부팅합니다.

## <a name="enable-hyper-v-with-cmd-and-dism"></a>CMD와 DISM을 사용하여 Hyper-V를 사용하도록 설정

DISM(배포 이미지 서비스 및 관리) 도구를 사용하면 Windows와 Windows 이미지를 구성하는 데 도움이 됩니다.  DISM은 많은 응용 프로그램을 갖추고 있으며, 운영 체제가 실행 중인 동안 Windows 기능을 사용하도록 설정할 수 있습니다.

DISM을 사용하여 Hyper-V 역할을 활성화하려면:

1. 관리자 권한으로 PowerShell 또는 CMD 세션을 엽니다.

1. 다음 명령을 입력합니다.

  ```powershell
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  ```

  ![콘솔 창에 사용하도록 설정된 Hyper-V가 표시됩니다.](media/dism_upd.png)

DISM에 대한 자세한 내용은 [DISM 기술 참조](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh824821(v=win.10)>)를 참조하세요.

## <a name="enable-the-hyper-v-role-through-settings"></a>설정을 통해 Hyper-V 역할 활성화

1. Windows 단추를 마우스 오른쪽 단추로 클릭하고 '앱 및 기능'을 선택합니다.

2. 관련 설정에서 오른쪽에 있는 **프로그램 및 기능** 을 선택 합니다. 

3. **Windows 기능 사용/사용 안 함**을 선택합니다.

4. **Hyper-V**를 선택하고 **확인**을 클릭합니다.

![Windows 프로그램 및 기능 대화 상자](media/enable_role_upd.png)

설치가 완료되면 컴퓨터를 다시 시작하라는 메시지가 표시됩니다.

## <a name="make-virtual-machines"></a>가상 컴퓨터 만들기

[첫 번째 가상 컴퓨터 만들기](quick-create-virtual-machine.md)
