---
title: "Windows 가상 컴퓨터와 장치 공유"
description: "장치를 Hyper-V 가상 컴퓨터(USB, 오디오, 마이크 및 탑재된 드라이브)와 공유하는 방법을 안내합니다."
keywords: windows 10, hyper-v
ms.author: scooley
ms.date: 10/20/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d1aeb9cb-b18f-43cb-a568-46b33346a188
ms.openlocfilehash: 52d51fca03f454a311a123f20e5aeda9376fdc3d
ms.sourcegitcommit: 3ec9917c456875f68180323eb9e0470d5115325a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/27/2017
---
# <a name="share-devices-with-your-virtual-machine"></a>가상 컴퓨터와 장치 공유

> Windows 가상 컴퓨터에만 사용할 수 있습니다.

고급 세션 모드를 사용하면 Hyper-V를 통해 RDP(원격 데스크톱 프로토콜)를 사용하여 가상 컴퓨터에 연결할 수 있습니다.  이렇게 하면 일반적인 가상 컴퓨터 보기 환경이 향상될 뿐만 아니라 RDP와 연결하면 가상 컴퓨터에서 장치를 컴퓨터와 공유할 수도 있습니다.  Windows 10에는 기본적으로 설정되어 있으므로 이미 RDP를 사용하여 Windows 가상 컴퓨터에 연결 중일 것입니다.  이 문서에서는 연결 설정 대화 상자의 장점과 숨겨진 옵션 몇 가지를 중점적으로 설명합니다.

RDP/고급 세션 모드:

* 가상 컴퓨터 크기를 조정하고 DPI 인식률을 높입니다.
* 가상 컴퓨터 통합 향상
  * 공유 클립보드
  * 끌어서 놓기 및 복사/붙여넣기를 통해 파일 공유
* 장치 공유 허용
  * 마이크/스피커
  * USB 장치
  * 데이터 디스크(C: 드라이브 포함)
  * 프린터

이 문서에서는 세션 유형을 확인하고 고급 세션 모드로 들어가고 세션 설정을 구성하는 방법에 대해 설명합니다.

## <a name="share-drives-and-devices"></a>드라이브 및 장치 공유

고급 세션 모드의 장치 공유 기능은 가상 컴퓨터에 연결할 때 나타나는 눈에 잘 띄지 않는 이 연결 창 내부에 숨겨져 있습니다.

![](media/esm-default-view.png)

기본적으로 고급 세션 모드를 사용하는 가상 컴퓨터는 클립보드 및 프린터를 공유합니다.  또한 기본적으로 가상 컴퓨터의 오디오를 컴퓨터 스피커에 다시 전달하도록 구성되어 있습니다.

장치를 가상 컴퓨터와 공유하거나 기본 설정을 변경하려면 다음과 같이 하세요.

1. 기타 옵션 표시

  ![](media/esm-show-options.png)

1. 로컬 리소스 보기

  ![](media/esm-local-resources.png)

### <a name="share-storage-and-usb-devices"></a>저장소 및 USB 장치 공유

기본적으로 고급 세션 모드를 사용하는 가상 컴퓨터는 사용자가 가상 컴퓨터에서 더욱 안전한 로그인 도구를 사용할 수 있도록 프린터와 클립보드를 공유하고 가상 컴퓨터에 스마트카드 및 기타 보안 장치를 전달합니다.

USB 장치 또는 사용자의 C: 드라이브와 같은 다른 장치를 공유하려면 "추가..." 메뉴를 선택하세요.  
![](media/esm-more-devices.png)

이 메뉴에서 가상 컴퓨터와 공유할 장치를 선택할 수 있습니다.  시스템 드라이브(Windows C:)는 파일 공유 시 특히 유용합니다.  
![](media/esm-drives-usb.png)

### <a name="share-audio-devices-speakers-and-microphones"></a>오디오 장치(스피커 및 마이크) 공유

가상 컴퓨터에서 오디오를 들을 수 있도록 고급 세션 모드를 사용하는 가상 컴퓨터는 기본적으로 오디오를 전달합니다.  가상 컴퓨터는 호스트 컴퓨터에서 현재 선택된 오디오 장치를 사용합니다.

이러한 설정을 변경하거나 가상 컴퓨터의 오디오를 녹음할 수 있도록 마이크 통과를 추가하려면 다음과 같이 하세요.

원격 오디오 설정을 구성하려면 "설정..." 메뉴를 선택합니다.  
![](media/esm-audio.png)

오디오 및 마이크 설정 지금 구성  
![](media/esm-audio-settings.png)

가상 컴퓨터가 로컬로 실행 중인 것 같으므로 "이 컴퓨터에서 재생" 및 "원격 컴퓨터에서 재생" 옵션은 동일한 결과를 냅니다.

## <a name="re-launching-the-connection-settings"></a>연결 설정 다시 시작

해상도 및 장치 공유 대화 상자가 나타나지 않으면 Windows 메뉴에서 또는 관리자 권한으로 명령줄을 실행하여 VMConnect를 별도로 다시 실행해 봅니다.  

``` Powershell
vmconnect.exe
```

## <a name="check-session-type"></a>세션 형식 확인

가상 컴퓨터 연결 도구(VMConnect) 상단에 있는 고급 세션 모드 아이콘을 사용하여 연결 유형을 확인할 수 있습니다.  이 버튼을 사용하여 기본 세션 및 고급 세션 모드 간에 전환할 수도 있습니다.

![](media/esm-button-location.png)

| 아이콘 | 연결 상태 |
|:-----|:---------|
|![](media/esm-basic.png)| 현재 고급 세션 모드로 실행 중입니다.  이 아이콘을 클릭하면 가상 컴퓨터가 기본 모드로 다시 연결됩니다. |
|![](media/esm-connect.png)| 현재 기본 세션 모드로 실행 중이지만 고급 세션 모드를 사용할 수 있습니다.  이 아이콘을 클릭하면 가상 컴퓨터가 고급 세션 모드로 다시 연결됩니다.  |
|![](media/esm-stop.png)| 현재 기본 모드로 실행 중입니다.  고급 세션 모드는 이 가상 컴퓨터에 사용할 수 없습니다. |