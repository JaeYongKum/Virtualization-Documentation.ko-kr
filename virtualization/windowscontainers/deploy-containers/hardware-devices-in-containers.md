---
title: Windows 컨테이너의 장치
description: Windows의 컨테이너에 대해 존재 하는 장치 지원
keywords: docker, 컨테이너, 장치, 하드웨어
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910603"
---
# <a name="devices-in-containers-on-windows"></a>Windows 컨테이너의 장치

기본적으로 Windows 컨테이너에는 Linux 컨테이너와 마찬가지로 호스트 장치에 대 한 최소한의 액세스 권한이 부여 됩니다. 호스트 하드웨어 장치에 액세스 하 고이를 전달 하는 데 도움이 되는 특정 워크 로드 또는 명령적 작업이 있습니다. 이 가이드에서는 컨테이너에서 지원 되는 장치와 시작 하는 방법을 설명 합니다.

## <a name="requirements"></a>요구 사항

이 기능이 작동 하려면 환경이 다음 요구 사항을 충족 해야 합니다.
- 컨테이너 호스트에서 Windows Server 2019 또는 Windows 10 버전 1809 이상이 실행 되 고 있어야 합니다.
- 컨테이너 기본 이미지 버전은 1809 이상 이어야 합니다.
- 컨테이너는 프로세스 격리 모드에서 실행 되는 Windows 컨테이너 여야 합니다.
- 컨테이너 호스트에서 Docker 엔진 19.03 이상 버전을 실행 해야 합니다.

## <a name="run-a-container-with-a-device"></a>장치를 사용 하 여 컨테이너 실행

장치를 사용 하 여 컨테이너를 시작 하려면 다음 명령을 사용 합니다.

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

`{interface class guid}`는 아래 섹션에서 찾을 수 있는 적절 한 [장치 인터페이스 클래스 GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)로 바꾸어야 합니다.

여러 장치를 사용 하 여 컨테이너를 시작 하려면 다음 명령과 여러 `--device` 인수를 함께 사용 합니다.

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Windows에서 모든 장치는 구현 하는 인터페이스 클래스 목록을 선언 합니다. 이 명령을 Docker에 전달 하 여 요청 된 클래스를 구현 하는 것으로 식별 되는 모든 장치를 컨테이너에 배관 하는지 확인 합니다.

즉, 호스트에서 장치를 할당 **하지 않습니다** . 대신 호스트가 컨테이너와 공유 하 고 있습니다. 마찬가지로, guid를 지정 하기 때문에 해당 GUID를 구현 하는 _모든_ 장치는 컨테이너와 공유 됩니다.

## <a name="what-devices-are-supported"></a>지원 되는 장치

현재 지원 되는 장치 및 장치 인터페이스 클래스 Guid는 다음과 같습니다.
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Device Type</center></th>
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

> [!IMPORTANT]
> 장치 지원은 드라이버에 따라 다릅니다. 위의 표에 정의 되어 있지 않은 클래스 Guid를 전달 하려고 하면 정의 되지 않은 동작이 발생할 수 있습니다.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v 격리 Windows 컨테이너 지원

Hyper-v 격리 Windows 컨테이너에서 작업에 대 한 장치 할당 및 장치 공유는 현재 지원 되지 않습니다.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v 격리 Linux 컨테이너 지원

Hyper-v 격리 Linux 컨테이너에서 작업에 대 한 장치 할당 및 장치 공유는 현재 지원 되지 않습니다.
