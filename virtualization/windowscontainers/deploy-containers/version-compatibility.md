---
title: Windows 컨테이너 버전 호환성
description: Windows에서 빌드를 실행하고 다양한 버전 간에 컨테이너를 실행할 수 있는 방법
keywords: 메타데이터, 컨테이너, 버전
author: taylorb-microsoft
ms.openlocfilehash: 5fe1cca67c330cb59362e82762651d719708b526
ms.sourcegitcommit: 27e9cd37beaf11e444767699886e5fdea5e1a2d0
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/30/2019
ms.locfileid: "10058508"
---
# <a name="windows-container-version-compatibility"></a>Windows 컨테이너 버전 호환성

Windows server 2016 및 Windows 10 기념일 업데이트 (버전 14393)는 Windows Server 컨테이너를 빌드하고 실행할 수 있는 첫 번째 Windows 릴리스 였습니다. 이러한 버전을 사용하여 빌드된 컨테이너는 Windows Server 버전 1709와 같은 최신 릴리스에서 실행할 수 있지만 실행을 시작하기 전에 알아야 할 몇 가지 사항이 있습니다.

Windows 컨테이너 기능을 발전시켜오면서 호환성에 영향을 미칠 수 있는 사항을 몇 가지 변경해야만 했습니다. 이전 컨테이너는 [hyper-v 격리](../manage-containers/hyperv-container.md)를 사용 하 여 최신 호스트에서 동일 하 게 실행 되며 동일한 (이전) 커널 버전을 사용 합니다. 그러나 최신 Windows 빌드를 기반으로 컨테이너를 실행 하려는 경우 새 호스트 빌드에서만 실행할 수 있습니다.

>[!NOTE]
> \ * Windows Server의 버전 1709은 더 이상 지원 되지 않습니다. 자세한 내용은 [기본 이미지 서비스 주기](base-image-lifecycle.md)를 참조 하세요.

## <a name="windows-server-version-1903-host-os-compatibility"></a>Windows Server, 버전 1903 host OS 호환성

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows Server, 버전 1903|예|예|
|WindowsServer 2019|예|아니오|
|Windows Server, 버전 1803|예|아니오|
|Windows Server, version 1709 *|예|아니오|
|WindowsServer 2016|예|아니오|

## <a name="windows-server-2019-host-os-compatibility"></a>Windows Server 2019 호스트 OS 호환성

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows Server, 버전 1903|아니요|아니요|
|WindowsServer 2019|예|예|
|Windows Server, 버전 1803|예|아니오|
|Windows Server, version 1709 *|예|아니오|
|WindowsServer 2016|예|아니오|

## <a name="windows-server-version-1803-host-os-compatibility"></a>Windows Server, 버전 1803 host OS 호환성

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows Server, 버전 1903|아니요|아니요|
|WindowsServer 2019|아니요|아니요|
|Windows Server, 버전 1803|예|예|
|Windows Server, version 1709 *|예|아니오|
|WindowsServer 2016|예|아니오|

## <a name="windows-server-version-1709-host-os-compatibility"></a>Windows Server, 버전 1709 호스트 OS 호환성 *

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows Server, 버전 1903|아니요|아니요|
|WindowsServer 2019|아니요|아니요|
|Windows Server, 버전 1803|아니요|아니요|
|Windows Server, version 1709 *|예|예|
|WindowsServer 2016|예|아니오|

## <a name="windows-server-2016-host-os-compatibility"></a>Windows Server 2016 호스트 OS 호환성

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows Server 2019, 버전 1903|아니요|아니요|
|WindowsServer 2019|아니요|아니요|
|Windows Server, 버전 1803|아니요|아니요|
|Windows Server, version 1709 *|아니요|아니요|
|WindowsServer 2016|예|예|

## <a name="windows-10-version-1903-host-os-compatibility"></a>Windows 10 버전 1903 호스트 OS 호환성

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows Server, 버전 1903|예|아니오|
|WindowsServer 2019|예|아니오|
|Windows Server, 버전 1803|예|아니오|
|Windows Server, version 1709 *|예|아니오|
|WindowsServer 2016|예|아니오|

## <a name="windows-10-version-1809-host-os-compatibility"></a>Windows 10 버전 1809 호스트 OS 호환성

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows Server, 버전 1903|아니요|아니요|
|WindowsServer 2019|예|아니오|
|Windows Server, 버전 1803|예|아니오|
|Windows Server, version 1709 *|예|아니오|
|WindowsServer 2016|예|아니오|

