---
title: Windows 10에서 Hyper-V를 사용하도록 설정
description: Windows 10에 Hyper-V 설치
keywords: windows 10, Hyper-V
author: scooley
ms.date: 02/15/2019
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: bad59fcc65bf66ab3c6dc940a17111e46a9bc226
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439700"
---
# <a name="install-hyper-v-on-windows-10"></a>Windows 10에 Hyper-V 설치

Windows 10에서 가상 컴퓨터를 만들 수 있도록 Hyper-V를 사용하도록 설정합니다.  
Windows 10 제어판, PowerShell 또는 DISM(Deployment Imaging Servicing and Management) 도구를 사용하는 방법을 포함하여 여러 가지 방법으로 Hyper-V를 사용하도록 설정할 수 있습니다. 이 문서에서는 각 옵션을 안내합니다.

> **참고:**  Hyper-V는 Windows에서 선택적 기능으로 기본 제공되므로 Hyper-V를 다운로드할 필요가 없습니다.

## <a name="check-requirements"></a>요구 사항 확인

* Windows 10 Enterprise, Pro 또는 Education
* 두 번째 수준 주소 변환(SLAT)을 사용하는 64비트 프로세서.
* VM 모니터 모드 확장(Intel CPU의 VT-c)을 지원하는 CPU.
* 최소 4GB의 메모리.

Hyper-V 역할은 Windows 10 Home에는 **설치할 수 없습니다**.

**설정** > **업데이트 및 보안** > **정품 인증**을 열어 Windows 10 Home 버전을 Windows 10 Pro로 업그레이드하세요.

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

DISM(배포 이미지 서비스 및 관리) 도구를 사용하면 Windows와 Windows 이미지를 구성하는 데 도움이 됩니다.  DISM은 많은 애플리케이션을 갖추고 있으며, 운영 체제가 실행 중인 동안 Windows 기능을 사용하도록 설정할 수 있습니다.

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

2. 오른쪽의 관련 설정에서 **프로그램 및 기능**를 선택합니다. 

3. **Windows 기능 사용/사용 안 함**을 선택합니다.

4. **Hyper-V**를 선택하고 **확인**을 클릭합니다.

![Windows 프로그램 및 기능 대화 상자](media/enable_role_upd.png)

설치가 완료되면 컴퓨터를 다시 시작하라는 메시지가 표시됩니다.

## <a name="make-virtual-machines"></a>가상 컴퓨터 만들기

[첫 번째 가상 머신 만들기](quick-create-virtual-machine.md)
