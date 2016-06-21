---
title: HCS PowerSWhell
description: HCS PowerShell 및 Windows 컨테이너를 사용합니다.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 45144ec5-f76a-4460-abd1-9b60e47506d6
---

# 관리 상호 운용성

**이 예비 콘텐츠는 변경될 수 있습니다.** 

## 모든 컨테이너 표시

컨테이너 목록을 반환하려면 `Get-ComputeProcess` 명령을 사용합니다.

```none
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## 컨테이너 중지

만든 방법이 PowerShell인지 Docker인지에 관계없이 컨테이너를 중지하려면 `Stop-ComputeProcess` 명령을 사용합니다.

> 작성 시점 현재 `Get-Container` 명령을 사용할 때 컨테이너가 중지된 것으로 표시되게 하려면 VMMS 서비스를 다시 시작해야 합니다.

```none
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```


<!--HONumber=May16_HO3-->


