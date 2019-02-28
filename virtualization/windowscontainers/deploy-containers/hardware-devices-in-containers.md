---
title: Windows의 컨테이너에서 장치
description: Windows의 컨테이너에 대 한 장치 지원은 어떤
keywords: docker, 컨테이너, 장치, 하드웨어
author: cwilhit
ms.openlocfilehash: da9785b051826efa4bb2c64542a7c75a12ddd2b4
ms.sourcegitcommit: 4490d384ade48699e7f56dc265185dac75bf9d77
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/05/2019
ms.locfileid: "9058993"
---
**이 현재 미리 보기 재료입니다. 자세한 내용은 '요구 사항 섹션 아래에서 네 번째 항목이 표시 됩니다.**

# <a name="devices-in-containers-on-windows"></a>Windows의 컨테이너에서 장치

기본적으로 Windows 컨테이너 호스트 디바이스--Linux 컨테이너와 마찬가지로 최소한의 액세스를 제공 됩니다. 가지 특정 워크 로드 것이 유용한-또는 명령적-액세스 하 고 호스트 하드웨어 장치와 통신 합니다. 이 가이드에서는 컨테이너에서 지원 되는 어떤 디바이스 및 시작 하는 방법을 설명 합니다.

## <a name="requirements"></a>요구 사항

- Windows를 실행 해야 Server 2019 이상 또는 Windows 10 Pro/Enterprise 2018 년 10 월을 사용 하 여 업데이트
- 컨테이너 이미지 버전 1809 이상 이어야 합니다.
- 컨테이너는 격리 프로세스 모드에서 실행 되는 Windows 컨테이너 이어야 합니다.
- Windows 장치 기능 Docker 디먼에 있는 동안이 아직 존재 하지 않는 Docker 클라이언트에서 (이 [끌어오기 요청](https://github.com/docker/cli/pull/1606) 을 추적 참조). Windows 용 Docker의 이후 릴리스에서 기다려야 /이 기능을 활용 하려면이 코드를 사용 하 여 Docker EE 합니다. 이 문서는 상태가 변경 될 때 업데이트 됩니다.

## <a name="run-a-container-with-a-device"></a>장치를 사용 하 여 컨테이너를 실행 합니다.

장치를 사용 하 여 컨테이너를 시작 하려면 다음 명령을 사용 합니다.

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

바꾸어야 합니다 `{interface class guid}` 섹션 아래에서 찾을 수 있는 적절 한 [장치 인터페이스 클래스 guid입니다](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/overview-of-device-interface-classes).

여러 장치를 사용 하 여 컨테이너를 시작 하려면 다음 명령을 사용 하 고 문자열을 함께 여러 `--device` 인수:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Windows에서는 모든 장치 목록을 구현 하는 인터페이스 클래스를 선언 합니다. 이 명령을, Docker에 전달 하 여 요청 된 클래스 구현으로 식별 하는 모든 장치 컨테이너에 연결 됩니다 수는 보장 됩니다.

이것은 의미 **호스트에서 장치를 할당할 수** 있습니다. 대신, 호스트 컨테이너와 공유 됩니다. 마찬가지로, 클래스 GUID를 지정 하는 때문에 해당 GUID를 구현 하는 _모든_ 장치 컨테이너와 공유 되지 것입니다.

## <a name="what-devices-are-supported"></a>어떤 장치는 지원

다음 장치 (및 해당 장치 클래스 Guid 인터페이스)는 현재 지원 합니다.
  
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
</tbody>
</table>

> [!TIP]
> 위에 나열 된 장치는 현재 지원 되는 Windows 컨테이너에 _만_ 장치입니다. 다른 클래스 Guid를 전달 하는 컨테이너 시작 실패 발생 합니다.

## <a name="hyper-v-container-device-support"></a>Hyper-v 컨테이너 장치 지원

장치 할당 및 장치 공유 지원 되지 않습니다 Hyper-v 격리 된 컨테이너에서 현재.

## <a name="linux-containers-on-windows-lcow-device-support"></a>Linux 컨테이너 (LCOW) Windows 장치 지원

장치 할당 및 장치 공유 지원 되지 않습니다 LCOW에서 현재.
