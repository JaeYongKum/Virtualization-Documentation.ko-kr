---
title: "Swarm 모드 시작"
description: "Swarm 클러스터 초기화, 오버레이 네트워크 만들기 및 네트워크에 서비스 연결"
keywords: "Docker, 컨테이너, Swarm, 오케스트레이션"
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
ms.openlocfilehash: 8dc87c28d402ed3bf7e704e7ffc7128e5a5aa15e
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/08/2017
---
# <a name="getting-started-with-swarm-mode"></a>Swarm 모드 시작 

## <a name="what-is-swarm-mode"></a>“Swarm 모드”란 무엇인가요?
Swarm 모드는 Docker 호스트의 네이티브 클러스터링과 컨테이너 워크로드 예약 등의 기본 제공 컨테이너 오케스트레이션 기능을 제공하는 Docker 기능입니다. 해당 Docker 엔진이 “Swarm 모드”에서 함께 실행될 때 Docker 호스트 그룹이 “Swarm” 클러스터를 구성합니다. Swarm 모드에 대한 추가 컨텍스트는 [Docker의 주요 설명서 사이트](https://docs.docker.com/engine/swarm/)를 참조하세요.

## <a name="manager-nodes-and-worker-nodes"></a>관리자 노드 및 작업자 노드
Swarm은 *관리자 노드*와 *작업자 노드*라는 두 가지 유형의 컨테이너 호스트로 구성됩니다. 모든 Swarm은 관리자 노드를 통해 초기화되며, Swarm 제어 및 모니터링을 위한 모든 Docker CLI 명령은 해당 관리자 노드 중 하나에서 실행되어야 합니다. 관리자 노드는 Swarm 상태의 “보관자”에 해당합니다. 여러 관리자 노드가 함께 Swarm에서 실행 중인 서비스의 상태 인식을 관리하는 합의 그룹을 구성하며, Swarm의 실제 상태가 항상 개발자나 관리자가 정의한 의도 상태와 일치하는지 확인하는 일을 합니다. 

>   **참고:** Swarm에 항상 *하나 이상*의 관리자 노드가 있어야 합니다. 

작업자 노드는 관리자 노드를 통해 Docker Swarm에 의해 오케스트레이션됩니다. 작업자 노드에 조인하려면 Swarm이 초기화될 때 작업자 노드가 관리자 노드에 의해 생성된 “조인 토큰”을 사용해야 합니다. 작업자 노드는 단순히 관리자 노드에서 작업을 받아서 실행하므로 Swarm 상태 인식을 필요로 하거나 소유하지 않습니다.

## <a name="swarm-mode-system-requirements"></a>Swarm 모드 시스템 요구 사항

*최신 업데이트가 모두 설치된\** **Windows 10 크리에이터스 업데이트** 또는 **Windows Server 2016**을 실행 중이고 컨테이너 호스트로 설정된 하나 이상의(적어도 두 개 이상의 노드에서 Swarm의 전체 기능을 사용할 수 있도록) 실제 또는 가상 컴퓨터 시스템([Windows 10에서 Docker 컨테이너를 시작하는 방법에 대한 자세한 내용은 Windows 10의 Windows 컨테이너](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10) 또는 [Windows Server의 Windows 컨테이너](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-server) 참조).

\***참고**: Windows Server 2016에서 Docker Swarm을 사용하려면 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217)이 필요

**Docker 엔진 v1.13.0 이상**

열린 포트: 각 호스트에서 다음 포트를 사용할 수 있어야 합니다. 일부 시스템에서는 이러한 포트가 기본적으로 열려 있습니다.
- TCP 포트 2377: 클러스터 관리 통신용
- TCP 및 UDP 포트 7946: 노드 간 통신용
- 오버레이 네트워크 트래픽용 UDP 포트 4789

## <a name="initializing-a-swarm-cluster"></a>Swarm 클러스터 초기화

Swarm을 초기화하려면 컨테이너 호스트 중 하나에서 다음 명령을 실행하기만 하면 됩니다(\<HOSTIPADDRESS\>를 호스트 컴퓨터의 로컬 IPv4 주소로 대체).

