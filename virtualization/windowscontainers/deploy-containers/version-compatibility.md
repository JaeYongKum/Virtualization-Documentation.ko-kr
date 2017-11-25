---
title: "Windows 컨테이너 버전 호환성"
description: "Windows에서 빌드를 실행하고 다양한 버전 간에 컨테이너를 실행할 수 있는 방법"
keywords: "메타데이터, 컨테이너, 버전"
author: patricklang
ms.openlocfilehash: ed9d88e1e861651426e560a4531fd4added2134a
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/08/2017
---
# <a name="windows-container-version-compatibility"></a>Windows 컨테이너 버전 호환성

Windows Server 2016 및 Windows 10 1주년 업데이트(두 버전 모두 14393)는 Windows Server 컨테이너를 빌드하고 실행할 수 있었던 최초의 Windows 릴리스였습니다. 이러한 버전을 사용하여 빌드된 컨테이너는 Windows Server 버전 1709와 같은 최신 릴리스에서 실행할 수 있지만 실행을 시작하기 전에 알아야 할 몇 가지 사항이 있습니다.

Windows 컨테이너 기능을 발전시켜오면서 호환성에 영향을 미칠 수 있는 사항을 몇 가지 변경해야만 했습니다. 기존 컨테이너는 [Hyper-V 격리](../manage-containers/hyperv-container.md)를 통해 최신 호스트에서 동일하게 실행되고 동일한 기존 커널 버전이 계속 사용됩니다. 그러나 최신 Windows 빌드를 기반으로 컨테이너를 실행하려면 최신 호스트 빌드에서만 실행할 수 있습니다.



<table>
    <tr>
    <th>컨테이너 OS 버전</th>
    <th span='2'>호스트 OS 버전</th>
    </tr>
    <tr>
        <td/>
        <td><b>Windows Server 2016 / Windows 10 1609, 1703</b><br/>빌드: 14393.*</td>
        <td><b>Windows Server 버전 1709 / Windows 10 Fall Creators Update</b><br/>빌드 16299.*</td>
    </tr>
    <tr>
        <td><b>Windows Server 2016 / Windows 10 1609, 1703</b><br/>빌드: 14393.*</td>
        <td>지원됨. `process` 또는 `hyperv` 격리</td>
        <td>지원됨. `hyperv` 격리</td>
    </tr>
    <tr>
        <td><b>Windows Server 버전 1709 / Windows 10 Fall Creators Update</b><br/>빌드 16299.*</td>
        <td>지원되지 않음</td>
        <td>지원됨. `process` 또는 `hyperv` 격리</td>
    </tr>
</table>               


## <a name="errors-from-mismatched-versions"></a>일치하지 않는 버전으로 인해 오류 발생

지원되지 않은 조합을 실행하려는 경우 다음과 같은 오류가 발생합니다.

```
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

이 문제를 해결하려면 다음과 같이 할 수 있습니다.

- `microsoft/nanoserver` 버전을 기반으로 컨테이너를 다시 빌드하거나 `microsoft/windowsservercore`
- 호스트가 최신 버전인 경우 `docker run --isolation=hyperv ...`
- 동일한 Windows 버전의 다른 호스트에서 실행을 사용합니다.

## <a name="choosing-container-os-versions"></a>컨테이너 OS 버전 선택

> 참고: "최신" 태그는 Windows Server 2016, 현재 [LTSC 제품](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview)과 함께 업데이트됩니다. 컨테이너 이미지가 Windows Server 버전 1709 릴리스와 일치시키려면 다음 내용을 읽어보세요.

사용자의 목적에 맞는 필요한 컨테이너 OS 버전이 무엇인지 알아야 합니다. Windows Server 버전 1709를 사용 중인데 이 버전에 대한 최신 패치를 얻으려면 원하는 기본 OS 컨테이너 이미지 버전을 지정할 때 "1709"라는 태그를 사용해야 합니다.

``` Dockerfile
FROM microsoft/windowsservercore:1709
...
```

그러나 Windows Server 버전 1709의 특정 패치를 원하는 경우 태그에 KB 번호를 지정할 수 있습니다. 예를 들어 Windows Server 버전 1709에서 Nano Server 기본 OS 컨테이너 이미지를 원하는 경우, 여기에 KB4043961이 적용되도록 지정할 수 있습니다.

``` Dockerfile
FROM microsoft/nanoserver:1709_KB4043961
...
```

Windows Server 2016에서 Nano Server 기본 OS 컨테이너 이미지가 필요한 경우 "최신"이라는 태그를 사용하여 이러한 기본 OS 컨테이너 이미지의 최신 버전을 얻을 수도 있습니다.

``` Dockerfile
FROM microsoft/nanoserver
...
```
태그에 OS 버전을 지정하여 이전에 사용했던 스키마에 필요한 동일한 패치를 계속 지정할 수도 있습니다.

``` Dockerfile
FROM microsoft/nanoserver:10.0.14393.1770
...
```

## <a name="matching-versions-using-docker-swarm"></a>Docker Swarm을 사용하여 버전 일치시키기

현재 Docker Swarm에는 컨테이너가 동일한 버전의 호스트 매칭에 사용하는 Windows 버전과 일치시키는 기본 제공 방식은 없습니다. 최신 컨테이너를 사용하도록 서비스가 업데이트되면 성공적으로 실행됩니다.

잠시 동안 다양한 Windows 버전을 실행해야 하는 경우 사용할 수 있는 접근 방식이 2가지 있습니다.  항상 Hyper-V 격리를 사용하거나 레이블 상수를 사용하도록 Windows 호스트를 구성합니다.

### <a name="finding-a-service-that-wont-start"></a>시작하지 않는 서비스 찾기

서비스가 시작되지 않으면 `MODE`는 `replicated`지만 `REPLICAS`는 0에서 문제가 발생합니다. OS 버전 문제인지 확인하려면 다음 명령을 사용하세요.

 `docker service ls` - 서비스 이름을 찾습니다.

```
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