## <a name="windows-10-version-1803-host-os-compatibility"></a>Windows 10 버전 1803 호스트 OS 호환성

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows 서비스 버전 1903|아니요|아니요|
|WindowsServer 2019|아니요|아니요|
|Windows Server, 버전 1803|예|아니오||
|Windows Server, version 1709 *|예|아니오|
|WindowsServer 2016|예|아니오|

## <a name="windows-10-fall-creators-update-host-os-compatibility"></a>Windows 10 낙하 작성자 업데이트 호스트 OS 호환성

|컨테이너 OS|' Hyper-v 격리 지원 '|프로세스 격리 지원|
|---|:---:|:---:|
|Windows Server, 버전 1903|아니요|아니요|
|WindowsServer 2019|아니요|아니요|
|Windows Server, 버전 1803|아니요|아니요|
|Windows Server, version 1709 *|예|아니오|
|WindowsServer 2016|예|아니오|

## <a name="matching-container-host-version-with-container-image-versions"></a>컨테이너 이미지 버전과 일치 하는 컨테이너 호스트 버전

### <a name="windows-server-containers"></a>Windows Server 컨테이너

Windows Server 컨테이너와 기본 호스트는 단일 커널을 공유 하기 때문에 컨테이너의 기본 이미지가 호스트와 일치 해야 합니다. 버전이 다르면 컨테이너가 시작 될 수 있지만, 완전 한 기능은 보장 되지 않습니다. Windows 운영 체제에는 주, 부, 빌드 및 수정의 네 가지 버전 관리가 있습니다. 예를 들어 버전 10.0.14393.103의 주 버전은 10, 부 버전은 0, 빌드 번호는 14393, 수정 번호는 103입니다. 빌드 번호는 버전 1709, 1803, 낙하 작성자 ' 업데이트 등의 새 OS 버전이 게시 되는 경우에만 변경 됩니다. 수정 번호는 Windows 업데이트가 적용되면 업데이트됩니다.

#### <a name="build-number-new-release-of-windows"></a>빌드 번호 (Windows의 새 릴리스)

컨테이너 호스트와 컨테이너 이미지 사이의 빌드 번호가 서로 다른 경우 Windows Server 컨테이너의 시작이 차단 됩니다. 예를 들어 컨테이너 호스트가 버전 10.0.14393 (Windows Server 2016)이 고 컨테이너 이미지가 버전 10.0.16299. * (Windows Server 버전 1709) 인 경우 컨테이너는 시작 되지 않습니다.  

#### <a name="revision-number-patching"></a>수정 번호 (패칭)

컨테이너 호스트와 컨테이너 이미지의 수정 번호가 서로 다른 경우 Windows Server 컨테이너가 시작 되지 않습니다. 예를 들어 컨테이너 호스트가 버전 10.0.14393.1914 (KB4051033 적용 된 Windows Server 2016)이 고 컨테이너 이미지가 버전 10.0.14393.1944 (Windows Server 2016를 적용 한 경우) 경우, 해당 수정에도 불구 하 고 이미지가 계속 시작 됩니다. 숫자가 다릅니다.

Windows Server 2016 기반 호스트나 이미지의 경우 컨테이너 이미지의 개정판은 지원 되는 구성으로 호스트와 일치 해야 합니다. 그러나 Windows Server 버전 1709 이상을 사용 하는 호스트 또는 이미지의 경우이 규칙이 적용 되지 않으며 호스트 및 컨테이너 이미지에는 일치 하는 수정 버전이 필요 하지 않아도 됩니다. 최신 패치와 업데이트를 사용 하 여 시스템을 최신 상태로 유지 하는 것이 좋습니다.

#### <a name="practical-application"></a>실용적인 응용 프로그램

예제 1: 컨테이너 호스트에서 KB4041691가 적용 된 Windows Server 2016를 실행 하 고 있습니다. 이 호스트에 배포 된 모든 Windows Server 컨테이너는 버전 10.0.14393.1770 컨테이너 기본 이미지를 기반으로 해야 합니다. 호스트 컨테이너에 KB4053579를 적용 하는 경우에는 호스트 컨테이너에서 해당 이미지를 지원 하는지 확인 하는 옵션도 업데이트 해야 합니다.