```
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
지정된 컨테이너 호스트에서 이 명령을 실행하면 해당 호스트의 Docker 엔진이 Swarm 모드에서 관리자 노드로 실행되기 시작합니다.

## <a name="adding-nodes-to-a-swarm"></a>Swarm에 노드 추가

> **참고:** Swarm 모드와 오버레이 네트워킹 모드 기능을 사용하기 위해 여러 노드가 필요하지는 *않습니다*. Swarm 모드에서 실행 중인 단일 호스트로 모든 Swarm/오버레이 기능을 사용할 수 있습니다(예: `docker swarm init` 명령을 사용하여 Swarm 모드로 설정된 관리자 노드).

### <a name="adding-workers-to-a-swarm"></a>Swarm에 작업자 추가
관리자 노드에서 Swarm이 초기화되면 또 다른 간단한 명령을 사용하여 다른 호스트를 Swarm에 작업자로 추가할 수 있습니다.

```
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

여기서 \<MANAGERIPADDRESS\>는 Swarm 관리자 노드의 로컬 IP 주소이고, \<WORKERJOINTOKEN\>은 관리자 노드에서 실행된 `docker swarm init` 명령이 출력으로 제공하는 작업자 조인 토큰입니다. Swarm이 초기화된 후 관리자 노드에서 다음 명령 중 하나를 실행하여 조인 토큰을 가져올 수도 있습니다.

```
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### <a name="adding-managers-to-a-swarm"></a>Swarm에 관리자 추가
다음 명령으로 Swarm 클러스터에 관리자 노드를 더 추가할 수 있습니다.

```
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

마찬가지로 \<MANAGERIPADDRESS\>는 Swarm 관리자 노드의 로컬 IP 주소입니다. 조인 토큰인 \<MANAGERJOINTOKEN\>은 기존 관리자 노드에서 다음 명령 중 하나를 실행하여 가져올 수 있는 Swarm에 대한 *관리자* 조인 토큰입니다.

```
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## <a name="creating-an-overlay-network"></a>오버레이 네트워크 만들기

Swarm 클러스터가 구성되면 Swarm에 오버레이 네트워크를 만들 수 있습니다. Swarm 관리자 모드에서 다음 명령을 실행하여 오버레이 네트워크를 만들 수 있습니다.

```
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

여기서 \<NETWORKNAME\>은 네트워크에 지정하려는 이름입니다.

## <a name="deploying-services-to-a-swarm"></a>Swarm에 서비스 배포
오버레이 네트워크가 만들어지면 서비스를 만들어 네트워크에 연결할 수 있습니다. 서비스는 다음 구문을 사용하여 만듭니다.

```
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

여기서 \<SERVICENAME\>은 서비스에 지정하려는 이름입니다. 이 이름은 Docker의 네이티브 DNS 서버를 사용하는 서비스 검색을 통해 서비스를 참조하는 데 사용됩니다. \<NETWORKNAME\>은 이 서비스를 연결하려는 네트워크의 이름입니다(예: "myOverlayNet"). \<CONTAINERIMAGE\>는 서비스를 정의할 컨테이너 이미지의 이름입니다.

> **참고** 이 명령의 두 번째 인수인 `--endpoint-mode dnsrr`는 서비스 컨테이너 끝점 간에 네트워크 트래픽을 분산하는 데 DNS 라운드 로빈이 사용되도록 Docker 엔진에 지정하는 데 필요합니다. 현재 Windows에서 지원되는 부하 분산 전략은 DNS 라운드 로빈뿐입니다. Windows Docker 호스트에 대한 [라우팅 메시](https://docs.docker.com/engine/swarm/ingress/)는 아직 지원되지 않지만 곧 제공될 예정입니다. 오늘날의 대체 부하 분산 전략을 찾고 있는 사용자는 NGINX와 같은 외부 부하 분산 장치를 설치하고 Swarm의 [포트 게시 모드](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)를 사용하여 부하를 분산할 컨테이너 호스트 포트를 노출할 수 있습니다.

## <a name="scaling-a-service"></a>서비스 확장
서비스가 Swarm 클러스터에 배포되면 해당 서비스를 구성하는 컨테이너 인스턴스가 클러스터 전체에 배포됩니다. 기본적으로 서비스를 지원하는 컨테이너 인스턴스 수(서비스의 “복제본” 또는 “작업” 수)는 1개입니다. 그러나 `docker service create` 명령에 `--replicas` 옵션을 사용하거나 서비스를 만든 다음 확장하여 여러 작업을 포함하는 서비스를 만들 수 있습니다.

서비스 확장성은 Docker Swarm에서 제공하는 주요 이점이며 단일 Docker 명령으로 활용 가능합니다.

```
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