`docker service ps <name>` - 상태 및 최근 시도 정보를 가져옵니다.

```
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

"컨테이너 시작 실패: ..."와 같은 메시지가 표시되면 다음 내용을 포함하는 전체 오류가 나타날 수 있습니다. `docker service ps --no-trunc <container name>`


```
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

동일한 오류를 보여주는 내용 `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`


### <a name="fix---update-the-service-to-use-a-matching-version"></a>수전 - 일치하는 버전을 사용하도록 서비스 업데이트

Docker Swarm을 사용할 때는 다음과 같은 두 가지 사항을 고려해야 합니다. 본인이 작성하지 않은 이미지를 사용하는 서비스가 있는 파일을 만들 때 참조를 적절히 업데이트하려는 경우가 있습니다. 아래 내용을 참조하세요.

``` docker-compose
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

나머지 하나는 가리키고 있는 이미지가 본인이 직접 만든 이미지인 다음과 같은 경우(예: contoso/myimage)입니다.
``` docker-compose
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```
이 경우 이전 섹션에 설명된 메서드를 사용하여 docker-compose 줄 대신 dockerfile을 수정할 수 있습니다.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>마이그레이션 - Docker Swarm을 통해 Hyper-V 격리 사용

컨테이너 단위로 Hyper-V 격리를 사용하여 지원하는 제안이 있지만 코드가 아직 완성되지 않았습니다. [GitHub](https://github.com/moby/moby/issues/31616)에서 진행 상황을 확인할 수 있습니다. 코드가 완성될 때까지 호스트는 항상 Hyper-V 격리로 실행되도록 구성해야 합니다.

이렇게 하려면 Docker 서비스 구성을 변경한 다음 Docker 엔진을 다시 시작해야 합니다.

1. 편집 `C:\ProgramData\docker\config\daemon.json`
2. 다음 줄 추가 `"exec-opts":["isolation=hyperv"]`

참고: daemon.json 파일은 기본적으로 존재하지 않습니다. 디렉터리를 미리 보는 경우라면 파일을 만들어야 합니다. 그런 다음 아래 내용에 복사할 수 있습니다.

```JSON
{
    "exec-opts":["isolation=hyperv"]
}
```
파일을 닫고 저장합니다. 그런 다음 docker 엔진을 다시 시작합니다.

```PowerShell
Stop-Service docker
Start-Service docker
```

서비스를 다시 시작한 후 컨테이너를 실행합니다. 컨테이너가 실행되면 컨테이너를 검사하여 컨테이너의 격리 수준을 확인할 수 있습니다.

```PowerShell
docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
```

"process" 또는 "hyperv"를 반환합니다. daemon.json을 수정하여 위에 설명한 대로 설정한 경우 후자로 표시되어야 합니다.

### <a name="mitigation---use-labels-and-constraints"></a>완화 방법: 레이블 및 제약 조건 사용

**1단계 각 노드에 레이블 추가**

각 노드에서 `OS` 및 `OsVersion`이라는 2개의 레이블을 추가합니다. 로컬로 실행하고 있다고 가정하지만 원격 호스트에서 설정하도록 수정할 수 있습니다.

```powershell
docker node update --label-add OS="windows" $ENV:COMPUTERNAME
docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
```

이후에 `docker node inspect`를 사용하여 이를 검사할 수 있습니다. 이렇게 하면 추가된 새 레이블이 표시됩니다.

```
        "Spec": {
            "Labels": {
                "OS": "windows",
                "OsVersion": "10.0.16296"
            },
            "Role": "manager",
            "Availability": "active"
        }
