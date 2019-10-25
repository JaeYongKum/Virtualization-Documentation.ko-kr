---
title: Windows 참가자 프로그램에서 컨테이너 사용
description: Windows 참가자 프로그램으로 windows 컨테이너를 사용 하 여 시작 하는 방법 알아보기
keywords: docker, 컨테이너, 참가자, windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 6080b2c5053720490d374f6fb0daa870d5ddd4e8
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257797"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Windows 참가자 프로그램에서 컨테이너 사용

이 연습에서는 Windows Insider Preview 프로그램의 최신 Windows Server 참가자 빌드에 Windows 컨테이너 기능을 배포하고 사용하는 방법을 안내합니다. 이 연습을 진행하는 과정에서 컨테이너 역할을 설치하고 기본 OS 이미지의 미리 보기 버전을 배포할 것입니다. 컨테이너에 대해 좀 더 숙지해야 하는 경우 [컨테이너 정보](../about/index.md)에서 이 정보를 찾을 수 있습니다.

## <a name="join-the-windows-insider-program"></a>Windows 참가자 프로그램에 참여하기

참가자 버전의 Windows 컨테이너를 실행 하려면 windows 참가자 프로그램 및/또는 windows 참가자 프로그램에서 windows 10의 최신 빌드를 실행 하는 windows Server의 최신 빌드를 사용 하는 호스트가 있어야 합니다. [Windows 참가자 프로그램](https://insider.windows.com/GettingStarted) 에 참여 하 고 사용 약관을 검토 합니다.

> [!IMPORTANT]
> Windows Insider preview 프로그램에서 windows server 참가자 미리 보기 프로그램 또는 windows 10의 빌드를 사용 하 여 아래 설명 된 기본 이미지를 사용 해야 합니다. 두 빌드 중 하나를 사용하지 않을 경우 이러한 기본 이미지를 사용하면 컨테이너가 시작되지 않는 오류가 발생합니다.

## <a name="install-docker"></a>Docker 설치

Docker가 이미 설치 되어 있지 않은 경우 [시작](../quick-start/set-up-environment.md) 가이드를 따라 docker를 설치 하세요.

## <a name="pull-an-insider-container-image"></a>참가자 컨테이너 이미지 가져오기

Windows 참가자 프로그램의 일부인 경우 기본 이미지에 대 한 최신 빌드를 사용할 수 있습니다.

Nano 서버 참가자 기본 이미지를 가져오려면 다음을 실행합니다.

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Windows Server Core 참가자 기본 이미지를 가져오려면 다음을 실행합니다.

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

"Windows" 및 "IoTCore" 기본 이미지에는 끌어오기에 사용할 수 있는 참가자 버전도 있습니다. [컨테이너 기본 이미지](../manage-containers/container-base-images.md) 문서에서 사용 가능한 참가자 기본 이미지에 대 한 자세한 내용을 확인할 수 있습니다.

> [!IMPORTANT]
> Windows 컨테이너 OS 이미지 [EULA](../images-eula.md ) 및 windows 참가자 프로그램 [사용 약관](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)을 참조 하세요.
