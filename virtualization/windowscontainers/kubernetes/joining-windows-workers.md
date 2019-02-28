---
title: Windows 노드를 가입
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: V1.13 사용 하 여 Kubernetes 클러스터에 Windows 노드를 가입합니다.
keywords: kubernetes, 1.13, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: ed0f13bd429e88f05469f91c3fc691bf0188b0a2
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120571"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Windows Server 노드 클러스터에 가입 #
[Kubernetes 마스터 노드를 설정](./creating-a-linux-master.md) 하 고 [원하는 네트워크 솔루션을 선택](./network-topologies.md)했다면 가입 Windows 서버 노드 클러스터를 만들 준비가 되었습니다. 이 [Windows 노드 준비](#preparing-a-windows-node) 에 조인 하기 전에 필요합니다.

## <a name="preparing-a-windows-node"></a>Windows 노드 준비 ##
> [!NOTE]  
> Windows 섹션의 모든 코드 조각을 _관리자 권한_ PowerShell로 실행해야 합니다.

### <a name="install-docker-requires-reboot"></a>(다시 부팅 해야 함)는 Docker 설치 ###
Kubernetes [Docker](https://www.docker.com/) 컨테이너 엔진을 사용 하므로 설치 해야 합니다. [공식 Docs 지침](../manage-docker/configure-docker-daemon.md#install-docker), [Docker 지침](https://store.docker.com/editions/enterprise/docker-ee-server-windows)을 따르거나 또는 다음 단계를 시도할 수 있습니다.

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

프록시 뒤에 있는 경우 다음 PowerShell 환경 변수를 정의해야 합니다.
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

다시 부팅 한 후 다음과 같은 오류가 표시 합니다.

![텍스트](media/docker-svc-error.png)

그런 다음 docker 서비스를 수동으로 시작.

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>"일시 중지" (인프라) 이미지 만들기 ###
> [!Important]
> 것에 충돌 하는 컨테이너 이미지입니다. 필요한 태그를가지고 있지 않은 경우 발생할 수는 `docker pull` 호환 되지 않는 컨테이너 이미지의 문제가 [배포와](./common-problems.md#when-deploying-docker-containers-keep-restarting) 같은 정해 지지 않은 `ContainerCreating` 상태.

`docker`를 설치했으므로 이제 인프라 포드를 준비하기 위해 Kubernetes에 의해 사용되는 "일시 중지" 이미지를 준비해야 합니다. 이 다음과 같은 세 가지 단계가 있습니다. 
  1. [이미지 가져오기](#pull-the-image)
  2. microsoft로 [것 태그](#tag-the-image) / nanoserver:latest
  3. [실행](#run-the-container) 및


#### <a name="pull-the-image"></a>이미지를 가져올 ####     
 사용 중인 특정 Windows 버전에 대 한 이미지를 가져옵니다. 예를 들어, Windows Server 2019를 실행 하는 경우:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>태그의 이미지 ####
이 가이드의 뒷부분에 사용할 Dockerfile에 대 한 확인 합니다 `:latest` 태그를 이미지. 사용자만 값을 가져오는 같이 nanoserver 이미지 태그:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>컨테이너를 실행 합니다. ####
컨테이너가 컴퓨터에서 실제로 실행 되는지 확인 합니다.

```powershell
docker run microsoft/nanoserver:latest
```

다음과 같이 표시 됩니다.

![텍스트](./media/docker-run-sample.png)

> [!tip]
> 실행할 수 없으면 컨테이너 하십시오 참고: [컨테이너 이미지와 일치 하는 컨테이너 호스트 버전](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Kubernetes에 대 한 Windows 디렉터리 준비 ####
모든도 배포 스크립트 및 config 파일 Kubernetes 바이너리를 저장 하는 "Windows 용 Kubernetes" 디렉터리를 만듭니다.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Kubernetes 인증서를 복사 합니다. #### 
Kubernetes 인증서 파일을 복사 (`$HOME/.kube/config`) [마스터를](./creating-a-linux-master.md#collect-cluster-information) `C:\k` 디렉터리입니다.

> [!tip]
> 노드 간에 config 파일을 전송 [xcopy](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/xcopy) 또는 [WinSCP](https://winscp.net/eng/download.php) 같은 도구를 사용할 수 있습니다.

#### <a name="download-kubernetes-binaries"></a>Kubernetes 바이너리를 다운로드 합니다. ####
Kubernetes를 실행할 수 있도록 먼저 다운로드 해야 합니다 `kubectl`, `kubelet`, 및 `kube-proxy` 이진 파일입니다. 에 있는 링크에서이 다운로드할 수는 `CHANGELOG.md` 파일의 [최신 릴리스](https://github.com/kubernetes/kubernetes/releases/)합니다.
 - 예를 들어 다음은 [v1.13 노드 이진 파일](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#node-binaries)입니다.
 - [확장 보관](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) 와 같은 도구를 사용 하 여 아카이브 압축을 풀고 바이너리에 `C:\k\`.

#### <a name="optional-setup-kubectl-on-windows"></a>(선택 사항) Windows 설치 프로그램 kubectl ####
Windows에서 클러스터를 제어 하려는 경우를 할 수 있는 사용 하 여는 `kubectl` 명령입니다. 첫 번째 되도록 `kubectl` 외부에서 사용할 수는 `C:\k\` 디렉터리를 수정 합니다 `PATH` 환경 변수:

```powershell
$env:Path += ";C:\k"
```

이 변경 사항을 영구적으로 유지하려면 다음과 같이 대상 컴퓨터에서 변수를 수정하십시오.

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

다음으로, [클러스터 인증서](#copy-kubernetes-certificate) 가 유효한 지 확인 합니다. 위치를 설정 하기 위해 여기서 `kubectl` 전달할 수 있는 구성 파일을 찾는 합니다 `--kubeconfig` 매개 변수 또는 수정 합니다 `KUBECONFIG` 환경 변수. 예를 들어 구성 파일이 `C:\k\config`에 있는 경우 다음과 같습니다.

```powershell
$env:KUBECONFIG="C:\k\config"
```

이 설정을 현재 사용자 범위에서 영구적으로 유지하려면 다음을 수행하십시오.

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

마지막으로, 구성이 적절 하 게 발견 된 경우를 확인 하려면 사용할 수 있습니다.

```powershell
kubectl config view
```

연결 오류가 나타나는 경우

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

다시 복사 하려고 하거나 kubeconfig 위치를 다시 확인 해야 합니다.

오류 없이 표시 노드 클러스터에 가입할 준비가 되었습니다.

## <a name="joining-the-windows-node"></a>Windows 노드를 가입 ##
[사용자가 선택한 네트워킹 솔루션](./network-topologies.md)따라 다음 작업을 수행할 수 있습니다.
1. [Windows Server 노드 Flannel (vxlan 또는 호스트 gw) 클러스터에 가입](#joining-a-flannel-cluster)
2. [Windows Server 노드에 ToR 스위치를 사용 하 여 클러스터에 가입](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Flannel 클러스터에 가입 ###
클러스터에이 노드를 가입 하는 데 도움이 되는 [Microsoft 리포지토리](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) 에서 Flannel 배포 스크립트 컬렉션이 있습니다.

콘텐츠를 압축 [Flannel start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) 스크립트를 다운로드 `C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

[Windows 노드 준비](#preparing-a-windows-node)를 가정 하 고 `c:\k` 디렉터리 아래와 같이 표시, 노드를 가입할 수 있습니다.

![텍스트](./media/flannel-directory.png)

#### <a name="join-node"></a>노드를 가입 #### 
Windows 노드를 가입 하는 프로세스를 간소화 하려면 하기만 하면 시작 하는 단일 Windows 스크립트를 실행 하려면 `kubelet`, `kube-proxy`, `flanneld`, 노드를 가입 하 고 있습니다.

> [!Note]
> [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) [install.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1)가 같은 추가 파일 다운로드 참조는 `flanneld` 실행 파일 및 [인프라 포드에 대 한 Dockerfile](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *및 설치 수에 대 한*합니다. 오버레이 네트워킹 모드에 대 한 로컬 UDP 포트 4789에 대 한 [방화벽](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) 열립니다. 여러 powershell 창 포드 네트워크에 대 한 새 외부 vSwitch를 처음 만드는 동안 몇 초 동안 네트워크 중단의 뿐만 아니라 열/종료 되 고 있을 수 있습니다.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
Windows 노드에 할당 된 IP 주소입니다. 사용할 수 있습니다 `ipconfig` 검색 합니다.

|  |  | 
|---------|---------|
|매개 변수     | `-ManagementIP`        |
|기본값    | n.A. **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
네트워크 모드 `l2bridge` (flannel 호스트 gw) 또는 `overlay` (flannel vxlan) [네트워크 솔루션](./network-topologies.md)으로 선택 합니다.

> [!Important] 
> `overlay` 네트워킹 모드 (flannel vxlan) Kubernetes v1.14 바이너리 필요 이상.

|  |  | 
|---------|---------|
|매개 변수     | `-NetworkMode`        |
|기본값    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[클러스터 서브넷 범위](./getting-started-kubernetes-windows.md#cluster-subnet-def)입니다.

|  |  | 
|---------|---------|
|매개 변수     | `-ClusterCIDR`        |
|기본값    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
[서비스 서브넷 범위](./getting-started-kubernetes-windows.md#service-subnet-def)입니다.

|  |  | 
|---------|---------|
|매개 변수     | `-ServiceCIDR`        |
|기본값    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Kubernetes DNS 서비스 IP](./getting-started-kubernetes-windows.md#kube-dns-def)합니다.

|  |  | 
|---------|---------|
|매개 변수     | `-KubeDnsServiceIP`        |
|기본값    | `10.96.0.10`        |


# [<a name="interfacename"></a>InterfaceName](#tab/InterfaceName)
Windows 호스트의 네트워크 인터페이스의 이름입니다. 사용할 수 있습니다 `ipconfig` 검색 합니다.

|  |  | 
|---------|---------|
|매개 변수     | `-InterfaceName`        |
|기본값    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Kubelet 및 kube 프록시 로그는 해당 출력 파일로 리디렉션됩니다 있는 디렉터리입니다.

|  |  | 
|---------|---------|
|매개 변수     | `-LogDir`        |
|기본값    | `C:\k`        |


---

> [!tip]
> 이미 기록한 클러스터 서브넷, 서비스 서브넷 및 Linux 마스터에서 kube DNS IP [이전](./creating-a-linux-master.md#collect-cluster-information)

이 실행 한 후에 있어야 합니다.
  * 보기를 사용 하 여 Windows 노드를 가입 `kubectl get nodes`
  * 3 powershell 창 열기에 대 한 참조 `kubelet`, 하나의 대 한 `flanneld`,이 고 다른 `kube-proxy`
  * 호스트 에이전트 프로세스에 대 한 참조 `flanneld`, `kubelet`, 및 `kube-proxy` 노드에서 실행

성공 하면 [다음 단계](#next-steps)를 계속 합니다.

## <a name="joining-a-tor-cluster"></a>ToR 클러스터에 가입 ##
> [!NOTE]
> Flannel에 네트워킹 솔루션 [이전에](./network-topologies.md#flannel-in-host-gateway-mode)으로 선택한 경우이 섹션을 건너뛸 수 있습니다.

이렇게 하려면 [Windows Server 컨테이너 업스트림 L3 라우팅 토폴로지에 대 한 kubernetes 설정](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)에 대 한 지침에 따라 해야 합니다. 각 노드 IP에 매핑됩니다 같은 포드 CIDR 접두사는 노드에 할당 업스트림 라우터를 구성 했는지 포함 됩니다.

로 "준비"으로 나열 된 새 노드의 가정 하 고 `kubectl get nodes`kubelet + kube 프록시를 실행 하 고 업스트림 ToR 라우터를 구성한 경우, 다음 단계에 대 한 준비가 완료 됩니다.

## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 Windows 작업자 우리의 Kubernetes 클러스터에 가입 하는 방법을 설명 했습니다. 이제 5 단계에 대 한 준비가:

> [!div class="nextstepaction"]
> [Linux 작업자에 가입](./joining-linux-workers.md)

또는 없는 경우 모든 Linux 작업자 자유롭게 6 단계로 건너 뛰:

> [!div class="nextstepaction"]
> [Kubernetes 리소스 배포](./deploying-resources.md)
