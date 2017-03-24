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
translationtype: Human Translation
ms.sourcegitcommit: f615c6dd268932a2ff99ac12c4e9ffdcf2cc217e
ms.openlocfilehash: ee6053003b31f226d2cfba8566f274ccc19d97ec
ms.lasthandoff: 03/02/2017

---

# Swarm 모드 시작 

**중요:** *현재 Swarm 모드와 오버레이 네트워킹 지원은 예정된 Windows 10 크리에이터 업데이트의 일부로 [Windows 참가자](https://insider.windows.com/)에게만 제공됩니다. 추가 Windows 플랫폼에 대한 지원도 곧 제공될 예정입니다.*

## “Swarm 모드”란 무엇인가요?
Swarm 모드는 Docker 호스트의 네이티브 클러스터링과 컨테이너 워크로드 예약 등의 기본 제공 컨테이너 오케스트레이션 기능을 제공하는 Docker 기능입니다. 해당 Docker 엔진이 “Swarm 모드”에서 함께 실행될 때 Docker 호스트 그룹이 “Swarm” 클러스터를 구성합니다. Swarm 모드에 대한 추가 컨텍스트는 [Docker의 주요 설명서 사이트](https://docs.docker.com/engine/swarm/)를 참조하세요.

## 관리자 노드 및 작업자 노드
Swarm은 *관리자 노드*와 *작업자 노드*라는 두 가지 유형의 컨테이너 호스트로 구성됩니다. 모든 Swarm은 관리자 노드를 통해 초기화되며, Swarm 제어 및 모니터링을 위한 모든 Docker CLI 명령은 해당 관리자 노드 중 하나에서 실행되어야 합니다. 관리자 노드는 Swarm 상태의 “보관자”에 해당합니다. 여러 관리자 노드가 함께 Swarm에서 실행 중인 서비스의 상태 인식을 관리하는 합의 그룹을 구성하며, Swarm의 실제 상태가 항상 개발자나 관리자가 정의한 의도 상태와 일치하는지 확인하는 일을 합니다. 

>    **참고:** Swarm에 항상 *하나 이상*의 관리자 노드가 있어야 합니다. 

작업자 노드는 관리자 노드를 통해 Docker Swarm에 의해 오케스트레이션됩니다. 작업자 노드에 조인하려면 Swarm이 초기화될 때 작업자 노드가 관리자 노드에 의해 생성된 “조인 토큰”을 사용해야 합니다. 작업자 노드는 단순히 관리자 노드에서 작업을 받아서 실행하므로 Swarm 상태 인식을 필요로 하거나 소유하지 않습니다.

## Swarm 모드 시스템 요구 사항

**Windows 10 크리에이터 업데이트**([Windows 참가자](https://insider.windows.com/) 프로그램의 구성원용)를 실행 중인 물리적 또는 가상 컴퓨터 시스템 한 대 이상(Swarm의 모든 기능을 사용하려면 노드 두 개 이상 권장). Windows 10에서 Docker 컨테이너를 시작하는 방법에 대한 자세한 내용은 [Windows 10의 Windows 컨테이너](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10) 항목을 참조하세요.

**Docker 엔진 v1.13.0 이상**

열린 포트: 각 호스트에서 다음 포트를 사용할 수 있어야 합니다. 일부 시스템에서는 이러한 포트가 기본적으로 열려 있습니다.
- TCP 포트 2377: 클러스터 관리 통신용
- TCP 및 UDP 포트 7946: 노드 간 통신용
- TCP 및 UDP 포트 4789: 오버레이 네트워크 트래픽용

## Swarm 클러스터 초기화
Swarm을 초기화하려면 컨테이너 호스트 중 하나에서 다음 명령을 실행하기만 하면 됩니다(\<HOSTIPADDRESS\>를 호스트 컴퓨터의 로컬 IPv4 주소로 대체).

```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
지정된 컨테이너 호스트에서 이 명령을 실행하면 해당 호스트의 Docker 엔진이 Swarm 모드에서 관리자 노드로 실행되기 시작합니다.

## Swarm에 노드 추가

> **참고:** Swarm 모드와 오버레이 네트워킹 모드 기능을 사용하기 위해 여러 노드가 필요하지는 *않습니다*. Swarm 모드에서 실행 중인 단일 호스트로 모든 Swarm/오버레이 기능을 사용할 수 있습니다(예: `docker swarm init` 명령을 사용하여 Swarm 모드로 설정된 관리자 노드).

### Swarm에 작업자 추가
관리자 노드에서 Swarm이 초기화되면 또 다른 간단한 명령을 사용하여 다른 호스트를 Swarm에 작업자로 추가할 수 있습니다.

```none
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

여기에서 \<MANAGERIPADDRESS\>는 Swarm 관리자 노드의 로컬 IP 주소이고, \<WORKERJOINTOKEN\>은 관리자 노드에서 실행된 `docker swarm init` 명령이 출력으로 제공하는 작업자 조인 토큰입니다. Swarm이 초기화된 후 관리자 노드에서 다음 명령 중 하나를 실행하여 조인 토큰을 가져올 수도 있습니다.

```none
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### Swarm에 관리자 추가
다음 명령으로 Swarm 클러스터에 관리자 노드를 더 추가할 수 있습니다.

```none
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

마찬가지로 \<MANAGERIPADDRESS\>는 Swarm 관리자 노드의 로컬 IP 주소입니다. 조인 토큰인 \<MANAGERJOINTOKEN\>은 기존 관리자 노드에서 다음 명령 중 하나를 실행하여 가져올 수 있는 Swarm에 대한 *관리자* 조인 토큰입니다.

```none
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## 오버레이 네트워크 만들기

Swarm 클러스터가 구성되면 Swarm에 오버레이 네트워크를 만들 수 있습니다. Swarm 관리자 모드에서 다음 명령을 실행하여 오버레이 네트워크를 만들 수 있습니다.

```none
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

여기에서 \<NETWORKNAME\>은 네트워크에 지정하려는 이름입니다.

## Swarm에 서비스 배포
오버레이 네트워크가 만들어지면 서비스를 만들어 네트워크에 연결할 수 있습니다. 다음 구문으로 네트워크가 만들어집니다.

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

여기에서 \<SERVICENAME\>은 서비스에 지정하려는 이름입니다. 이 이름은 Docker의 네이티브 DNS 서버를 사용하는 서비스 검색을 통해 서비스를 참조하는 데 사용됩니다. \<NETWORKNAME\>은 이 서비스를 연결하려는 네트워크의 이름입니다(예: "myOverlayNet"). \<CONTAINERIMAGE\>는 서비스를 정의할 컨테이너 이미지의 이름입니다.

> **참고** 이 명령의 두 번째 인수인 `--endpoint-mode dnsrr`는 서비스 컨테이너 끝점 간에 네트워크 트래픽을 분산하는 데 DNS 라운드 로빈이 사용되도록 Docker 엔진에 지정하는 데 필요합니다. 현재 Windows에서 지원되는 부하 분산 전략은 DNS 라운드 로빈뿐입니다. Windows Docker 호스트에 대한 [라우팅 메시](https://docs.docker.com/engine/swarm/ingress/)는 아직 지원되지 않지만 곧 제공될 예정입니다. 오늘날의 대체 부하 분산 전략을 찾고 있는 사용자는 NGINX와 같은 외부 부하 분산 장치를 설치하고 Swarm의 [포트 게시 모드](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)를 사용하여 부하를 분산할 컨테이너 호스트 포트를 노출할 수 있습니다.

## 서비스 확장
서비스가 Swarm 클러스터에 배포되면 해당 서비스를 구성하는 컨테이너 인스턴스가 클러스터 전체에 배포됩니다. 기본적으로 서비스를 지원하는 컨테이너 인스턴스 수(서비스의 “복제본” 또는 “작업” 수)는 1개입니다. 그러나 `docker service create` 명령에 `--replicas` 옵션을 사용하거나 서비스를 만든 다음 확장하여 여러 작업을 포함하는 서비스를 만들 수 있습니다.

서비스 확장성은 Docker Swarm에서 제공하는 주요 이점이며 단일 Docker 명령으로 활용 가능합니다.

```none
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

여기에서 \<SERVICENAME\>은 확장 중인 서비스의 이름이며, \<REPLICAS\>는 서비스가 확장되고 있는 대상 작업 또는 컨테이너 인스턴스의 수입니다.

## Swarm 상태 보기

Swarm에서 실행 중인 서비스와 Swarm의 상태를 보는 몇 가지 유용한 명령이 있습니다.

### Swarm 노드 나열
다음 명령을 사용하여 Swarm에 현재 조인되어 있는 노드의 목록을 봅니다. 각 노드의 상태에 대한 정보도 확인할 수 있습니다. 이 명령은 **관리자 노드**에서 실행해야 합니다.

```none
C:\ docker node ls
```

이 명령의 출력에서 노드 중 하나에 별표(*)가 표시되어 있음을 확인할 수 있습니다. 별표는 단순히 현재 노드 즉, `docker node ls` 명령이 실행된 노드를 나타냅니다.

### 네트워크 나열
다음 명령을 사용하여 지정된 노드에 있는 네트워크의 목록을 봅니다. 오버레이 네트워크를 보려면 Swarm 모드로 실행 중인 **관리자 모드**에서 이 명령을 실행해야 합니다.

```none
C:\ docker network ls
```

### 서비스 나열
다음 명령을 사용하여 Swarm에서 현재 실행되어 있는 서비스의 목록을 봅니다. 해당 상태에 대한 정보도 확인할 수 있습니다.

```none
C:\ docker service ls
```

### 서비스를 정의하는 컨테이너 인스턴스 나열
다음 명령을 사용하여 지정된 서비스에 대해 실행 중인 컨테이너 인스턴스의 세부 정보를 봅니다. 이 명령의 출력에는 각 컨테이너가 실행 중인 ID 및 노드와 컨테이너의 상태에 대한 정보가 포함됩니다.  

```none
C:\ docker service ps <SERVICENAME>
```

## 제한 사항
현재 Windows의 Swarm 모드에는 다음과 같은 제한 사항이 있습니다.
- 알려진 버그: 현재는 이더넷에 연결된 호스트에서만 오버레이 및 Swarm 모드가 지원됩니다. **WiFi에 연결된 호스트에서는 오버레이 및 Swarm 모드가 작동하지 않습니다.** 이 버그 수정을 위해 최선을 다하고 있습니다.
- 데이터 평면 암호화가 지원되지 않습니다(예: `--opt encrypted` 옵션을 사용하는 컨테이너-컨테이너 트래픽).
- Windows Docker 호스트에 대한 [라우팅 메시](https://docs.docker.com/engine/swarm/ingress/)는 아직 지원되지 않지만 곧 제공될 예정입니다. 오늘날의 대체 부하 분산 전략을 찾고 있는 사용자는 NGINX와 같은 외부 부하 분산 장치를 설치하고 Swarm의 [포트 게시 모드](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)를 사용하여 부하를 분산할 컨테이너 호스트 포트를 노출할 수 있습니다.  



