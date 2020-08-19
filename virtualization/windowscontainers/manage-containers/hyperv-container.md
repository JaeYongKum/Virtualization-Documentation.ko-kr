---
title: 격리 모드
description: Hyper-V 격리가 프로세스 격리 컨테이너와 어떻게 다른지 설명합니다.
keywords: Docker, 컨테이너
author: cwilhit
ms.date: 09/26/2019
ms.topic: conceptual
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: c7bcb25b2c3b65be745971ae2dec4d509266a1b3
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161952"
---
# <a name="isolation-modes"></a>격리 모드

Windows 컨테이너는 두 가지 고유한 런타임 격리 모드인 `process` 및 `Hyper-V` 격리를 제공합니다. 두 격리 모드에서 실행되는 컨테이너는 동일하게 생성, 관리 및 작동합니다. 또한 동일한 컨테이너 이미지를 만들고 사용합니다. 두 격리 모드의 차이점은 컨테이너, 호스트 운영 체제, 해당 호스트에서 실행되는 다른 모든 컨테이너 간에 만들어지는 격리의 수준입니다.

## <a name="process-isolation"></a>프로세스 격리

컨테이너의 "전통적인" 격리 모드이며 [Windows 컨테이너 개요](../about/index.md)에 설명되어 있습니다. 프로세스 격리 모드에서는 네임스페이스, 리소스 제어 및 프로세스 격리 기술을 통해 제공되는 격리를 사용하여 여러 컨테이너 인스턴스가 지정된 호스트에서 동시에 실행됩니다. 이 모드에서 실행할 때 컨테이너는 동일한 커널을 서로 공유하고 호스트와도 공유합니다.  이는 Linux 컨테이너가 실행되는 방식과 거의 동일합니다.

![OS와 하드웨어에서 격리되는 애플리케이션으로 가득 찬 컨테이너를 보여 주는 다이어그램입니다.](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V 격리
이 격리 모드는 호스트 버전과 컨테이너 버전 간에 향상된 보안과 광범위한 호환성을 제공합니다. Hyper-V 격리를 사용하면 여러 컨테이너 인스턴스가 호스트에서 동시에 실행되지만, 각 컨테이너는 고도로 최적화된 가상 머신 내에서 실행되며 효과적으로 자체 커널을 가져옵니다. 가상 머신이 있으면 각 컨테이너와 컨테이너 호스트 간에 하드웨어 수준 격리가 제공됩니다.

![물리적 머신 내의 OS에서 실행 중인 비주얼 머신의 OS 내에서 분리된 컨테이너의 다이어그램입니다.](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>격리 예제

### <a name="create-container"></a>컨테이너 만들기

Docker를 사용하여 Hyper-V 격리 컨테이너를 관리하는 방법은 프로세스 격리 컨테이너를 관리하는 방법과 거의 동일합니다. Docker를 통해 Hyper-V 격리를 사용하여 컨테이너를 만들려면 `--isolation` 매개 변수를 `--isolation=hyperv`로 설정합니다.

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Docker를 통해 프로세스 격리를 사용하여 컨테이너를 만들려면 `--isolation` 매개 변수를 `--isolation=process`로 설정합니다.

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows Server에서 실행되는 Windows 컨테이너는 기본적으로 프로세스 격리를 사용하여 실행됩니다. Windows 10 Pro 및 Enterprise에서 실행되는 Windows 컨테이너는 기본적으로 Hyper-V 격리를 사용하여 실행됩니다. Windows 10 2018년 10월 업데이트부터 Windows 10 Pro 또는 Enterprise 호스트를 실행하는 사용자는 프로세스 격리를 사용하는 Windows 컨테이너를 실행할 수 있습니다. 사용자는 `--isolation=process` 플래그를 사용하여 직접 프로세스 격리를 요청해야 합니다.

> [!WARNING]
> Windows 10 Pro 및 Enterprise에서 프로세스 격리를 사용하여 실행하는 목적은 개발/테스트입니다. 호스트에서 Windows 10 빌드 17763 이상을 실행해야 하며, 엔진 18.09 이상을 사용하는 Docker 버전이 있어야 합니다.
>
> 프로덕션 배포를 위한 호스트로는 Windows Server를 계속 사용해야 합니다. Windows 10 Pro 및 Enterprise에서 이 기능을 사용하면 호스트와 컨테이너 버전 태그가 일치하는지도 확인해야 합니다. 일치하지 않으면 컨테이너가 시작되지 않거나 정의되지 않은 동작이 발생할 수 있습니다.

### <a name="isolation-explanation"></a>격리 설명

이 예제에서는 프로세스 격리와 Hyper-V 격리 간의 격리 기능 차이점을 보여줍니다.

여기서는 프로세스 격리 컨테이너를 배포하고 장기 실행 ping 프로세스를 호스팅합니다.

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

반대로 이 예제에서는 ping 프로세스로 Hyper-V 격리 컨테이너를 시작합니다.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
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
