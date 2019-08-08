---
title: Hyper-V 격리
description: Hyper-v 격리가 프로세스 격리 컨테이너와 어떻게 다른 지를 Explaination.
keywords: Docker, 컨테이너
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 092312848173102bec5a791f2c48fe8166e70d5f
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998330"
---
# <a name="hyper-v-isolation"></a>Hyper-V 격리

Windows 컨테이너 기술에는 컨테이너, 프로세스 및 Hyper-v 격리에 대 한 두 가지 고유한 격리 수준이 포함 됩니다. 두 형식 모두 동일한 방식으로 생성, 관리 및 작동 합니다. 또한 동일한 컨테이너 이미지를 만들고 사용합니다. 차이점은 컨테이너, 호스트 운영 체제 및 해당 호스트에서 실행되는 다른 모든 컨테이너 간에 만들어지는 격리의 수준입니다.

**프로세스 격리** – 네임 스페이스, 리소스 컨트롤, 프로세스 격리 기술을 통해 제공 되는 격리를 사용 하 여 호스트에서 여러 컨테이너 인스턴스가 동시에 실행 될 수 있습니다.  컨테이너는 동일한 커널을 호스트와 함께 공유 합니다.  이는 대략 Linux에서 컨테이너를 실행 하는 방법과 같습니다.

**Hyper-v 격리** – 여러 컨테이너 인스턴스가 호스트에서 동시에 실행 될 수 있지만, 각 컨테이너는 특별 한 가상 컴퓨터 내에서 실행 됩니다. 이를 통해 컨테이너 호스트와 각 컨테이너 간의 커널 수준 격리를 제공할 수도 있습니다.

## <a name="hyper-v-isolation-examples"></a>Hyper-v 격리 예제

### <a name="create-container"></a>컨테이너 만들기

Docker를 사용 하 여 Hyper-v 격리 컨테이너를 관리 하는 것은 Windows Server 컨테이너 관리와 거의 동일 합니다. Hyper-v 격리 정밀 Docker를 사용 하 여 컨테이너를 만들려면 `--isolation` 매개 변수를 사용 하 여 `--isolation=hyperv`설정 합니다.

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>격리 설명

이 예제에서는 Windows Server와 Hyper-v 격리 간의 격리 기능 차이를 보여 줍니다.

여기에서 격리 된 프로세스 컨테이너는 배포 되 고 있으며 장기 실행 ping 프로세스를 호스트 합니다.

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

이 예제에서는 또한 ping 프로세스를 사용 하 여 Hyper-v 격리 컨테이너를 시작 합니다.

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