여기서 \<SERVICENAME\>은 확장 중인 서비스의 이름이며, \<REPLICAS\>는 서비스가 확장되는 대상 작업 또는 컨테이너 인스턴스의 수입니다.


## <a name="viewing-the-swarm-state"></a>Swarm 상태 보기

Swarm에서 실행 중인 서비스와 Swarm의 상태를 보는 몇 가지 유용한 명령이 있습니다.

### <a name="list-swarm-nodes"></a>Swarm 노드 나열
다음 명령을 사용하여 Swarm에 현재 조인되어 있는 노드의 목록을 봅니다. 각 노드의 상태에 대한 정보도 확인할 수 있습니다. 이 명령은 **관리자 노드**에서 실행해야 합니다.

```
C:\> docker node ls
```

이 명령의 출력에서 노드 중 하나에 별표(*)가 표시되어 있음을 확인할 수 있습니다. 별표는 단순히 현재 노드 즉, `docker node ls` 명령이 실행된 노드를 나타냅니다.

### <a name="list-networks"></a>네트워크 나열
다음 명령을 사용하여 지정된 노드에 있는 네트워크의 목록을 봅니다. 오버레이 네트워크를 보려면 Swarm 모드로 실행 중인 **관리자 모드**에서 이 명령을 실행해야 합니다.

```
C:\> docker network ls
```

### <a name="list-services"></a>서비스 나열
다음 명령을 사용하여 Swarm에서 현재 실행되어 있는 서비스의 목록을 봅니다. 해당 상태에 대한 정보도 확인할 수 있습니다.

```
C:\> docker service ls
```

### <a name="list-the-container-instances-that-define-a-service"></a>서비스를 정의하는 컨테이너 인스턴스 나열
다음 명령을 사용하여 지정된 서비스에 대해 실행 중인 컨테이너 인스턴스의 세부 정보를 봅니다. 이 명령의 출력에는 각 컨테이너가 실행 중인 ID 및 노드와 컨테이너의 상태에 대한 정보가 포함됩니다.  

```
C:\> docker service ps <SERVICENAME>
```
## <a name="linuxwindows-mixed-os-clusters"></a>Linux+Windows 혼합 OS 클러스터

