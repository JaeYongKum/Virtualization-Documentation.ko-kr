---
title: Windows 컨테이너 기본 이미지 기록
description: SHA256 계층 해시과 함께 릴리스된 Windows 컨테이너 이미지 목록
keywords: Docker, 컨테이너, 해시
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: b2f2d6418fdda2ad0aa0b81c05efad6b99f74375
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107907"
---
# <a name="container-base-images"></a>컨테이너 기본 이미지

## <a name="supported-base-images"></a>지원 되는 기본 이미지

Windows 컨테이너는 Windows Server Core, Nano 서버, Windows, IoT Core 등 네 가지 컨테이너 기본 이미지와 함께 제공 됩니다. 일부 구성은 두 개의 OS 이미지를 모두 지원하지 않습니다. 이 테이블은 지원되는 구성을 자세히 설명합니다.

|호스트 운영 체제|Windows 컨테이너|Hyper-V 격리|
|---------------------|-----------------|-----------------|
|Windows Server 2016 또는 Windows Server 2019 (Standard 또는 Datacenter)|Server Core, Nano 서버, Windows|Server Core, Nano 서버, Windows|
|Nano Server|Nano Server|Server Core, Nano 서버, Windows|
|Windows 10 Pro 또는 Windows 10 Enterprise|사용할 수 없음|Server Core, Nano 서버, Windows|
|IoT Core|IoT Core|사용할 수 없음|

> [!WARNING]  
> Windows Server 버전 1709부터 Nano 서버는 더 이상 컨테이너 호스트로 사용할 수 없습니다.

## <a name="base-image-differences"></a>기본 이미지 차이점

어떤 방법으로 빌드에 적합 한 기본 이미지를 선택 하나요? 원하는 모든 항목을 사용 하 여 빌드할 수 있지만, 각 이미지에 대 한 일반적인 지침은 다음과 같습니다.

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): 응용 프로그램에 전체 .net framework가 필요한 경우이 이미지를 사용 하는 것이 가장 좋습니다.
- [Nano 서버](https://hub.docker.com/_/microsoft-windows-nanoserver): .net Core만 필요로 하는 응용 프로그램의 경우 Nano 서버는 훨씬 얇고 이미지를 제공 합니다.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): 응용 프로그램이 서버 Core 또는 Nano 서버 이미지 (예: GDI 라이브러리)에 없는 구성 요소 또는 .dll에 종속 되어 있는 것을 확인할 수 있습니다. 이 이미지는 Windows의 전체 종속성 집합을 전달 합니다.
- [Iot Core](https://hub.docker.com/_/microsoft-windows-iotcore):이 이미지는 [IoT 애플리케이션을](https://developer.microsoft.com/windows/iot)위해 작성 된 용도입니다. IoT Core host를 대상으로 하는 경우이 컨테이너 이미지를 사용 해야 합니다.

대부분의 사용자는 Windows Server Core 또는 Nano 서버가 가장 적절 한 이미지를 사용 하 게 됩니다. 다음은 Nano 서버 위에 빌드할 때 고려해 야 할 사항입니다.

- 서비스 스택 제거됨
- .NET Core 미포함([.NET Core Nano 서버 이미지](https://hub.docker.com/r/microsoft/dotnet/) 사용 가능)
- PowerShell 제거됨
- WMI 제거됨
- Windows Server 버전 1709부터 응용 프로그램이 사용자 컨텍스트로 실행되므로 응용 프로그램 관리자 권한이 필요한 명령이 실패하게 됩니다. 앞으로는 NanoServer에서 관리자 계정을 완전히 제거 하기 위해--user 플래그 (예: docker 실행--사용자 ContainerAdministrator)를 통해 컨테이너 관리자 계정을 지정할 수 있습니다.

이러한 것들이 가장 큰 차이점이며 그 외에도 다른 차이점이 더 있습니다. 여기서는 다루지 않았지만 제외된 다른 구성 요소가 더 있습니다. 적합하다고 판단될 경우 언제든지 Nano 서버 위에 계층을 추가할 수 있다는 점을 기억하세요. 관련 예제는 [NET Core Nano 서버 Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)을 참조하세요.
