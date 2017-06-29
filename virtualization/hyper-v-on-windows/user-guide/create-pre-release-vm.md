---
title: "Hyper-V의 시험판 기능 체험"
description: "Hyper-V의 시험판 기능 체험"
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 426c87cc-fa50-4b8d-934e-0b653d7dea7d
ms.openlocfilehash: 8df30a00eaa2c98feeb4c80c302937c9dfc6d758
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: ko-KR
---
# <a name="try-pre-release-features-for-hyper-v"></a>Hyper-V의 시험판 기능 체험

> 이 예비 콘텐츠는 변경될 수 있습니다.  
  시험판 가상 컴퓨터는 Microsoft에서 지원되지 않으므로 개발 또는 테스트 환경에만 사용됩니다.

Windows Server 2016 Technical Preview에서 Hyper-V의 시험판 기능에 미리 액세스하여 개발 또는 테스트 환경에서 사용해 보세요. 최신 Hyper-V 기능을 가장 먼저 확인하고 초기 피드백을 제공하여 제품 구성에 도움을 줄 수 있습니다.

시험판으로 만드는 가상 컴퓨터는 빌드 간 호환성이 없거나 향후 지원되지 않을 수 있습니다.  그러므로 프로덕션 환경에서 사용하지 마세요.

다음은 이러한 컴퓨터를 프로덕션 환경용으로 사용할 수 없는 몇 가지 이유입니다.

* 시험판 가상 컴퓨터에는 다음 버전과의 호환성이 없습니다. 이러한 가상 컴퓨터를 새 구성 버전으로 업그레이드할 수 없습니다.
* 시험판 가상 컴퓨터의 빌드 사이에 일관된 정의가 없습니다. 호스트 운영 체제를 업데이트하는 경우 기존 시험판 가상 컴퓨터가 호스트와 호환되지 않을 수 있습니다. 이러한 가상 컴퓨터는 시작되지 않거나, 초기에는 작동하는 것처럼 보이지만 나중에는 중요한 호환성 문제가 발생할 수 있습니다.
* 다른 빌드를 사용하는 호스트로 시험판 가상 컴퓨터를 가져오는 경우 결과를 예측할 수 없습니다. 시험판 가상 컴퓨터를 다른 호스트로 이동할 수는 있습니다. 그러나 이 시나리오는 두 호스트가 모두 동일한 빌드에서 실행되는 경우에만 작동합니다.

## <a name="create-a-pre-release-virtual-machine"></a>시험판 가상 컴퓨터 만들기

Windows Server 2016 Technical Preview가 실행되는 Hyper-V 호스트에 시험판 가상 컴퓨터를 만들 수 있습니다.

1. Windows 데스크톱에서 시작 단추를 클릭하고 **Windows PowerShell** 이름의 일부를 입력합니다.
2. **Windows PowerShell**을 마우스 오른쪽 단추로 클릭하고 **관리자 권한으로 실행**을 선택합니다.
3. -Prerelease 플래그와 함께 [NEW-VM](https://technet.microsoft.com/library/hh848537.aspx) cmdlet을 사용하여 시험판 가상 컴퓨터를 만듭니다. 예를 들어 다음 명령을 실행합니다. 여기서 VM Name은 만들려는 가상 컴퓨터의 이름입니다.

``` PowerShell
New-VM -Name <VM Name> -Prerelease
```
다른 예제에서는 다음과 같이 -Prerelease 플래그를 사용할 수 있습니다.
 - 기존 가상 하드 디스크 또는 새 하드 디스크를 사용하는 가상 컴퓨터를 만들려면 [Create a virtual machine in Hyper-V on Windows Server 2016 Technical Preview(Windows Server 2016 Technical Preview의 Hyper-V에서 가상 컴퓨터 만들기)](https://technet.microsoft.com/library/mt126140.aspx#BKMK_PowerShell)에서 PowerShell 예제를 참조하세요.
 - 운영 체제 이미지로 부팅되는 새 가상 하드 디스크를 만들려면 [Windows 10의 Hyper-V에 Windows 가상 컴퓨터 배포](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_create_vm)에서 PowerShell 예제를 참조하세요.

 이러한 문서에서 다루는 예제는 Windows 10 또는 Windows Server 2016 Technical Preview가 실행되는 Hyper-V 호스트에서 작동합니다. 그러나 지금은 -Prerelease 플래그만 사용하여 Windows Server 2016 Technical Preview가 실행되는 Hyper-V 호스트에 시험판 가상 컴퓨터를 만들 수 있습니다.

## <a name="see-also"></a>참고 항목
-  [Virtualization Blog(가상화 블로그)](https://blogs.technet.microsoft.com/virtualization/) - 사용할 수 있는 시험판 기능 및 체험 방법에 대해 알아봅니다.
- [Supported virtual machine configuration versions(지원되는 가상 컴퓨터 구성 버전)](https://technet.microsoft.com/library/mt695898.aspx#BKMK_SupportedConfigVersions) - 가상 컴퓨터 구성 버전을 확인하는 방법 및 Microsoft에서 지원하는 버전에 대해 알아봅니다.
