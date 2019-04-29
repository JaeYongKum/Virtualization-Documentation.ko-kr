---
title: Hyper-V 격리
description: Hyper-v 격리 프로세스 격리 된 컨테이너에서 어떻게 다른 지 설명은 합니다.
keywords: Docker, 컨테이너
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 2ff2d1204e1f973d49af5e1d4441e4eacd946101
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576904"
---
# <a name="hyper-v-isolation"></a>Hyper-V 격리

Windows 컨테이너 기술은 두 가지 수준의 격리 컨테이너, 프로세스 및 Hyper-v 격리를 포함합니다. 두 유형 모두 생성, 관리, 작동은 동일 합니다. 또한 동일한 컨테이너 이미지를 만들고 사용합니다. 차이점은 컨테이너, 호스트 운영 체제 및 해당 호스트에서 실행되는 다른 모든 컨테이너 간에 만들어지는 격리의 수준입니다.

**프로세스 격리** – 여러 컨테이너 인스턴스를 격리를 사용 하 여 호스트에서 동시에 실행할 수를 통해 제공 네임 스페이스, 리소스 제어 및 프로세스 격리 기술을.  컨테이너는 호스트와도 서로 동일한 커널을 공유 합니다.  이 값은 대략 동일한 linux 컨테이너를 실행 하는 방법입니다.

**Hyper-v 격리** – 각 컨테이너 특별 한 가상 컴퓨터 내에서 실행 되는 반면, 여러 컨테이너 인스턴스를 호스트에서 동시에 실행할 수 있습니다. 각 컨테이너 뿐만 아니라 컨테이너 호스트 사이 커널 수준 격리가 제공 됩니다.

## <a name="hyper-v-isolation-examples"></a>Hyper-v 격리 예제

### <a name="create-container"></a>컨테이너 만들기

Docker로 Hyper-v 격리 된 컨테이너를 관리 하는 것은 Windows Server 컨테이너 관리와 거의 동일 합니다. Hyper-v 격리를 사용 하 여 컨테이너를 만드는 철저 한 Docker를 사용 하는 `--isolation` 매개 변수를 설정 `--isolation=hyperv`.

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>격리 설명

이 예제에서는 Windows Server 및 Hyper-v 격리 간의 격리 기능 차이 보여 줍니다.

여기에서는 프로세스 격리 된 컨테이너를 배포 하 고 장기 실행 ping 프로세스를 호스트 합니다.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
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

반대로이 예제에서는 ping 프로세스에 Hyper-v 격리 된 컨테이너를 시작 합니다.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 ping -t localhost
```

마찬가지로 `docker top`을 사용하여 컨테이너에서 실행 중인 프로세스를 반환할 수 있습니다.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

그러나 컨테이너 호스트에서 프로세스를 검색하면 ping 프로세스가 검색되지 않고 오류가 throw됩니다.

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
