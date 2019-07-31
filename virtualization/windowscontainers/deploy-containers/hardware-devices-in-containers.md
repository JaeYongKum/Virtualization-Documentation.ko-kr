---
title: Windows 용 컨테이너의 장치
description: Windows의 컨테이너에 대해 어떤 장치 지원이 존재 함
keywords: docker, 컨테이너, 장치, 하드웨어
author: cwilhit
ms.openlocfilehash: ee9c5da5ef87dceb3374977670da2ea50ea87382
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883166"
---
# <a name="devices-in-containers-on-windows"></a>Windows 용 컨테이너의 장치

기본적으로 Windows 컨테이너에는 Linux 컨테이너와 같이 호스트 장치에 대 한 최소 액세스 권한이 부여 됩니다. 도움이 되는 특정 작업 부하와 호스트 하드웨어 장치에 액세스 하 고 통신 하는 데에는 필수적입니다. 이 가이드에서는 컨테이너에서 지원 되는 장치와 시작 하는 방법에 대해 설명 합니다.

## <a name="requirements"></a>요구 사항

이 기능이 작동 하려면 환경이 다음 요구 사항을 충족 해야 합니다.
- 컨테이너 호스트는 Windows Server 2019 또는 Windows 10, 버전 1809 이상을 실행 중 이어야 합니다.
- 컨테이너 기본 이미지 버전은 1809 이상 이어야 합니다.
- 컨테이너는 프로세스 격리 모드에서 실행 되는 Windows 컨테이너 여야 합니다.
- 컨테이너 호스트는 Docker 엔진 19.03 이상 버전을 실행 중 이어야 합니다.

## <a name="run-a-container-with-a-device"></a>장치를 사용 하 여 컨테이너 실행

장치에서 컨테이너를 시작 하려면 다음 명령을 사용 합니다.

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

는 `{interface class guid}` 다음 섹션에서 찾을 수 있는 적절 한 [디바이스 인터페이스 클래스 GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)로 바꿔야 합니다.

여러 디바이스를 사용 하 여 컨테이너를 시작 하려면 다음 명령을 사용 하 고 여러 `--device` 인수를 문자열로 결합 합니다.

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Windows에서 모든 장치는 구현 하는 인터페이스 클래스 목록을 선언 합니다. 이 명령을 Docker에 전달 하면 요청 된 클래스를 구현 하는 것으로 식별 되는 모든 장치를 컨테이너에 배관 하 게 됩니다.

이는 사용자가 호스트에서 장치를 멀리 떨어진 곳에 할당 **하지** 않았다는 의미입니다. 대신 호스트가 컨테이너와 공유 합니다. 마찬가지로, 클래스 GUID를 지정 하는 것 이기 때문에 해당 GUID를 구현 하는 _모든_ 장치가 컨테이너와 공유 됩니다.

## <a name="what-devices-are-supported"></a>지원 되는 장치

다음 장치 및 해당 디바이스 인터페이스 클래스 Guid가 현재 지원 됩니다.
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>장치 유형</center></th>
<th><center>인터페이스 클래스 GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C 버스</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM 포트</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI 버스</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU 가속</center></td>
<td><center><a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU 가속</a> 문서 참조</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> 위에 나열 된 장치는 현재 __ Windows 컨테이너에서 지원 되는 장치입니다. 다른 클래스 Guid를 전달 하려고 하면 컨테이너가 시작 하지 못할 수 있습니다.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-격리 된 Windows 컨테이너 지원

현재 Hyper-v 격리 Windows 컨테이너의 작업 부하에 대 한 장치 할당 및 장치 공유는 지원 되지 않습니다.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-isolated Linux 컨테이너 지원

현재 Hyper-v 격리 Linux 컨테이너의 작업 부하에 대 한 장치 할당 및 장치 공유는 지원 되지 않습니다.