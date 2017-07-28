---
title: "Hyper-V를 사용하여 가상 컴퓨터 만들기"
description: "Windows 10 크리에이터스 업데이트에서 Hyper-V를 사용하여 새 가상 컴퓨터 만들기"
keywords: windows 10, hyper-v
author: aoatkinson
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 1b2b778e882b413d29f52adf3e46e12e8aceede1
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/21/2017
---
# Hyper-V를 사용하여 가상 컴퓨터 만들기

가상 컴퓨터를 만들고 운영 체제를 설치합니다.  

실행하려는 운영 체제의 .iso 파일이 필요합니다. 파일이 없는 경우 [TechNet Evaluation Center](http://www.microsoft.com/en-us/evalcenter/)에서 Windows 평가판을 가져옵니다.


> Windows 10 크리에이터스 업데이트에는 새 가상 컴퓨터를 간단하게 만들 수 있는 새로운 **빨리 만들기** 도구가 도입되었습니다.  
  Windows 10 크리에이터스 업데이트 이상을 실행하고 있지 않으면 다음 지침에 따라 새 가상 컴퓨터 마법사를 대신 사용합니다.  
  [새 가상 컴퓨터 만들기](create-virtual-machine.md)  
  [가상 네트워크 만들기](connect-to-network.md)

그럼 시작해 보겠습니다.

![](media/quickcreatesteps_inked.jpg)

1. **Hyper-V 관리자 열기**  
  Windows 키를 누르고 "Hyper-V 관리자"를 입력하여 Hyper-V 관리자를 검색하거나 시작 메뉴에서 Hyper-V 관리자를 찾을 때까지 응용 프로그램을 스크롤합니다.

2. **빨리 만들기 열기**  
  Hyper-V 관리자의 오른쪽에 있는 **작업** 메뉴에서 **빨리 만들기**를 찾습니다.

3. **가상 컴퓨터 사용자 지정**
  * (선택 사항) 가상 컴퓨터 이름을 지정합니다.  
    가상 컴퓨터 내부에 배포될 게스트 운영 체제에 지정되는 컴퓨터 이름이 아니라 Hyper-V에서 가상 컴퓨터에 사용하는 이름입니다.
  * 가상 컴퓨터를 설치할 미디어를 선택합니다. .iso 또는 .vhdx 파일로 설치할 수 있습니다.  
    가상 컴퓨터에서 Windows를 설치하는 경우 Windows 보안 부팅을 사용할 수 있습니다. 그렇지 않으면 선택되지 않은 상태로 둡니다.
  * 네트워크를 설정합니다.  
    기존 가상 스위치가 있는 경우 네트워크 드롭다운 목록에서 선택할 수 있습니다. 기존 스위치가 없는 경우 자동 네트워크를 설정하는 단추가 보일 것입니다. 이 단추는 자동으로 외부 스위치를 구성합니다.

4. **가상 컴퓨터에 연결**  
  **연결**을 선택하면 가상 컴퓨터 연결이 실행되고 가상 컴퓨터가 시작됩니다.     
  설정 편집에 대해 걱정할 필요가 없습니다. 언제든지 돌아가서 변경할 수 있습니다.  
  
    'CD 또는 DVD에서 부팅하려면 아무 키나 누르십시오.'라는 메시지가 표시되면 아무 키나 눌러 진행합니다.  가상 컴퓨터는 사용자가 CD로 설치하는 것으로 인식합니다.

축하합니다. 새 가상 컴퓨터가 생겼습니다.  이제 운영 체제를 설치할 준비가 완료되었습니다.  

가상 컴퓨터가 다음과 같이 보일 것입니다.  
![](media/OSDeploy_upd.png) 

> **참고:** 볼륨 라이선스 버전의 Windows를 실행 중이 아닌 경우 가상 컴퓨터 내에서 실행 중인 Windows에 대한 별도 라이선스가 필요합니다. 가상 컴퓨터의 운영 체제는 별개의 호스트 운영 체제입니다.
