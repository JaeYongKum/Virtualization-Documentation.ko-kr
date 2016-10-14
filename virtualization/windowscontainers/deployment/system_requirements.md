---
title: "Windows의 컨테이너 요구 사항"
description: "Windows의 컨테이너 요구 사항입니다."
keywords: "메타데이터, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: 08355d7a7da50d0f244bd750508fd42084818d7f
ms.openlocfilehash: ac48bc7ee7b70483d8a368749aea0862c52f049c

---

# Windows의 컨테이너 요구 사항

이 가이드는 Windows 컨테이너 호스트에 대한 요구 사항을 나열합니다.

## OS 요구 사항

- Windows 컨테이너 기능은 Windows Server 2016(코어 및 데스크톱 경험 포함), Nano Server, Windows 10 Professional 및 Enterprise(Anniversary Edition)에서만 사용할 수 있습니다.
- Hyper-V 컨테이너가 실행되면 Hyper-V 역할이 설치되어야 합니다.
- Windows Server 컨테이너 호스트에는 c:\.에 Windows가 설치되어 있어야 합니다. Hyper-V 컨테이너만 배포할 경우 이 제한이 적용되지 않습니다.

## 가상화된 컨테이너 호스트

Windows 컨테이너 호스트가 Hyper-V 가상 컴퓨터에서 실행되고 Hyper-V 호스트 컨테이너를 호스팅할 경우 중첩된 가상화를 사용해야 합니다. 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- Windows Server 2016 또는 호스트 시스템의 Windows 10, 가상 컴퓨터의 Windows Server(Full, Core) 또는 Nano Server.
- Intel VT-x가 포함된 프로세서.(이 기능은 현재 Intel 프로세서에 대해서만 사용할 수 있습니다)
- 또한 컨테이너 호스트 VM에는 적어도 2개의 가상 프로세서가 필요합니다.

## 지원되는 기본 이미지

Windows 컨테이너는 두 컨테이너 기본 이미지(Windows Server Core 및 Nano Server)와 함께 제공됩니다. 일부 구성은 두 개의 OS 이미지를 모두 지원하지 않습니다. 이 테이블은 지원되는 구성을 자세히 설명합니다.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>호스트 운영 체제</center></th>
<th><center>Windows Server 컨테이너</center></th>
<th><center>Hyper-V 컨테이너</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 데스크톱</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro/Enterprise</center></td>
<td><center>사용할 수 없음</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
</tbody>
</table>


<!--HONumber=Oct16_HO2-->


