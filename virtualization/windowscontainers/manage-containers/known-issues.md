---
title: 알려진 문제
description: Windows Server 컨테이너의 알려진 문제
keywords: 메타데이터, 컨테이너, 버전,
author: weijuans
ms. author: weijuans
manager: taylob
ms.date: 05/26/2020
ms.openlocfilehash: f53ff0c8c07e86b25358a3acba09622fdd3a6bbb
ms.sourcegitcommit: 82f45088d8b39e2c1f3f2a33bf359d18a88f975a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/18/2020
ms.locfileid: "84978335"
---
# <a name="known-issues"></a>알려진 문제

## <a name="know-issues-of-windows-server-version-2004"></a>Windows Server, 버전 2004의 알려진 문제

### <a name="1-performance-issue-on-server-core-container"></a>1. Server Core 컨테이너의 성능 문제
Windows Server, 버전 2004 릴리스의 일반 공급을 준비하면서 2020년 5월 27일 릴리스의 현재 Server Core 컨테이너 이미지에 대해 2019년 12월 블로그에 게시한 성능 개선 사항([여기](https://techcommunity.microsoft.com/t5/containers/making-windows-server-core-containers-40-smaller/ba-p/1058874))과 .NET 팀 블로그([여기](https://devblogs.microsoft.com/dotnet/we-made-windows-server-core-container-images-40-smaller/))에 있는 성능 개선 사항을 비교하면서 .NET 팀과 함께 잠재적인 성능 문제를 확인했습니다. 당시 성능 분석은 Windows Server, 버전 2004 Insider Preview Release Server Core 컨테이너 이미지에서 수행되었습니다. 

관찰한 증상은 다음과 같습니다.

Server Core 컨테이너 이미지를 사용하여 자체 이미지를 빌드하고 Azure Container Registry와 같은 원격 컨테이너 레지스트리에 업로드한 다음, 레지스트리에서 해당 이미지를 가져와서 실행하면 컨테이너의 성능이 저하됩니다. 그러나 이미지를 빌드하고 로컬로 이미지를 실행하는 경우 성능 차이를 알 수 없습니다.

이 문제는 현재 근본 원인이며 수정 작업을 진행하고 있습니다. 이 문제를 추적하는 다음 링크를 찾을 수 있습니다. [microsoft/hcsshim#830](https://github.com/microsoft/hcsshim/issues/830);

[moby/moby#41066](https://github.com/moby/moby/issues/41066);

[containerd/containerd#4301](https://github.com/containerd/containerd/issues/4301).

또한 파트너인 Mirantis와 함께 Docker EE에서 적극적으로 수정 방법을 구하고 있습니다.

## <a name="know-issues-of-windows-server-version-1909"></a>Windows Server, 버전 1909의 알려진 문제

## <a name="know-issues-of-windows-server-version-1903"></a>Windows Server, 버전 1903의 알려진 문제

## <a name="know-issues-of-windows-server-2019windows-server-version-1809"></a>Windows Server 2019/Windows Server, 버전 1809의 알려진 문제

## <a name="know-issues-of-windows-server-2016"></a>Windows Server 2016의 알려진 문제
