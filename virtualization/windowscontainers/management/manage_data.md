---
title: "컨테이너 데이터 볼륨"
description: "Windows 컨테이너로 데이터 볼륨을 만들고 관리합니다."
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
redirect_url: https://docs.docker.com/engine/tutorials/dockervolumes/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 60dc25d879b5c8de6515899d9de9e920a08b7e62

---

# 컨테이너 데이터 볼륨

**이 예비 콘텐츠는 변경될 수 있습니다.** 

컨테이너를 만들 때 새 데이터 디렉터리를 만들거나 기존 디렉터리를 컨테이너에 추가해야 할 수 있습니다. 이 작업은 데이터 볼륨 추가를 통해 수행할 수 있습니다. 데이터 볼륨은 컨테이너와 컨테이너 호스트에 모두 표시되며 서로 간에 데이터를 공유할 수 있습니다. 또한 데이터 볼륨은 동일한 컨테이너 호스트에 있는 여러 컨테이너 간에 공유할 수 있습니다. 이 문서에서는 데이터 볼륨 만들기, 검사 및 제거에 대해 자세히 설명합니다.

## 데이터 볼륨

### 새 데이터 볼륨 만들기

`docker run` 명령의 `-v` 매개 변수를 사용하여 새 데이터 볼륨을 만듭니다. 기본적으로 새 데이터 볼륨은 호스트의 'c:\ProgramData\Docker\volumes' 아래에 저장됩니다.

다음 예제에서는 'new-data-volume'이라는 데이터 볼륨을 만듭니다. 이 데이터 볼륨은 실행 중인 컨테이너의 'c:\new-data-volume'에서 액세스할 수 있습니다.

```none
docker run -it -v c:\new-data-volume windowsservercore cmd
```

볼륨 만들기에 대한 자세한 내용은 [docker.com의 Manage data in containers(컨테이너에서 데이터 관리)](https://docs.docker.com/engine/userguide/containers/dockervolumes/#data-volumes)를 참조하세요.

### 기존 디렉터리 탑재

새 데이터 볼륨을 만들 뿐만 아니라 호스트에서 컨테이너로 기존 디렉터리를 전달할 수도 있습니다. 이 작업도 `docker run` 명령의 `-v` 매개 변수를 사용하여 수행할 수 있습니다. 호스트 디렉터리 내의 모든 파일은 컨테이너에서도 사용할 수 있습니다. 탑재된 볼륨에서 컨테이너에 의해 만들어진 모든 파일은 호스트에서 사용할 수 있습니다. 동일한 디렉터리를 많은 컨테이너에 탑재할 수 있습니다. 이 구성에서는 컨테이너 간에 데이터를 공유할 수 있습니다.

다음 예제에서는 원본 디렉터리 'c:\source'가 'c:\destination'으로 컨테이너에 탑재됩니다.

```none
docker run -it -v c:\source:c:\destination windowsservercore cmd
```

호스트 디렉터리 탑재에 대한 자세한 내용은 [docker.com의 Manage data in containers(컨테이너에서 데이터 관리)](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume)를 참조하세요.

### 단일 파일 탑재

Windows 컨테이너에 단일 파일을 탑재할 수 없습니다. 다음 명령을 실행하면 실패하지 않지만 결과 컨테이너 파일이 포함되지 않습니다. 

```none
docker run -it -v c:\config\config.ini microsoft/windowsservercore cmd
```

이 문제를 해결하려면 컨테이너에 탑재된 모든 파일을 디렉터리에서 탑재해야 합니다.

```none
docker run -it -v c:\config:c:\config microsoft/windowsservercore cmd
```

### 전체 드라이브 탑재

다음과 유사한 명령을 사용하여 전체 드라이브를 탑재할 수 있습니다.

```none
docker run -it -v d:\:d: windowsservercore cmd
```

현재 두 번째 드라이브의 일부를 탑재할 수는 없습니다. 예를 들어 다음은 가능하지 않습니다.

```none
docker run -it -v d:\source:d:\destination windowsservercore cmd
```

### 데이터 볼륨 컨테이너

데이터 볼륨은 `docker run` 명령의 `--volumes-from` 매개 변수를 사용하여 실행 중인 다른 컨테이너에서 상속할 수 있습니다. 이 상속을 사용하면 컨테이너화된 응용 프로그램의 데이터 볼륨을 호스트하려는 명시적인 용도로 컨테이너를 만들 수 있습니다. 

다음 예제에서는 'cocky_bell' 컨테이너에서 새 컨테이너로 데이터 볼륨을 탑재합니다. 새 컨테이너가 시작된 후에는 이 볼륨에 있는 데이터를 컨테이너에서 실행 중인 응용 프로그램에 사용할 수 있습니다.  

```none
docker run -it --volumes-from cocky_bell windowsservercore cmd
```

데이터 컨테이너에 대한 자세한 내용은 [docker.com의 Manage data in containers(컨테이너에서 데이터 관리)](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-file-as-a-data-volume)를 참조하세요.

### 공유 데이터 볼륨 검사

탑재된 볼륨은 `docker inspect` 명령을 사용하여 볼 수 있습니다.

```none
docker inspect backstabbing_kowalevski
```

그러면 ‘Mounts’라는 섹션을 비롯하여 컨테이너에 대한 정보가 반환되며, 원본 및 대상 디렉터리와 같이 탑재된 볼륨에 대한 데이터를 포함합니다.

```none
"Mounts": [
    {
        "Source": "c:\\container-share",
        "Destination": "c:\\data",
        "Mode": "",
        "RW": true,
        "Propagation": ""
}
```

볼륨 검사에 대한 자세한 내용은 [docker.com의 Manage data in containers(컨테이너에서 데이터 관리)](https://docs.docker.com/engine/userguide/containers/dockervolumes/#locating-a-volume)를 참조하세요.




<!--HONumber=Sep16_HO2-->


