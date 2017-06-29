---
title: "Hyper-V 컨테이너"
description: "Hyper-V 컨테이너를 배포합니다."
keywords: "Docker, 컨테이너"
author: enderb-ms
ms.date: 09/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 88c452117299d855073668d33e4323243d93e9d1
ms.sourcegitcommit: fa56cb152977de952ec1e493005dfc0e62538b4f
ms.translationtype: HT
ms.contentlocale: ko-KR
---
# <a name="hyper-v-containers"></a>Hyper-V 컨테이너

**이 예비 콘텐츠는 변경될 수 있습니다.** 

Windows 컨테이너 기술은 Windows Server 컨테이너와 Hyper-V 컨테이너 등 두 가지 다른 유형의 컨테이너를 포함합니다. 두 컨테이너 유형 모두 생성, 관리, 작동은 동일합니다. 또한 동일한 컨테이너 이미지를 만들고 사용합니다. 차이점은 컨테이너, 호스트 운영 체제 및 해당 호스트에서 실행되는 다른 모든 컨테이너 간에 만들어지는 격리의 수준입니다.

**Windows Server 컨테이너** – 네임스페이스, 리소스 제어 및 프로세스 격리 기술을 통해 제공되는 격리를 통해 호스트에서 여러 컨테이너 인스턴스를 동시에 실행할 수 있습니다.  Windows Server 컨테이너는 동일한 커널을 서로 공유하고 호스트와도 공유합니다.

**Hyper-V 컨테이너** – 호스트에서 여러 컨테이너 인스턴스를 동시에 실행할 수 있지만 각 컨테이너는 특별한 가상 컴퓨터 안에서 실행됩니다. 이를 통해 각 Hyper-V 컨테이너와 컨테이너 호스트 사이에 커널 수준 격리가 제공됩니다.

## <a name="hyper-v-container"></a>Hyper-V 컨테이너

### <a name="create-container"></a>컨테이너 만들기

Docker로 Hyper-V 컨테이너를 관리하는 것은 Windows Server 컨테이너 관리와 거의 동일합니다. Docker로 Hyper-V 컨테이너를 만들 때 `--isolation=hyperv` 매개 변수를 사용합니다.

```none
docker run -it --isolation=hyperv microsoft/nanoserver cmd
```

### <a name="isolation-explanation"></a>격리 설명

이 예제에서는 Windows Server 및 Hyper-V 컨테이너 간의 격리 기능을 구별합니다. 

여기에서는 Windows Server 컨테이너를 배포하고 장기 실행 ping 프로세스를 호스트합니다.

```none
docker run -d microsoft/windowsservercore ping localhost -t
```

`docker top` 명령을 사용하여 컨테이너 내부에 표시된 대로 ping 프로세스를 반환합니다. 이 예제의 프로세스에서는 ID 3964를 사용합니다.

```none
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

컨테이너 호스트에서는 `get-process` 명령을 사용하여 호스트에서 실행 중인 ping 프로세스를 반환할 수 있습니다. 다음 예제에는 하나가 있으며 프로세스 ID가 컨테이너와 일치합니다. 컨테이너와 호스트에서 모두 표시되는 동일한 프로세스입니다.

```none
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

반대로 이 예제에서는 ping 프로세스로 Hyper-V 컨테이너를 시작합니다. 

```none
docker run -d --isolation=hyperv microsoft/nanoserver ping -t localhost
```

마찬가지로 `docker top`을 사용하여 컨테이너에서 실행 중인 프로세스를 반환할 수 있습니다.

```none
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

그러나 컨테이너 호스트에서 프로세스를 검색하면 ping 프로세스가 검색되지 않고 오류가 throw됩니다.

```none
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

마지막으로 호스트에서 `vmwp` 프로세스가 표시되는데, 이는 실행 중인 컨테이너를 캡슐화하고 호스트 운영 체제에서 실행 중인 프로세스를 보호하는 실행 중인 가상 컴퓨터입니다.

```none
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
