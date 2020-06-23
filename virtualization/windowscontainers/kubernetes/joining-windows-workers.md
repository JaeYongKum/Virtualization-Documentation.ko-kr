---
title: Windows 노드 조인
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: how-to
ms.prod: containers
description: V 1.14를 사용 하 여 Windows 노드를 Kubernetes 클러스터에 조인 합니다.
keywords: kubernetes, 1.14, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: f808428547a0134e6fea2d9165a4b5cee35b6cfb
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192650"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>클러스터에 Windows Server 노드 조인 #
[Kubernetes 마스터 노드를 설정](./creating-a-linux-master.md) 하 고 [원하는 네트워크 솔루션을 선택한](./network-topologies.md)후에는 Windows Server 노드를 조인 하 여 클러스터를 구성할 준비가 된 것입니다. 이렇게 하려면 조인 하기 전에 [Windows 노드에서](#preparing-a-windows-node) 몇 가지 준비가 필요 합니다.

## <a name="preparing-a-windows-node"></a>Windows 노드 준비 ##
> [!NOTE]
> Windows 섹션의 모든 코드 조각은 _관리자 권한_ PowerShell에서 실행 됩니다.

### <a name="install-docker-requires-reboot"></a>Docker 설치 (다시 부팅 필요) ###
Kubernetes는 [Docker](https://www.docker.com/) 를 컨테이너 엔진으로 사용 하므로 설치 해야 합니다. [공식 문서 지침](../manage-docker/configure-docker-daemon.md#install-docker)과 [Docker 지침](https://store.docker.com/editions/enterprise/docker-ee-server-windows)을 따르거나 다음 단계를 수행해 볼 수 있습니다.

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

프록시 뒤에 있는 경우 다음과 같은 PowerShell 환경 변수를 정의 해야 합니다.
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

다시 부팅 한 후 다음 오류가 표시 됩니다.

![text](media/docker-svc-error.png)

그런 다음 docker 서비스를 수동으로 시작 합니다.

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>"일시 중지" (인프라) 이미지 만들기 ###
> [!Important]
> 충돌 하는 컨테이너 이미지에 주의 해야 합니다. 필요한 태그가 없으면 `docker pull` 호환 되지 않는 컨테이너 이미지의가 발생 하 여 무한 상태와 같은 [배포 문제](./common-problems.md#when-deploying-docker-containers-keep-restarting) 를 일으킬 수 있습니다 `ContainerCreating` .

이제를 `docker` 설치 했으므로 Kubernetes에서 인프라 pod을 준비 하는 데 사용 하는 "일시 중지" 이미지를 준비 해야 합니다. 이에 대 한 세 가지 단계가 있습니다.
  1. [이미지 끌어오기](#pull-the-image)
  2. microsoft/nanoserver로 [태그](#tag-the-image) 지정: 최신
  3. 및 [실행](#run-the-container)


#### <a name="pull-the-image"></a>이미지 끌어오기 ####
 특정 Windows 릴리스에 대 한 이미지를 끌어옵니다. 예를 들어 Windows Server 2019를 실행 하는 경우:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>이미지 태그 ####
이 가이드의 뒷부분에서 사용 하는 Dockerfiles은 `:latest` 이미지 태그를 찾습니다. 다음과 같이 방금 끌어온 nanoserver 이미지에 태그를 표시 합니다.

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>컨테이너를 실행 합니다. ####
컨테이너가 컴퓨터에서 실제로 실행 되는지 두 번 확인 합니다.

```powershell
docker run microsoft/nanoserver:latest
```

다음과 유사한 결과가 표시됩니다.

![text](./media/docker-run-sample.png)

> [!tip]
> 컨테이너를 실행할 수 없는 경우 컨테이너 [호스트 버전 일치 컨테이너 이미지를](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions) 참조 하세요.


#### <a name="prepare-kubernetes-for-windows-directory"></a>Windows 디렉터리에 대 한 Kubernetes 준비 ####
Kubernetes 이진 파일 뿐만 아니라 모든 배포 스크립트와 구성 파일을 저장 하는 "Kubernetes for Windows" 디렉터리를 만듭니다.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Kubernetes certificate 복사 ####
Kubernetes certificate 파일 ()을 `$HOME/.kube/config` [master에서](./creating-a-linux-master.md#collect-cluster-information) 이 새 디렉터리에 복사 `C:\k` 합니다.

> [!tip]
> [Xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) 또는 [winscp](https://winscp.net/eng/download.php) 와 같은 도구를 사용 하 여 노드 간에 구성 파일을 전송할 수 있습니다.

#### <a name="download-kubernetes-binaries"></a>Kubernetes 이진 파일 다운로드 ####
Kubernetes를 실행할 수 있으려면 먼저 `kubectl` , `kubelet` 및 이진 파일을 다운로드 해야 `kube-proxy` 합니다. `CHANGELOG.md` [최신 릴리스](https://github.com/kubernetes/kubernetes/releases/)파일의 링크에서 다운로드할 수 있습니다.
 - 예를 들어 다음은 [v 1.14 Node 이진 파일](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries)입니다.
 - [확장-아카이브](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) 와 같은 도구를 사용 하 여 보관 파일을 추출 하 고 이진 파일을에 저장 `C:\k\` 합니다.

#### <a name="optional-setup-kubectl-on-windows"></a>필드 Windows에서 kubectl 설정 ####
Windows에서 클러스터를 제어 하려는 경우 명령을 사용 하 여이 작업을 수행할 수 있습니다 `kubectl` . 먼저 `kubectl` 디렉터리 외부에서 사용할 수 있도록 하려면 `C:\k\` 환경 변수를 수정 합니다 `PATH` .

```powershell
$env:Path += ";C:\k"
```

이 변경 내용을 영구적으로 적용 하려면 컴퓨터 대상에서 변수를 수정 합니다.

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

다음으로 [클러스터 인증서](#copy-kubernetes-certificate) 가 유효한 지 확인 합니다. 에서 구성 파일을 찾을 위치를 설정 하기 위해 `kubectl` `--kubeconfig` 매개 변수를 전달 하거나 환경 변수를 수정할 수 있습니다 `KUBECONFIG` . 예를 들어 구성이 다음 위치에 있는 경우 `C:\k\config` :

```powershell
$env:KUBECONFIG="C:\k\config"
```

현재 사용자의 범위에 대해이 설정을 영구적으로 설정 하려면:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

마지막으로, 구성이 제대로 검색 되었는지 확인 하려면 다음을 사용할 수 있습니다.

```powershell
kubectl config view
```

연결 오류를 수신 하는 경우

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Kubeconfig 위치를 다시 확인 하거나 다시 복사 하십시오.

오류가 표시 되지 않으면 노드가 클러스터에 조인할 준비가 된 것입니다.

## <a name="joining-the-windows-node"></a>Windows 노드 조인 ##
[선택한 네트워킹 솔루션](./network-topologies.md)에 따라 다음을 수행할 수 있습니다.
1. [Flannel (vxlan 또는 gw) 클러스터에 Windows Server 노드를 조인 합니다.](#joining-a-flannel-cluster)
2. [연결 스위치를 사용 하 여 클러스터에 Windows Server 노드 연결](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Flannel 클러스터 조인 ###
이 [Microsoft 리포지토리에](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) 는이 노드를 클러스터에 연결 하는 데 도움이 되는 Flannel deployment 스크립트 모음이 있습니다.

[Flannel start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) 스크립트를 다운로드 합니다 .이 스크립트의 내용은 다음과 같이 추출 되어야 합니다 `C:\k` .

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

[Windows 노드를 준비](#preparing-a-windows-node)하 고 `c:\k` 디렉터리를 아래와 같이 표시 하면 노드에 가입할 준비가 된 것입니다.

![text](./media/flannel-directory.png)

#### <a name="join-node"></a>조인 노드 ####
Windows 노드의 조인 프로세스를 간소화 하려면 단일 windows 스크립트만 실행 하 여 노드를 시작 하 고, 및를 `kubelet` `kube-proxy` 조인 해야 `flanneld` 합니다.

> [!Note]
> [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) 는 [ ](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1) `flanneld` [인프라 pod 용 dockerfile](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) 및 실행 파일과 같은 추가 파일을 다운로드 *하 고 설치*하는install.ps1을 참조 합니다. 오버레이 네트워킹 모드의 경우 로컬 UDP 포트 4789에 대 한 [방화벽이](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) 열립니다. Pod 네트워크에 대 한 새 외부 vSwitch를 처음 만드는 동안 몇 초 정도의 네트워크 중단 및 여러 powershell 창을 열거나 닫을 수 있습니다.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# <a name="managementip"></a>[ManagementIP](#tab/ManagementIP)
Windows 노드에 할당 된 IP 주소입니다. 를 사용 하 여이를 찾을 수 있습니다 `ipconfig` .

|  |  |
|---------|---------|
|매개 변수     | `-ManagementIP`        |
|기본값    | N.a. **필수**        |

# <a name="networkmode"></a>[NetworkMode](#tab/NetworkMode)
네트워크 `l2bridge` 솔루션으로 선택 된 네트워크 모드 (flannel 호스트-gw) 또는 `overlay` (flannel vxlan [network solution](./network-topologies.md))입니다.

> [!Important]
> `overlay`네트워킹 모드 (flannel vxlan)에는 Kubernetes v 1.14 이진 (이상) 및 [KB4489899](https://support.microsoft.com/help/4489899)가 필요 합니다.

|  |  |
|---------|---------|
|매개 변수     | `-NetworkMode`        |
|기본값    | `l2bridge`        |


# <a name="clustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
[클러스터 서브넷 범위](./getting-started-kubernetes-windows.md#cluster-subnet-def)입니다.

|  |  |
|---------|---------|
|매개 변수     | `-ClusterCIDR`        |
|기본값    | `10.244.0.0/16`        |


# <a name="servicecidr"></a>[ServiceCIDR](#tab/ServiceCIDR)
[서비스 서브넷 범위](./getting-started-kubernetes-windows.md#service-subnet-def)입니다.

|  |  |
|---------|---------|
|매개 변수     | `-ServiceCIDR`        |
|기본값    | `10.96.0.0/12`        |


# <a name="kubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS 서비스 IP](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster)입니다.

|  |  |
|---------|---------|
|매개 변수     | `-KubeDnsServiceIP`        |
|기본값    | `10.96.0.10`        |


# <a name="interfacename"></a>[InterfaceName](#tab/InterfaceName)
Windows 호스트의 네트워크 인터페이스 이름입니다. 를 사용 하 여이를 찾을 수 있습니다 `ipconfig` .

|  |  |
|---------|---------|
|매개 변수     | `-InterfaceName`        |
|기본값    | `Ethernet`        |


# <a name="logdir"></a>[LogDir](#tab/LogDir)
Kubelet 및 kube 로그가 해당 출력 파일로 리디렉션되는 디렉터리입니다.

|  |  |
|---------|---------|
|매개 변수     | `-LogDir`        |
|기본값    | `C:\k`        |


---

> [!tip]
> [이전](./creating-a-linux-master.md#collect-cluster-information) 에 Linux 마스터에서 클러스터 서브넷, 서비스 서브넷 및 kube IP를 이미 적어 둔 경우

이 작업을 실행 한 후 다음을 수행할 수 있습니다.
  * 을 사용 하 여 조인 된 Windows 노드 보기`kubectl get nodes`
  * 3 개의 powershell 창이 열려 있고, 하나는에 대해, 다른 하나는에 대해 하나를 참조 하세요. `kubelet` `flanneld``kube-proxy`
  * `flanneld` `kubelet` 노드에서 실행 되는, 및에 대 한 호스트 에이전트 프로세스를 참조 하세요. `kube-proxy`

성공 하면 [다음 단계](#next-steps)를 계속 합니다.

## <a name="joining-a-tor-cluster"></a>연결 클러스터 조인 ##
> [!NOTE]
> [이전에](./network-topologies.md#flannel-in-host-gateway-mode)네트워킹 솔루션으로 Flannel을 선택한 경우이 섹션을 건너뛸 수 있습니다.

이렇게 하려면 [업스트림 L3 라우팅 토폴로지에 대해 Kubernetes에서 Windows Server 컨테이너를 설정](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)하는 방법에 대 한 지침을 따라야 합니다. 여기에는 노드에 할당 된 pod CIDR 접두사가 해당 노드 IP에 매핑되도록 업스트림 라우터를 구성 해야 합니다.

새 노드가 "준비"로 표시 되 고 `kubectl get nodes` kubelet + kube가 실행 중 이며 업스트림 사용자 라우터를 구성한 경우 다음 단계를 수행할 준비가 된 것입니다.

## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 Windows 작업자를 Kubernetes 클러스터에 조인 하는 방법을 살펴보았습니다. 이제 5 단계를 수행할 준비가 되었습니다.

> [!div class="nextstepaction"]
> [Linux 작업자 가입](./joining-linux-workers.md)

또는 Linux 작업자를 사용 하지 않는 경우 6 단계로 넘어갈 수 있습니다.

> [!div class="nextstepaction"]
> [Kubernetes 리소스 배포](./deploying-resources.md)
