---
title: 격리 모드
description: Hyper-v 격리가 프로세스 격리 컨테이너와 어떻게 다른 지에 대 한 설명입니다.
keywords: Docker, 컨테이너
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 362805fa230f461414ccc53643644f6c1b3474a8
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853957"
---
# <a name="isolation-modes"></a>격리 모드

Windows 컨테이너는 두 가지 고유한 런타임 격리 모드인 `process`와 `Hyper-V` 격리를 제공 합니다. 두 격리 모드에서 실행 되는 컨테이너는 동일 하 게 생성, 관리 및 작동 합니다. 또한 동일한 컨테이너 이미지를 만들고 사용합니다. 격리 모드의 차이점은 컨테이너, 호스트 운영 체제 및 해당 호스트에서 실행 되는 다른 모든 컨테이너 간에 만들어지는 격리 수준을 결정 하는 것입니다.

## <a name="process-isolation"></a>프로세스 격리

이는 컨테이너에 대 한 "기존" 격리 모드 이며 [Windows 컨테이너 개요](../about/index.md)에 설명 되어 있습니다. 프로세스 격리를 사용 하면 네임 스페이스, 리소스 제어 및 프로세스 격리 기술을 통해 제공 되는 격리를 통해 지정 된 호스트에서 여러 컨테이너 인스턴스가 동시에 실행 됩니다. 이 모드에서 실행 하는 경우 컨테이너는 호스트와 동일한 커널을 공유 합니다.  이는 Linux 컨테이너가 실행 되는 방법과 거의 동일 합니다.

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V 격리
이 격리 모드는 호스트와 컨테이너 버전 간의 향상 된 보안 및 광범위 한 호환성을 제공 합니다. Hyper-v 격리를 사용 하면 여러 컨테이너 인스턴스가 호스트에서 동시에 실행 됩니다. 그러나 각 컨테이너는 최적화 된 가상 머신 내에서 실행 되며 효과적으로 자체 커널을 가져옵니다. 가상 머신이 있으면 각 컨테이너와 컨테이너 호스트 간의 하드웨어 수준 격리가 제공 됩니다.

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>격리 예

### <a name="create-container"></a>컨테이너 만들기

Docker를 사용 하 여 Hyper-v 격리 컨테이너를 관리 하는 것은 프로세스 격리 컨테이너를 관리 하는 것과 거의 동일 합니다. Hyper-v 격리 정밀 Docker를 사용 하 여 컨테이너를 만들려면 `--isolation` 매개 변수를 사용 하 여 `--isolation=hyperv`를 설정 합니다.

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Docker를 통해 프로세스 격리를 사용 하 여 컨테이너를 만들려면 `--isolation` 매개 변수를 사용 하 여 `--isolation=process`를 설정 합니다.

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows Server에서 실행 되는 windows 컨테이너는 기본적으로 프로세스 격리를 사용 하 여 실행 됩니다. Windows 10 Pro 및 Enterprise에서 실행 되는 windows 컨테이너는 기본적으로 Hyper-v 격리로 실행 됩니다. Windows 10 10 월 2018 업데이트부터 Windows 10 Pro 또는 Enterprise 호스트를 실행 하는 사용자는 프로세스 격리가 포함 된 Windows 컨테이너를 실행할 수 있습니다. 사용자는 `--isolation=process` 플래그를 사용 하 여 직접 프로세스 격리를 요청 해야 합니다.

> [!WARNING]
> Windows 10 Pro 및 Enterprise에서 프로세스 격리를 사용 하 여를 실행 하는 것은 개발/테스트를 위한 것입니다. 호스트에서 Windows 10 build 17763 +를 실행 해야 하며, 18.09 이상의 Docker 버전이 있어야 합니다.
> 
> 프로덕션 배포를 위한 호스트로 Windows Server를 계속 사용 해야 합니다. Windows 10 Pro 및 Enterprise에서이 기능을 사용 하 여 호스트 및 컨테이너 버전 태그가 일치 하는지 확인 해야 합니다. 그렇지 않으면 컨테이너를 시작할 수 없거나 정의 되지 않은 동작이 발생 합니다.

### <a name="isolation-explanation"></a>격리 설명

이 예제에서는 프로세스와 Hyper-v 격리 간의 격리 기능 차이점을 보여 줍니다.

여기에서 프로세스 격리 컨테이너는 배포 중 이며 장기 실행 ping 프로세스를 호스트 합니다.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

`docker top` 명령을 사용하여 컨테이너 내부에 표시된 대로 ping 프로세스를 반환합니다. 이 예제의 프로세스에서는 ID 3964를 사용합니다.

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

컨테이너 호스트에서는 `get-process` 명령을 사용하여 호스트에서 실행 중인 ping 프로세스를 반환할 수 있습니다. 다음 예제에는 하나가 있으며 프로세스 ID가 컨테이너와 일치합니다. 컨테이너와 호스트에서 모두 표시되는 동일한 프로세스입니다.

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

이와 대조적으로이 예제에서는 ping 프로세스를 사용 하 여 Hyper-v와 관련 된 컨테이너를 시작 합니다.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

마찬가지로 `docker top`을 사용하여 컨테이너에서 실행 중인 프로세스를 반환할 수 있습니다.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

그러나 컨테이너 호스트에서 프로세스를 검색할 때 ping 프로세스를 찾을 수 없으며 오류가 throw 됩니다.

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

마지막으로 호스트에서 `vmwp` 프로세스가 표시되는데, 이는 실행 중인 컨테이너를 캡슐화하고 호스트 운영 체제에서 실행 중인 프로세스를 보호하는 실행 중인 가상 컴퓨터입니다.

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
