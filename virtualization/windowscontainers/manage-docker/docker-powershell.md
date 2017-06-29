---
title: "Docker용 PowerShell"
description: "PowerShell을 사용하여 Docker 컨테이너를 관리하는 방법"
keywords: "docker, 컨테이너, powershell"
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 4a0e907d-0d07-42f8-8203-2593391071da
ms.openlocfilehash: bcbc2e4e76c48a3d9a1a9720b09ef366a396bf30
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: ko-KR
---
### <a name="powershell-for-docker"></a>Docker용 PowerShell

포럼, Twitter, GitHub를 통한 사용자와의 대화, 사용자와의 직접적인 대화를 통해 파악하게 된 가장 많은 질문 하나는 PowerShell에서 Docker 컨테이너를 볼 수 없는 이유는 무엇인가요?입니다. 

지금까지 찬성, 반대 및 다양한 옵션을 살펴본 대로 컨테이너 PowerShell 모듈에 업데이트가 필요하다는 결론에 도달했습니다. 따라서 Windows Server 2016의 Preview 빌드에서 제공되던 컨테이너 PowerShell 모듈을 더 이상 사용하지 않고 새로운 Docker용 PowerShell 모듈로 교체하는 작업을 시작했습니다.  이 새 모듈의 개발은 이미 진행 중이지만 과거와 다른 접근 방식을 사용하여 공개적으로 작업을 수행하고 있습니다.  이 모듈의 목표는 Docker 엔진을 통해 우수한 컨테이너용 PowerShell 환경을 만드는 커뮤니티 공동 작업입니다.  이 새 모듈은 Docker 엔진의 REST 인터페이스 위에 직접 구축되어 사용자가 Docker CLI, PowerShell 또는 둘 다를 선택할 수 있습니다.

뛰어난 PowerShell 모듈 작성은 간단한 작업이 아닙니다. 모든 코드를 적절히 배치하는 것과 개체와 매개 변수 집합의 올바른 균형을 잡는 것, cmdlet 이름 모두 매우 중요합니다.  따라서 이 새 모듈을 시작할 때 최종 사용자와 이 모듈을 형성하는 데 도움이 되는 광범위한 PowerShell 및 Docker 커뮤니티를 고려합니다.  중요한 매개 변수 집합은 무엇인가요?  "docker run"에 해당하는 명령이 있는 경우 또는 new-container를 start-container에 파이프해야 하는 경우 원하는 작업은 무엇인가요?  이 모듈에 대해 자세히 알아보고 개발에 참여하려면 GitHub 페이지(https://github.com/Microsoft/Docker-PowerShell/)를 검토하고 참가하세요.

개발이 진행되고 견고한 알파 품질 모듈에 도달하면 PowerShell 갤러리에 게시하고 사용 지침으로 이 페이지를 업데이트할 예정입니다.
