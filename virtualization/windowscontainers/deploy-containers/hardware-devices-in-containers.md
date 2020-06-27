---
title: Windows 컨테이너의 디바이스
description: Windows 컨테이너를 지원하는 디바이스
keywords: docker, 컨테이너, 디바이스, 하드웨어
author: cwilhit
ms.topic: how-to
ms.openlocfilehash: bef8e3236294588e38d7bff235ed1d3a98278375
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192179"
---
# <a name="devices-in-containers-on-windows"></a>Windows 컨테이너의 디바이스

기본적으로 Windows 컨테이너에는 Linux 컨테이너와 마찬가지로 호스트 디바이스에 대한 최소한의 액세스 권한만 부여됩니다. 호스트 하드웨어 디바이스에 액세스하고 통신할 때 도움이 되는 또는 반드시 필요한 특정 워크로드가 있습니다. 이 가이드에서는 컨테이너에서 지원되는 디바이스와 시작하는 방법을 설명합니다.

## <a name="requirements"></a>요구 사항

이 기능이 작동하려면 환경이 다음 요구 사항을 충족해야 합니다.
- 컨테이너 호스트에서 Windows Server 2019 또는 Windows 10 버전 1809 이상을 실행해야 합니다.
- 컨테이너 기본 이미지 버전이 1809 이상이어야 합니다.
- 컨테이너는 프로세스 격리 모드에서 실행되는 Windows 컨테이너여야 합니다.
- 컨테이너 호스트에서 Docker 엔진 19.03 이상 버전을 실행해야 합니다.

## <a name="run-a-container-with-a-device"></a>디바이스에서 컨테이너 실행

디바이스에서 컨테이너를 시작하려면 다음 명령을 사용합니다.

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

`{interface class guid}`를 아래 섹션에서 찾을 수 있는 적절한 [디바이스 인터페이스 클래스 GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)로 바꿔야 합니다.

여러 디바이스에서 컨테이너를 시작하려면 다음 명령과 문자열을 여러 `--device` 인수와 함께 사용합니다.

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Windows에서는 모든 디바이스가 구현하는 인터페이스 클래스 목록을 선언합니다. 이 명령을 Docker에 전달하면 요청된 클래스를 구현하는 것으로 식별되는 모든 디바이스가 컨테이너로 전달됩니다.

즉, 호스트의 디바이스를 할당하는 것이 **아니라** 호스트가 컨테이너와 디바이스를 공유하는 것입니다. 마찬가지로, 클래스 GUID를 지정하는 경우 해당 GUID를 구현하는 _모든_ 디바이스를 컨테이너와 공유하게 됩니다.

## <a name="what-devices-are-supported"></a>지원되는 디바이스

현재 지원되는 디바이스(및 디바이스 인터페이스 클래스 GUID)는 다음과 같습니다.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>디바이스 유형</center></th>
<th><center>인터페이스 클래스 GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C Bus</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM 포트</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI Bus</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU 가속화</center></td>
<td><center><a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU 가속화</a> 문서 참조</center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> 디바이스 지원은 드라이버에 따라 다릅니다. 위의 표에 정의되지 않은 클래스 GUID를 전달하려고 시도하면 정의되지 않은 동작이 발생할 수 있습니다.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V 격리 Windows 컨테이너 지원

Hyper-V 격리 Windows 컨테이너의 워크로드에 대한 디바이스 할당 및 디바이스 공유는 현재 지원되지 않습니다.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V 격리 Linux 컨테이너 지원

Hyper-V 격리 Linux 컨테이너의 워크로드에 대한 디바이스 할당 및 디바이스 공유는 현재 지원되지 않습니다.