최근에 저희 팀원 중 한 명이 Docker Swarm을 사용하여 Linux+Windows 혼합 OS 응용 프로그램을 설치하는 방법에 대한 짧은 3부작 데모를 게시했습니다. Docker Swarm을 처음 접하거나 혼합 OS 응용 프로그램을 실행하는 데 사용하려는 분들이 처음 시작할 때 많은 도움이 될 것입니다. 지금 확인하세요.
- [Docker Swarm을 사용하여 Windows+Linux 컨테이너화 응용 프로그램 실행(1/3)](https://www.youtube.com/watch?v=ZfMV5JmkWCY&t=170s)
- [Docker Swarm을 사용하여 Windows+Linux 컨테이너화 응용 프로그램 실행(2/3)](https://www.youtube.com/watch?v=VbzwKbcC_Mg&t=406s)
- [Docker Swarm을 사용하여 Windows+Linux 컨테이너화 응용 프로그램 실행(3/3)](https://www.youtube.com/watch?v=I9oDD78E_1E&t=354s)

### <a name="initializing-a-linuxwindows-mixed-os-cluster"></a>Linux + Windows 혼합 OS 클러스터 초기화
방화벽 규칙이 적절하게 구성되어 있고 호스트 간 액세스가 가능하다면 간단하게 혼합 OS Swarm 클러스터를 초기화할 수 있으며, `docker swarm join` 명령만 있으면 Swarm에 Linux 호스트를 추가할 수 있습니다.
```
C:\> docker swarm join --token <JOINTOKEN> <MANAGERIPADDRESS>
```
또한 Windows 호스트에서 Swarm을 초기화할 때 실행하는 명령과 동일한 명령을 사용하여 Linux 호스트에서 Swarm을 초기화할 수 있습니다.
```
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```

### <a name="adding-labels-to-swarm-nodes"></a>Swarm 노드에 레이블 추가
Docker Service를 혼합 OS Swarm 클러스터로 실행하려면 해당 서비스가 설계된 대상 OS를 실행하는 Swarm 노드와 그렇지 않은 Swarm 노드를 구별하는 방법이 있어야 합니다. [Docker 개체 레이블](https://docs.docker.com/engine/userguide/labels-custom-metadata/)은 대상 OS와 일치하는 노드에서만 실행되는 서비스를 만들고 구성할 수 있도록 노드에 레이블을 지정하는 유용한 방법을 제공합니다. 

> 참고: [Docker 개체 레이블](https://docs.docker.com/engine/userguide/labels-custom-metadata/)은 컨테이너 이미지, 컨테이너, 볼륨, 네트워크 등 다양한 Docker 개체에 메타데이터를 적용하는 데 사용할 수 있으며 그 외에도 다양한 목적으로 사용할 수 있습니다(예: 레이블을 사용하면 프런트 엔드 마이크로 서비스는 '프런트 엔드' 레이블이 지정된 노드에서만 예약 가능하고 백 엔드 마이크로 서비스는 '백 엔드' 레이블이 지정된 노드에서만 예약 가능하게 하여 응용 프로그램의 '프런트 엔드' 구성 요소와 '백 엔드' 구성 요소를 분리할 수 있음). 여기서는 노드에 레이블을 사용하여 Windows OS 노드와 Linux OS 노드를 구별하겠습니다.

기존 Swarm 노드에 레이블을 지정하려면 다음 구문을 사용합니다.

```
C:\> docker node update --label-add <LABELNAME>=<LABELVALUE> <NODENAME>
```

여기서 `<LABELNAME>`은 만들려는 레이블의 이름입니다. 예를 들어 이 예에서는 OS를 통해 노드를 구분할 것이므로 레이블의 논리 이름으로 "os"를 사용할 수 있습니다. `<LABELVALUE>` 레이블의 값이며 이 예에서는 "windows" 및 "linux" 값을 선택할 수 있습니다. 물론 일관성을 유지하는 범위 내에서 레이블 및 레이블 값에 대한 명명 규칙을 원하는 대로 선택할 수 있습니다. `<NODENAME>` 레이블을 지정할 노드의 이름이며 `docker node ls` 명령을 실행하여 노드 이름을 미리 알릴 수 있습니다. 

**예를 들어** 클러스터에 Windows 노드 두 개, Linux 노드 두 개, 총 네 개의 Swarm 노드가 있는 경우 레이블 업데이트 명령은 다음과 같습니다.

```
# Example -- labeling 2 Windows nodes and 2 Linux nodes in a cluster...
C:\> docker node update --label-add os=windows Windows-SwarmMaster
C:\> docker node update --label-add os=windows Windows-SwarmWorker1
C:\> docker node update --label-add os=linux Linux-SwarmNode1
C:\> docker node update --label-add os=linux Linux-SwarmNode2
```

### <a name="deploying-services-to-a-mixed-os-swarm"></a>혼합 OS Swarm에 서비스 배포
Swarm 노드에 레이블을 사용하면 클러스터에 서비스를 쉽게 배포할 수 있습니다. 간단하게 `--constraint` 옵션을 [`docker service create`](https://docs.docker.com/engine/reference/commandline/service_create/) 명령에 사용하면 됩니다.

```
# Deploy a service with swarm node constraint
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> --constraint node.labels.<LABELNAME>=<LABELVALUE> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

예를 들어 위의 예에서 보여드린 레이블 및 레이블 값 명명법과 서비스 만들기 명령 집합(하나는 Windows 기반 서비스용, 또 하나는 Linux 기반 서비스용)을 사용하면 다음과 같이 표시됩니다.

```
# Example -- using the 'os' label and 'windows'/'linux' label values, service creation commands might look like these...

# A Windows service
C:\> docker service create --name=win_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==windows' microsoft/nanoserver:latest powershell -command { sleep 3600 }

# A Linux service
C:\> docker service create --name=linux_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==linux' redis
```

## <a name="limitations"></a>제한 사항
현재 Windows의 Swarm 모드에는 다음과 같은 제한 사항이 있습니다.
- 데이터 평면 암호화가 지원되지 않습니다(예: `--opt encrypted` 옵션을 사용하는 컨테이너 간 트래픽).
- Windows Docker 호스트에 대한 [라우팅 메시](https://docs.docker.com/engine/swarm/ingress/)는 아직 지원되지 않지만 곧 제공될 예정입니다. 오늘날의 대체 부하 분산 전략을 찾고 있는 사용자는 NGINX와 같은 외부 부하 분산 장치를 설치하고 Swarm의 [포트 게시 모드](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)를 사용하여 부하를 분산할 컨테이너 호스트 포트를 노출할 수 있습니다. 여기에 대한 자세한 내용은 아래를 참조하세요.

## <a name="publish-ports-for-service-endpoints"></a>서비스 끝점에 대한 포트 게시
Docker Swarm의 [라우팅 메시](https://docs.docker.com/engine/swarm/ingress/) 기능은 아직 Windows에서 지원되지 않지만 서비스 끝점에 대한 포트를 게시하려는 사용자는 포트 게시 모드를 사용하여 게시할 수 있습니다. 

서비스를 정의하는 각 작업/컨테이너 끝점에 대한 호스트 포트를 게시하려면 `--publish mode=host,target=<CONTAINERPORT>` 인수를 `docker service create` 명령에 사용합니다.

```
# Create a service for which tasks are exposed via host port
C:\ > docker service create --name=<SERVICENAME> --publish mode=host,target=<CONTAINERPORT> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

예를 들어 다음 명령은 서비스 's1'을 만들며, 각 작업은 컨테이너 포트 80 및 임의로 선택된 호스트 포트를 통해 이 서비스에 노출됩니다.

```
C:\ > docker service create --name=s1 --publish mode=host,target=80 --endpoint-mode dnsrr web_1 powershell -command {echo sleep; sleep 360000;}
```

포트 게시 모드를 사용하여 서비스를 만든 후에는 서비스를 쿼리하여 각 서비스 작업에 대한 포트 매핑을 볼 수 있습니다.

```
C:\ > docker service ps <SERVICENAME>
```
위의 명령은 모든 Swarm 호스트에서 서비스에 대해 실행되는 모든 컨테이너 인스턴스에 대한 세부 정보를 반환합니다. 출력의 열 중 하나인 “포트” 열에는 각 호스트의 포트 정보가 \<HOSTPORT\>->\<CONTAINERPORT\>/tcp 형태로 포함됩니다. 각 컨테이너가 자체 호스트 포트에 게시되므로 \<HOSTPORT\> 값은 컨테이너 인스턴스마다 다릅니다.


## <a name="tips--insights"></a>팁과 고급 정보 

#### *<a name="existing-transparent-network-can-block-swarm-initializationoverlay-network-creation"></a>기존 투명 네트워크는 Swarm 초기화/오버레이 네트워크 생성을 차단할 수 있습니다.* 
Windows에서 오버레이 네트워크 드라이버와 투명 네트워크 드라이버를 (가상) 호스트 네트워크 어댑터에 바인딩하려면 모두 외부 vSwitch가 필요합니다. 오버레이 네트워크가 만들어지면 새로운 스위치가 만들어지고 열린 네트워크 어댑터에 연결됩니다. 투명 네트워킹 모드도 호스트 네트워크 어댑터를 사용합니다. 이와 동시에, 모든 지정된 네트워크 어댑터는 한 번에 한 스위치에만 바인딩할 수 있습니다. 호스트에 네트워크 어댑터가 하나만 있는 경우 vSwitch가 오버레이 네트워크용인지 아니면 투명 네트워크용인지에 관계없이 한 번에 한 외부 vSwitch에만 연결할 수 있습니다. 

그러므로 컨테이너 호스트에 네트워크 어댑터가 하나밖에 없으면 투명 네트워크가 오버레이 네트워크 생성을 차단하는(또는 그 반대) 문제가 발생할 수 있습니다. 현재 투명 네트워크가 호스트의 유일한 가상 네트워크 인터페이스를 점유하고 있기 때문입니다.

이 문제를 해결하는 두 가지 방법이 있습니다.
- *옵션 1 - 기존 투명 네트워크 삭제:* Swarm을 초기화하기 전에 컨테이너 호스트에 기존 투명 네트워크가 없는지 확인합니다. 투명 네트워크를 삭제하여 호스트에서 오버레이 네트워크 생성에 사용할 가상 네트워크 어댑터를 확보합니다.
- *옵션 2 - 호스트에 추가 (가상) 네트워크 어댑터 만들기:* 호스트에 있는 투명 네트워크를 제거하는 대신 호스트에 추가 네트워크 어댑터를 만들어서 오버레이 네트워크 생성에 사용할 수 있습니다. 이렇게 하려면 간단하게 PowerShell Hyper-V 관리자를 사용하여 새 외부 네트워크 어댑터를 만들면 됩니다. 새 인터페이스가 준비된 상태에서 Swarm이 초기화되면 HNS(호스트 네트워크 서비스)가 자동으로 호스트에서 새 외부 네트워크 어댑터를 인식하고 외부 vSwitch를 바인딩에 사용하여 오버레이 네트워크를 만듭니다.