예제 2: 컨테이너 호스트가 KB4043961를 적용 한 Windows Server 버전 1709를 실행 하 고 있습니다. 이 호스트에 배포 된 모든 Windows Server 컨테이너는 Windows Server 버전 1709 (10.0.16299) 컨테이너 기본 이미지를 기반으로 해야 하지만 host KB를 일치 시킬 필요는 없습니다. KB4054517가 호스트에 적용 되는 경우 컨테이너 이미지는 계속 지원 되지만,이를 업데이트 하 여 잠재적인 보안 문제를 해결 하는 것이 좋습니다.

#### <a name="querying-version"></a>버전 쿼리

방법 1: 버전 1709에서 도입 된 이제 cmd prompt 및 **ver** 명령이 수정 정보를 반환 합니다.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

방법 2: 다음 레지스트리 키 쿼리: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

예를 들면 다음과 같습니다.

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

기본 이미지가 사용 하는 버전을 확인 하려면 Docker 허브 또는 이미지 설명에 제공 된 이미지 해시 테이블의 태그를 검토 합니다. [Windows 10 업데이트 기록](https://support.microsoft.com/help/12387/windows-10-update-history) 페이지에는 각 빌드 및 수정 버전이 릴리스되면 목록이 표시 됩니다.

### <a name="hyper-v-isolation-for-containers"></a>컨테이너에 대 한 hyper-v 격리

Windows 컨테이너를 Hyper-v 격리 여부에 관계 없이 실행할 수 있습니다. Hyper-V 격리는 최적화된 VM을 사용하여 컨테이너 주위에 안전한 경계를 만듭니다. 컨테이너와 호스트 간에 커널을 공유 하는 표준 Windows 컨테이너와 달리 각 Hyper-v 격리 컨테이너에는 고유한 Windows 커널의 인스턴스가 있습니다. 즉, 컨테이너 호스트와 이미지에서 다른 OS 버전을 사용할 수 있습니다 (자세한 내용은 다음 호환성 매트릭스를 참조 하세요).  

Hyper-V 격리를 사용하여 컨테이너를 실행하려면 간단하게 docker run 명령에 `--isolation=hyperv` 태그를 추가하기만 하면 됩니다.

## <a name="errors-from-mismatched-versions"></a>일치하지 않는 버전으로 인해 오류 발생

지원 되지 않는 조합을 실행 하려고 하면 다음과 같은 오류 메시지가 나타납니다.

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

이 오류는 다음과 같은 세 가지 방법으로 해결할 수 있습니다.

- 또는의 `mcr.microsoft.com/windows/nanoserver` 올바른 버전을 기반으로 컨테이너를 다시 작성 합니다. `mcr.microsoft.com/windows/servercore`
- 호스트가 최신인 경우 **docker 실행--격리 = hyperv ...** 을 실행 합니다.
- 동일한 Windows 버전을 사용 하는 다른 호스트에서 컨테이너를 실행 해 봅니다.

## <a name="choose-which-container-os-version-to-use"></a>사용할 컨테이너 OS 버전 선택

>[!NOTE]
>2019 년 4 월 16 일 기준으로 [Windows 기반 OS 컨테이너 이미지](https://hub.docker.com/_/microsoft-windows-base-os-images)에 대해 "최신" 태그가 더 이상 게시 되거나 유지 되지 않습니다. 이러한 리포지토리가에서 이미지를 가져오거나 참조할 때 특정 태그를 선언 하십시오.

컨테이너에 사용 해야 하는 버전을 알아야 합니다. 예를 들어 Windows Server 버전 1809을 컨테이너 OS로 사용 하 고 최신 패치를 설치 하려는 경우 다음과 같이 원하는 기본 OS 컨테이너 이미지 버전을 `1809` 지정할 때 태그를 사용 해야 합니다.

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

그러나 Windows Server 버전 1809의 특정 패치를 원할 경우 태그에서 KB 번호를 지정할 수 있습니다. 예를 들어 KB4493509를 적용 한 Windows Server 버전 1809에서 Nano 서버 기반 OS 컨테이너 이미지를 가져오려면 다음과 같이 지정 합니다.

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

또한 태그에 OS 버전을 지정 하 여 이전에 사용한 스키마와 함께 필요한 정확한 패치를 지정할 수도 있습니다.

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Windows Server 2019 및 Windows Server 2016을 기반으로 하는 서버 핵심 기본 이미지는 [장기 서비스 채널 (LTSC)](https://docs.microsoft.com/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) 릴리스입니다. Windows Server 2019을 서버 핵심 이미지의 컨테이너 OS로 사용 하 고 최신 패치를 설치 하려는 경우 다음과 같이 LTSC 릴리스를 지정할 수 있습니다.

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Docker Swarm을 사용하여 버전 일치시키기

Docker Swarm에는 컨테이너가 동일한 버전의 호스트에 사용 하는 Windows 버전과 일치 하는 기본 제공 방법이 없습니다. 최신 컨테이너를 사용 하도록 서비스를 업데이트 하면 성공적으로 실행 됩니다.

오랜 시간 동안 여러 버전의 Windows를 실행 해야 하는 경우 다음 두 가지 방법으로 수행할 수 있습니다. 항상 Hyper-v 격리를 사용 하도록 Windows 호스트를 구성 하거나 레이블 제약 조건을 사용 합니다.

### <a name="finding-a-service-that-wont-start"></a>시작하지 않는 서비스 찾기

서비스가 시작 되지 않으면 `MODE` is가 `replicated` `REPLICAS` 0에서 중지 됨을 확인할 수 있습니다. OS 버전이 문제 인지 확인 하려면 다음 명령을 실행 합니다.

서비스 이름을 찾으려면 **docker 서비스 ls** 를 실행 합니다.

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

상태 및 최신 시도를 얻으려면 **docker 서비스 ps (서비스 이름)** 를 실행 합니다.

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

표시 `starting container failed: ...`되는 경우 **docker 서비스 ps--trunc (컨테이너 이름)** 에 대 한 전체 오류를 확인할 수 있습니다.

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

이 오류는과 동일 합니다 `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>수전 - 일치하는 버전을 사용하도록 서비스 업데이트

Docker Swarm을 사용할 때는 다음과 같은 두 가지 사항을 고려해야 합니다. 자신이 만들지 않은 이미지를 사용 하는 서비스가 있는 구성 파일이 있는 경우에는이에 따라 참조를 업데이트 해야 할 수 있습니다. 예를 들면 다음과 같습니다.

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

다른 고려 사항은 사용자가 직접 만든 이미지 (예: contoso/myimage)가 가리키는 경우입니다.

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

이 경우 [일치 하지 않는 버전의 오류](#errors-from-mismatched-versions) 에 설명 된 메서드를 사용 하 여 docker 작성 줄 대신 dockerfile을 수정 해야 합니다.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>마이그레이션 - Docker Swarm을 통해 Hyper-V 격리 사용

컨테이너 별로 Hyper-v 격리를 사용 하는 방법을 제안 하는 제안이 있지만 아직 코드는 완료 되지 않았습니다. [GitHub](https://github.com/moby/moby/issues/31616)에서 진행 상황을 확인할 수 있습니다. 코드가 완성될 때까지 호스트는 항상 Hyper-V 격리로 실행되도록 구성해야 합니다.

이렇게 하려면 Docker 서비스 구성을 변경한 다음 Docker 엔진을 다시 시작해야 합니다.

1. 편집 `C:\ProgramData\docker\config\daemon.json`
2. 다음 줄 추가 `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >디먼 파일은 기본적으로 존재 하지 않습니다. 디렉터리를 미리 보는 경우라면 파일을 만들어야 합니다. 다음과 같은 경우에 복사 합니다.

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. 파일을 닫고 저장 한 다음 PowerShell에서 다음 cmdlet을 실행 하 여 docker 엔진을 다시 시작 합니다.

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. 서비스를 다시 시작한 후 컨테이너를 실행 합니다. 이 작업이 실행 되 면 다음 cmdlet을 사용 하 여 컨테이너를 검사 하 여 컨테이너의 격리 수준을 확인할 수 있습니다.

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

"process" 또는 "hyperv"를 반환합니다. daemon.json을 수정하여 위에 설명한 대로 설정한 경우 후자로 표시되어야 합니다.

### <a name="mitigation---use-labels-and-constraints"></a>완화 방법: 레이블 및 제약 조건 사용

레이블과 제약 조건을 사용 하 여 버전을 일치 시키는 방법은 다음과 같습니다.

1. 각 노드에 레이블을 추가 합니다.

    각 노드에서 다음과 같이 `OS` 두 개의 레이블을 추가 `OsVersion`합니다. 로컬로 실행하고 있다고 가정하지만 원격 호스트에서 설정하도록 수정할 수 있습니다.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    나중에, **docker 노드 검사** 명령을 실행 하 여 해당 레이블을 확인 하 고 새로 추가 된 레이블이 표시 되도록 할 수 있습니다.

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. 서비스 제약 조건을 추가 합니다.

    이제 각 노드의 레이블을 지정 했으므로 서비스의 배치를 결정 하는 제약 조건을 업데이트할 수 있습니다. 다음 예제에서는 "contoso_service"를 실제 서비스의 이름으로 바꿉니다.

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    이렇게 하면 노드가 실행될 수 있는 위치가 강제하고 제한됩니다.

서비스 제약 조건을 사용 하는 방법에 대 한 자세한 내용은 [서비스 만들기 참조](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)를 참고 하세요.

## <a name="matching-versions-using-kubernetes"></a>Kubernetes를 통한 버전 일치

[Swarm를 사용 하 여 일치](#matching-versions-using-docker-swarm) 하는 버전에서 설명한 것과 동일한 문제는 Kubernetes에서 pods를 예약할 때 발생할 수 있습니다. 다음과 유사한 방법으로이 문제를 방지할 수 있습니다.

- 개발 및 프로덕션에서 동일한 OS 버전을 기반으로 컨테이너를 다시 작성 합니다. 방법을 알아보려면 [사용할 컨테이너 OS 버전 선택](#choose-which-container-os-version-to-use)을 참조 하세요.
- Windows Server 2016 및 Windows Server 버전 1709 노드가 모두 동일한 클러스터에 있는 경우 노드 레이블 및 nodeSelectors를 사용 하 여 호환 되는 노드에 pods가 예약 되어 있는지 확인 합니다.
- OS 버전을 기반으로 별도의 클러스터를 사용합니다.

### <a name="finding-pods-failed-on-os-mismatch"></a>OS 불일치에서 실패한 포드 찾기

이 경우 배포에는 OS 버전이 일치 하지 않고 Hyper-v 격리를 사용 하지 않는 경우 노드에 예약 된 pod가 포함 되어 있습니다.

동일한 오류가 이벤트에 표시되며 `kubectl describe pod <podname>`와 함께 나열됩니다. 몇 번의 시도 후에는 pod 상태가 될 `CrashLoopBackOff`수 있습니다.

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>완화-노드 레이블 및 nodeSelector 사용

**Kubectl get 노드** 를 실행 하 여 모든 노드의 목록을 가져옵니다. 그런 다음 **kubectl 설명 노드 (노드 이름)** 를 실행 하 여 자세한 정보를 얻을 수 있습니다.

다음 예제에서는 두 개의 Windows 노드가 서로 다른 버전을 실행 하 고 있습니다.

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

이 예제를 사용 하 여 버전과 일치 하는 방법을 보여 드리겠습니다.

1. 각 노드 이름 및 `Kernel Version` 시스템 정보를 기록해 둡니다.

    이 예제에서는 정보가 다음과 같이 표시 됩니다.

    이름         | 버전
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. 레이블을 `beta.kubernetes.io/osbuild`라는 각 노드에 추가합니다. Hyper-v 격리 없이는 Windows Server 2016에 주 버전과 부 버전 (이 예제의 경우 14393.1715)을 모두 지원 해야 합니다. Windows Server 버전 1709에는 일치 하는 주 버전 (이 예에서는 16299)만 필요 합니다.

    이 예제에서는 레이블을 추가 하는 명령이 다음과 같이 표시 됩니다.

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. **Kubectl get 노드 (표시-레이블**)를 실행 하 여 레이블을 선택 합니다.

    이 예제에서 출력은 다음과 같이 표시 됩니다.

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. 배포에 노드 선택기를 추가 합니다. 이 예제에서는 컨테이너에 사용 되는 기본 `nodeSelector` OS와 일치 하도록 = `beta.kubernetes.io/os` windows 및 `beta.kubernetes.io/osbuild` = 14393. * 또는 16299를 사용 하 여 컨테이너 사양에 a를 추가 합니다.

    Windows Server 2016용으로 만들어진 컨테이너를 실행하는 전체 샘플 코드는 다음과 같습니다.

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    이제 포드는 업데이트된 배포에서 시작할 수 있습니다. 노드 선택기도에 `kubectl describe pod <podname>`표시 되므로 해당 명령을 실행 하 여 추가 되었는지 확인할 수 있습니다.

    이 예제에 대 한 출력은 다음과 같습니다.

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```
