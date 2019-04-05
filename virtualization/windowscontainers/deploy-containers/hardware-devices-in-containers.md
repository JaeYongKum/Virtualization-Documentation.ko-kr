---
title: Windows의 컨테이너에서 장치
description: Windows의 컨테이너에 대 한 장치 지원은 어떤
keywords: docker, 컨테이너, 장치, 하드웨어
author: cwilhit
ms.openlocfilehash: 18ae4ab229a677c63c3e17d684a3c3193df49c5e
ms.sourcegitcommit: 3c81b0efd1ac2c4c93d58f16edae1044c9a5ad55
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/05/2019
ms.locfileid: "9284597"
---
# <a name="devices-in-containers-on-windows"></a>Windows의 컨테이너에서 장치

기본적으로 Windows 컨테이너 호스트 디바이스--Linux 컨테이너와 마찬가지로 최소한의 액세스를 제공 됩니다. 가지 특정 워크 로드 것이 유용한-또는 명령적-액세스 하 고 호스트 하드웨어 장치와 통신 합니다. 이 가이드에서는 컨테이너에서 지원 되는 어떤 디바이스 및 시작 하는 방법을 설명 합니다.

> [!IMPORTANT]
> 이 기능을 지 원하는 Docker 버전이 필요 합니다 `--device` Windows 컨테이너에 대 한 명령줄 옵션입니다. 공식 Docker 지원 예정 된 다음 Docker EE 엔진 19.03 릴리스에 대 한 합니다. 그 때까지 Docker에 대 한 [업스트림 소스](https://master.dockerproject.org/) 에 필요한 비트 포함 되어 있습니다.

## <a name="requirements"></a>요구 사항

이 기능이 제대로 작동 하도록 환경에는 다음 요구 사항을 충족 해야 합니다.
- Windows 10, 버전 1809 이상 또는 Windows Server 2019 컨테이너 호스트를 실행 해야 합니다.
- 컨테이너 기본 이미지 버전 1809 이상 이어야 합니다.
- 컨테이너는 격리 프로세스 모드에서 실행 되는 Windows 컨테이너 이어야 합니다.
- 컨테이너 호스트 19.03 또는 최신 Docker 엔진을 실행 되어야 합니다.

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
<tr valign="top">
<td><center>DirectX GPU 가속</center></td>
<td><center>전용된 문서를 참조 하세요.</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> 위에 나열 된 장치는 현재 지원 되는 Windows 컨테이너에 _만_ 장치입니다. 다른 클래스 Guid를 전달 하는 컨테이너 시작 실패 발생 합니다.

## <a name="hyper-v-isolated-windows-container-support"></a>V-격리 하이퍼 Windows 컨테이너 지원

장치 할당 및 장치 공유 하이퍼 V 격리 Windows 컨테이너에서 워크 로드에 대 한 현재 지원 되지 않습니다.

## <a name="hyper-v-isolated-linux-container-support"></a>V-격리 하이퍼 Linux 컨테이너 지원

장치 할당 및 장치 공유 하이퍼 V 격리 Linux 컨테이너에서 워크 로드에 대 한 현재 지원 되지 않습니다.
