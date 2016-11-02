---
title: "Windows 10의 Hyper-V 소개"
description: "Windows 10의 Hyper-V 소개"
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 702894a9f1d89d1f6a5846048f9dd794d9d1a59c

---

# Windows 10의 Hyper-V 소개

많은 소프트웨어 개발자, IT 전문가 또는 기술 매니아들은 여러 운영 체제를 실행해야 합니다.  각 컴퓨터에 물리적 하드웨어를 개별적으로 지정하는 대신 Hyper-V를 사용하여 Windows 컴퓨터에서 여러 운영 체제를 VM(가상 컴퓨터)으로 실행할 수 있습니다.

> Microsoft Virtual PC는 2017년 4월에 운영이 종료됩니다. Windows 10 Enterprise 및 Windows 10 Professional의 Hyper-V가 대체 도구로 지원됩니다.  

## 가상화를 사용하는 이유
가상화를 사용하면 누구나 여러 운영 체제, 소프트웨어 구성 및 하드웨어 구성을 동일한 물리적 컴퓨터에서 실행할 수 있습니다.  Hyper-V는 가상화뿐 아니라 가상 컴퓨터를 관리하는 도구도 제공합니다.

Hyper-V는 다양한 방법으로 사용할 수 있습니다. 예를 들면 다음과 같습니다.

* 이전 버전의 Windows나 Windows가 아닌 운영 체제가 필요한 소프트웨어를 실행합니다. 

* 다른 운영 체제를 시험해 봅니다. Hyper-V를 사용하면 다른 운영 체제를 아주 쉽게 만들고 제거할 수 있습니다.

* 여러 가상 컴퓨터를 사용하여 여러 운영 체제에서 소프트웨어를 테스트합니다. Hyper-V를 사용하면 모든 가상 컴퓨터를 하나의 데스크톱 또는 노트북 컴퓨터에서 실행할 수 있습니다. 이러한 가상 컴퓨터를 내보낸 다음 Azure를 비롯한 다른 Hyper-V 시스템으로 가져올 수 있습니다.

* 어느 Hyper-V 배포에서나 가상 컴퓨터 문제를 해결합니다. 프로덕션 환경에서 가상 컴퓨터를 내보내고 Hyper-V를 실행하는 데스크톱에서 열고 가상 컴퓨터 문제를 해결한 다음 프로덕션 환경으로 다시 내보낼 수 있습니다. 

* 가상 네트워킹을 사용하여 프로덕션 네트워크에는 영향을 미치지 않도록 하면서 테스트/개발/데모를 위한 다중 컴퓨터 환경을 만들 수 있습니다.

## 시스템 요구 사항
Hyper-V는 Windows 8 이상의 Windows Professional, Enterprise 및 Education 버전에서만 사용할 수 있습니다.

Hyper-V에는 SLAT(Second Level Address Translation)가 있는 64비트 시스템이 필요합니다. SLAT는 Intel 및 AMD에서 제공하는 최신 세대 64비트 프로세서에 있는 기능입니다.  또한 64비트 버전의 Windows도 필요합니다.  
이와 더불어 Hyper-V는 가상 컴퓨터 내에서 32비트 및 64비트 운영 체제를 모두 지원합니다.

RAM이 4GB인 호스트에서 3~4개의 기본 가상 컴퓨터를 실행할 수 있고, 가상 컴퓨터 수가 늘면 더 많은 리소스가 필요합니다. 크게는 물리적 하드웨어에 따라 32개의 프로세서와 512GB RAM을 사용하는 대형 가상 컴퓨터를 만들 수도 있습니다.

Hyper-V 시스템 요구 사항 및 Hyper-V가 컴퓨터에서 실행되는지 확인하는 방법에 대한 자세한 내용은 [연습: Windows 10 Hyper-V 시스템 요구 사항](..\quick_start\walkthrough_install.md)을 참조하세요.


## 가상 컴퓨터에서 실행할 수 있는 운영 체제
"게스트"라는 말은 가상 컴퓨터를, "호스트"는 가상 컴퓨터를 실행하는 컴퓨터를 의미합니다. Windows의 Hyper-V는 다양한 Linux, FreeBSD 및 Windows 버전을 포함하여 여러 게스트 운영 체제를 지원합니다. 

참고로 VM에서 사용하는 모든 운영 체제에는 유효한 라이선스가 필요합니다. 

Windows의 Hyper-V에서 게스트로 지원되는 운영 체제에 대한 자세한 내용은 [지원되는 Windows 게스트 운영 체제](supported_guest_os.md) 및 [Hyper-V의 Linux 및 FreeBSD 가상 컴퓨터](https://technet.microsoft.com/library/dn531030.aspx)를 참조하세요. 


## Windows의 Hyper-V와 Windows Server의 Hyper-V 간 차이점
Windows에서 Hyper-V를 실행할 때 몇 가지 기능은 Windows Server에서 Hyper-V를 실행할 때와는 다르게 작동합니다. 

메모리 관리 모델이 Windows의 Hyper-V에서 다릅니다. 서버에서 Hyper-V 메모리는 해당 서버에서 관리 컴퓨터만 실행된다는 가정 하에 관리됩니다. Windows의 Hyper-V에서 메모리는 대부분의 클라이언트 컴퓨터가 가상 컴퓨터 실행 외에도 호스트의 소프트웨어를 실행한다는 예상에 따라 관리됩니다. 예를 들어, 개발자가 동일한 컴퓨터에서 여러 가상 컴퓨터와 Visual Studio를 실행할 수 있습니다.

### Windows Server에서만 사용할 수 있는 Hyper-V 기능
Windows의 Hyper-V에는 Windows Server의 Hyper-V 기능이 몇 가지 포함되지 않았습니다. 확인할 수 있습니다.

* RemoteFX를 사용하여 GPU 가상화 
* 가상 컴퓨터를 실시간으로 한 호스트에서 다른 호스트로 마이그레이션
* Hyper-V 복제본
* 가상 파이버 채널
* SR-IOV 네트워킹
* 공유 .VHDX

## 제한 사항
가상화 사용에는 제한이 있습니다. 특정 하드웨어에 종속된 기능이나 응용 프로그램은 가상 컴퓨터에서 제대로 작동하지 않습니다. 예를 들어 GPU를 통한 처리가 필요한 게임이나 응용 프로그램은 제대로 작동하지 않을 수 있습니다. 또한 라이브 뮤직 믹싱 응용 프로그램이나 고정밀 시간과 같이 10밀리초 미만의 타이머를 사용하는 응용 프로그램은 가상 컴퓨터에서 실행될 때 문제가 있을 수 있습니다.

또한 Hyper-V를 사용하도록 설정한 경우 대기 시간에 민감한 고정밀 응용 프로그램은 호스트에서 실행될 때도 문제가 있을 수 있습니다.  이는 가상화를 사용하도록 설정하면 호스트 OS도 게스트 운영 체제처럼 Hyper-V 가상화 계층 위에서 실행되기 때문입니다. 그러나 게스트와 달리 호스트 OS는 모든 하드웨어에 직접 액세스할 수 있다는 점에서 특별합니다. 즉, 하드웨어 요구 사항이 특별한 응용 프로그램도 호스트 OS에서 문제 없이 실행될 수 있다는 의미입니다.

## 다음 단계
[연습: Windows 10에 Hyper-V 설치](..\quick_start\walkthrough_install.md) 

Windows 10의 Hyper-V에서 [새로운 기능](whats_new.md)을 확인해 보세요.




<!--HONumber=Oct16_HO4-->


