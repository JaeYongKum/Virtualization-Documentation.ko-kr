---
title: "Windows 컨테이너 FAQ"
description: "Windows 컨테이너 FAQ"
keywords: "Docker, 컨테이너"
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: b084bf179d9360e4a72e8e88b4fec80eafb2906c
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/21/2017
---
# 질문과 대답

## Windows 컨테이너 정보

**Windows Server 컨테이너란?**

Windows Server 컨테이너는 응용 프로그램 또는 서비스를, 동일한 컨테이너 호스트에서 실행되는 다른 서비스와 구분하는 데 사용되는 가벼운 운영 체제 가상화 방법입니다. 이 방법을 사용하려면 각 컨테이너에 운영 체제, 프로세스, 파일 시스템, 레지스트리 및 IP 주소에 대한 자체 뷰가 있어야 합니다.  

**Hyper-V 컨테이너란?**

Hyper-V 컨테이너를 Hyper-V 파티션 내부에서 실행 중인 Windows Server 컨테이너로 간주할 수 있습니다.

Hyper-V 컨테이너는 매우 효율적인 고밀도 Windows Server 컨테이너와, 격리 수준이 높은 하드웨어 가상화 Hyper-V 가상 컴퓨터 간에 추가적인 배포 옵션을 제공합니다. 서로 다른 신뢰 경계의 응용 프로그램이 동일한 호스트에서 실행되는 환경에서는 추가적인 격리가 필요할 수 있습니다. Hyper-V 컨테이너는 최적화된 가상화 및 Windows Server 운영 체제를 통해, 컨테이너와 호스트 운영 체제가 분리되고 컨테이너 간에도 분리되는 더 높은 수준의 격리를 제공합니다. 두 컨테이너 배포 옵션은 동일한 관리 API, ehrn 및 이미지 형식을 사용합니다. 배포 시 고객은 요구 사항에 가장 부합하는 배포 모드를 선택하기만 하면 됩니다.

**Linux와 Windows Server 컨테이너 간의 차이는 무엇입니까?**

Linux와 Windows Server 컨테이너는 유사하며 커널과 코어 운영 체제 안에서는 비슷한 기술을 구현합니다. 차이는 컨테이너 안에서 실행되는 워크로드와 플랫폼에 있습니다.  
고객이 Windows Server 컨테이너를 사용할 경우 .NET, ASP.NET, PowerShell 등의 기존 Windows 기술과 통합할 수 있습니다.

**개발자가 각 컨테이너 유형마다 앱을 다시 작성해야 하나요?**

아니요. Windows 컨테이너 이미지는 Windows Server 컨테이너와 Hyper-V 컨테이너 간에 공통입니다. 컨테이너를 시작할 때 컨테이너 유형을 선택합니다. 개발자의 관점에서 Windows Server 컨테이너와 Hyper-V 컨테이너는 같은 물건의 두 버전이라고 할 수 있습니다. 개발, 프로그래밍, 관리 환경이 동일하고, 개방형 및 확장 가능하며, Docker를 통해 동일한 수준의 통합과 지원을 포함합니다. 

개발자는 적합한 런타임 플래그를 지정하는 것 말고는 별다른 변경 없이도 Windows Server 컨테이너를 사용하여 컨테이너 이미지를 만들고 Hyper-V 컨테이너로 배포하거나 혹은 반대의 경우로 할 수 있습니다.

Windows Server 컨테이너는 속도가 중요한 상황에서 더 높은 밀도와 성능을 제공합니다(중첩 구성 대비 스핀 시간 감소, 런타임 성능 빠름). Hyper-V 컨테이너는 더 높은 격리 수준을 제공하므로 한 컨테이너에서 실행되는 코드가 호스트 운영 체제나, 동일한 호스트에서 실행되는 타 컨테이너에 손상을 입히거나 영향을 끼지지 않습니다. 이 유형은 SaaS 운영 체제와 계산 호스팅 등, 다중 테넌트 시나리오에서 유용합니다(신뢰할 수 없는 코드 호스팅에 대한 요구 사항 있음).

**Hyper-V/Windows Server 컨테이너가 추가 기능인가요 아니면 Windows Server 안에 통합되나요?**

컨테이너 기능은 Windows Server 2016에 통합됩니다.  

**Windows Server 컨테이너와 Drawbridge는 서로 어떤 관계인가요?**

Drawbridge는 컨테이너에 대한 중요한 정보를 얻는 데 도움을 준 여러 연구 프로젝트 중 하나였습니다.  Windows Server 2016의 컨테이너 기술 중 대다수는 Drawbridge와의 경험에서 탄생했으며 Windows Server 2016에 세계적 수준의 컨테이너 기술을 담아 고객에게 제공할 수 있게 되어 기쁘게 생각합니다.

**Windows Server 컨테이너와 Hyper-V 컨테이너의 필수 구성 요소는 무엇인가요?**

Windows Server 컨테이너와 Hyper-V 컨테이너 모두 Windows Server 2016이 필요합니다. 이 기술은 이전 Windows 버전에서 작동하지 않습니다.


## Windows 컨테이너 관리

**Hyper-V 컨테이너 Docker 생태계에서 사용 가능한가요?**

예. Hyper-V 컨테이너도 Docker를 통해 Windows Server와 동일한 수준의 통합과 관리를 제공할 것입니다.  목표는 개방 및 일관된 플랫폼 간 환경을 구현하는 것입니다.   
Docker 플랫폼도 컨테이너 옵션 전체에서의 작업 환경을 크게 간소화 및 개선합니다. Windows Server 컨테이너를 사용하여 개발된 응용 프로그램은 변경 없이 Hyper-V 컨테이너 형태로 배포할 수 있습니다.


## Microsoft의 오픈 에코 시스템

**Microsoft OCI( Open Container Initiative)에 해당하나요?**

패키징 형식을 범용으로 유지하기 위해 최근 Docker는 기구에서 주도하는 형식에 따른 개방성 유지를 목표로 하는 OCI( Open Container Initiative)를 구성했습니다. Microsoft는 창립 구성원 중 하나입니다.

**Microsoft가 실제 Docker와 협력하나요?**

예.  
개발자들이 동일한 Docker 도구를 사용하여 Windows Server 및 Linux 컨테이너를 모두 만들고 관리하며 배포할 수 있도록 Docker와 협력하고 있습니다. Windows Server를 목표로 하는 개발자들은 더 이상 광범위한 Windows Server 기술을 사용하는 것과 컨테이너화된 응용 프로그램의 구축 사이에서 선택을 고민할 필요가 없습니다.  

Docker는 프로젝트의 오픈 소스 그룹인 동시에 회사를 나타냅니다. 이 파트너 관계는 둘 모두에 해당하는 것으로 간주합니다. Docker 컨테이너 기술을 바탕으로 구축된 생동감 넘치는 생태계를 보유한 Docker는 부분적으로 성공을 거두었습니다.  Microsoft은 Windows Server 컨테이너와 Hyper-V 컨테이너에 대한 지원을 구현하는 Docker 프로젝트에 기여하고 있습니다.  

자세한 내용은 [New Windows Server containers and Azure support for Docker(새 Windows Server 컨테이너 및 Docker에 대한 Azure 지원)](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD) 블로그 게시물을 참조하세요.
