---
title: "컨테이너 리소스 관리"
description: "Windows 컨테이너를 사용하여 컨테이너 리소스를 관리합니다."
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b2192e64-9d74-474e-8af0-2d8b3ad1deee
redirect_url: https://docs.docker.com/engine/reference/run/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 7544e91856d969b09241df7a466d776d818a2968

---

# 컨테이너 리소스 관리

**이 예비 콘텐츠는 변경될 수 있습니다.** 

Windows 컨테이너는 컨테이너가 사용할 수 있는 CPU, 디스크 IO, 네트워크 및 메모리 리소스의 양을 관리하는 기능을 포함합니다. 컨테이너 리소스 사용 제한을 통해 호스트 리소스를 효율적으로 사용할 수 있으며 과사용을 방지할 수 있습니다. 이 문서에서는 Docker를 사용한 컨테이너 리소스 관리에 대해 자세히 설명합니다.

## Docker를 사용하여 리소스 관리 

Docker를 통해 컨테이너 리소스의 하위 집합을 관리하는 기능을 제공합니다. 특히 사용자가 컨테이너에서 CPU가 공유되는 방법을 지정할 수 있도록 허용합니다. 

### CPU

컨테이너 간의 CPU 공유는 --cpu 공유 플래그를 통해 런타임 시 관리할 수 있습니다. 기본적으로 모든 컨테이너는 CPU 시간과 동일한 비율로 이용할 수 있습니다. 컨테이너가 사용하는 CPU의 상대 공유를 변경하려면 1-10000의 값으로 --cpu 공유 플래그를 실행합니다. 기본적으로 모든 컨테이너는 5000의 가중치를 받습니다. CPU 공유 제약 조건에 대한 자세한 내용은 [Docker Run Reference(Docker Run 참조)]( https://docs.docker.com/engine/reference/run/#cpu-share-constraint)를 참조하세요. 

```none 
docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## 알려진 문제

- CPU 및 IO 리소스 컨트롤은 현재 Hyper-V 컨테이너에서 지원되지 않습니다.
- IO 리소스 컨트롤은 현재 컨테이너 데이터 볼륨에서 지원되지 않습니다.


<!--HONumber=Sep16_HO2-->