```

**2단계 - 서비스 제약 조건 추가**

각 노드에 레이블이 지정되었으면 서비스 배치를 결정하는 제약 조건을 업데이트할 수 있습니다. 아래 예에서는 "contoso_service"를 실제 서비스 이름과 대체합니다.

```
docker service update \
    --constraint-add "node.labels.OS == windows" \
    --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
    contoso_service
```

이렇게 하면 노드가 실행될 수 있는 위치가 강제하고 제한됩니다.

서비스 제약 조건을 방법에 대한 자세한 내용은 서비스 생성 [참조](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)를 확인하세요.


## <a name="matching-versions-using-kubernetes"></a>Kubernetes를 통한 버전 일치

포드가 Kubernetes에 예약되어 있는 경우 동일한 문제가 발생할 수 있습니다. 다음과 유사한 전략을 사용하여 이러한 문제를 방지할 수 있습니다.

- 배포 및 제작 시 동일한 OS 버전을 기반으로 컨테이너를 다시 빌드합니다. - 위의 **컨테이너 OS 버전 선택** 참조
- Windows Server 2016 및 Windows Server 버전 1709 노드가 동일한 클러스터에 있는 경우 노드 레이블 및 nodeSelectors를 사용하여 포드가 호환되는 노드에 예정되어 있는지 확인합니다.
- OS 버전을 기반으로 별도의 클러스터를 사용합니다.


### <a name="finding-pods-failed-on-os-mismatch"></a>OS 불일치에서 실패한 포드 찾기

OS 버전이 불일치하는 노드에 예정된 포드가 배포에 포함되었으며 Hyper-V 격리를 사용하도록 설정되지 않는 경우입니다. 동일한 오류가 이벤트에 표시되며 `kubectl describe pod <podname>`와 함께 나열됩니다. 몇 번의 시도 후 포드 상태가 다음과 같이 나타날 것입니다. `CrashLoopBackOff`

```
$ kubectl -n plang describe po fabrikamfiber.web-789699744-rqv6p

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


### <a name="mitigation---using-node-labels--nodeselector"></a>완화 방법 - 노드 레이블 및 NodeSelector 사용

`kubectl get node` 는 모든 노드 목록을 가져오고, `kubectl describe node <nodename>`을 사용하여 자세한 정보를 얻을 수 있습니다. 

이 경우 다른 버전을 실행하는 두 개의 Windows 노드가 있습니다.

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


1. 노드 이름과 시스템 정보의 `Kernel Version`을 기록해 둡니다.

이름         | 버전
-------------|--------------------------------------------------------
38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354



2. 레이블을 `beta.kubernetes.io/osbuild`라는 각 노드에 추가합니다. Hyper-V 격리 없이 Windows Server 2016을 지원하려면 주 버전 및 부 버전(14393.1715)이 모두 필요합니다. Windows Server 버전 1709만 주 버전(16299)과 일치해야 합니다.

위 샘플에서 가져온 아래 내용을 참조하세요.

```
$ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


node "38519acs9010" labeled
$ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

node "38519acs9011" labeled

```

3. 다음 줄을 사용하여 레이블이 있는지 확인합니다. `kubectl get nodes --show-labels`

```
$ kubectl get nodes --show-labels

NAME                        STATUS                     AGE       VERSION                    LABELS
38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
```

4. 배포에 노드 선택기 추가

`beta.kubernetes.io/os` = windows 및 `beta.kubernetes.io/osbuild` = 14393.* 또는 16299를 사용하여 컨테이너 spec에 `nodeSelector`를 추가해 컨테이너에서 사용되는 기본 OS를 일치시킵니다.

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

이제 포드는 업데이트된 배포에서 시작할 수 있습니다. 노드 선택기가 `kubectl describe pod <podname>`에도 표시되므로 노드 선택기가 추가되었음을 확인할 수 있습니다.

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
