---
title: Windows 노드 참가
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: V 1.14를 사용 하 여 Windows 노드를 Kubernetes 클러스터에 가입 합니다.
keywords: kubernetes, 1.14, windows, 시작 하기
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882986"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>클러스터에 Windows Server 노드 가입 #
[Kubernetes 마스터 노드를 설정](./creating-a-linux-master.md) 하 고 [원하는 네트워크 솔루션을 선택한](./network-topologies.md)후에는 Windows Server 노드에 참가 하 여 클러스터를 형성할 수 있습니다. 이를 위해서는 참가 하기 전에 [Windows 노드에 대 한 준비가](#preparing-a-windows-node) 필요 합니다.

## <a name="preparing-a-windows-node"></a>Windows 노드 준비 ##
> [!NOTE]  
> Windows 섹션의 모든 코드 조각을 _관리자 권한_ PowerShell로 실행해야 합니다.

### <a name="install-docker-requires-reboot"></a>Docker 설치 (재부팅 필요) ###
Kubernetes는 해당 컨테이너 엔진으로 [Docker](https://www.docker.com/) 를 사용 하므로 설치 해야 합니다. [공식 Docs 지침](../manage-docker/configure-docker-daemon.md#install-docker), [Docker 지침](https://store.docker.com/editions/enterprise/docker-ee-server-windows)을 따르거나 또는 다음 단계를 시도할 수 있습니다.

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

다시 부팅 한 후 다음 오류가 표시 됩니다.

![텍스트](media/docker-svc-error.png)

그런 다음 수동으로 docker 서비스를 시작 합니다.

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>"Pause" (인프라) 이미지 만들기 ###
> [!Important]
> 컨테이너 이미지가 충돌 하는 것을 주의 해야 합니다. 예상 되는 태그가 없으면 호환 되지 않는 `docker pull` 컨테이너 이미지를 생성 하 여 무한 `ContainerCreating` 상태 등의 [배포 문제](./common-problems.md#when-deploying-docker-containers-keep-restarting) 를 일으킬 수 있습니다.

`docker`를 설치했으므로 이제 인프라 포드를 준비하기 위해 Kubernetes에 의해 사용되는 "일시 중지" 이미지를 준비해야 합니다. 다음 세 단계를 수행 합니다. 
  1. [이미지 가져오기](#pull-the-image)
  2. microsoft/nanoserver: 최신 버전으로 [태그](#tag-the-image) 지정
  3. 및 [실행](#run-the-container)


#### <a name="pull-the-image"></a>이미지 가져오기 ####     
 특정 Windows 릴리스에 맞게 이미지를 당깁니다. 예를 들어, Windows Server 2019을 실행 하는 경우:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>이미지에 태그 지정 ####
이 가이드의 뒷부분에서 사용할 Dockerfiles는 `:latest` image 태그를 찾습니다. 다음과 같이 방금 끌어온 nanoserver 이미지에 태그를 지정 합니다.

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>컨테이너를 실행 합니다. ####
컨테이너가 컴퓨터에서 실제로 실행 되는지 다시 한 번 확인 합니다.

```powershell
docker run microsoft/nanoserver:latest
```

다음과 같은 항목이 표시 됩니다.

![텍스트](./media/docker-run-sample.png)

> [!tip]
> 컨테이너를 실행할 수 없는 경우 다음을 참조 하세요. 컨테이너 [이미지와 일치 하는 컨테이너 호스트 버전](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Windows 용 Kubernetes 준비 디렉터리 ####
"Windows 용 Kubernetes" 디렉터리를 만들어 Kubernetes 바이너리 및 배포 스크립트와 구성 파일을 저장 합니다.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Kubernetes 인증서 복사 #### 
Kubernetes 인증서 파일 (`$HOME/.kube/config`)을 [마스터에서](./creating-a-linux-master.md#collect-cluster-information) 이 새로운 `C:\k` 디렉터리에 복사 합니다.

> [!tip]
> [Xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) 또는 [winscp](https://winscp.net/eng/download.php) 등의 도구를 사용 하 여 노드 간에 구성 파일을 전송할 수 있습니다.

#### <a name="download-kubernetes-binaries"></a>Kubernetes 바이너리 다운로드 ####
Kubernetes를 실행 하려면 먼저 `kubectl`, `kubelet`및 `kube-proxy` 이진 파일을 다운로드 해야 합니다. `CHANGELOG.md` [최신 릴리스](https://github.com/kubernetes/kubernetes/releases/)파일의 링크에서이를 다운로드할 수 있습니다.
 - 예를 들어 여기에는 [v 1.14 노드 바이너리가](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries)있습니다.
 - [확장 저장소](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) 와 같은 도구를 사용 하 여 보관 파일을 추출 하 고 이진 `C:\k\`파일을에 배치 합니다.

#### <a name="optional-setup-kubectl-on-windows"></a>) Windows 설치 kubectl ####
Windows에서 클러스터를 제어 하기 위해 `kubectl` 명령을 사용할 수 있습니다. 먼저, `kubectl` `C:\k\` 디렉터리 외부에서 사용 가능 하 게 하려면 환경 변수 `PATH` 를 수정 합니다.

```powershell
$env:Path += ";C:\k"
```

이 변경 사항을 영구적으로 유지하려면 다음과 같이 대상 컴퓨터에서 변수를 수정하십시오.

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

그 다음에는 [클러스터 인증서](#copy-kubernetes-certificate) 가 유효한 지 확인 합니다. 구성 파일을 `kubectl` 찾을 위치를 설정 하려면 매개 변수를 `--kubeconfig` 전달 하거나 `KUBECONFIG` 환경 변수를 수정 하면 됩니다. 예를 들어 구성 파일이 `C:\k\config`에 있는 경우 다음과 같습니다.

```powershell
$env:KUBECONFIG="C:\k\config"
```

이 설정을 현재 사용자 범위에서 영구적으로 유지하려면 다음을 수행하십시오.

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

마지막으로, 구성이 올바르게 검색 되었는지 확인 하려면 다음을 사용할 수 있습니다.

```powershell
kubectl config view
```

연결 오류가 나타나는 경우

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Kubeconfig 위치를 다시 한 번 확인 하거나이를 복사 하십시오.

오류가 표시 되지 않으면 이제 노드가 클러스터에 가입할 준비가 된 것입니다.

## <a name="joining-the-windows-node"></a>Windows 노드 참가 ##
[선택한 네트워킹 솔루션](./network-topologies.md)에 따라 다음을 수행할 수 있습니다.
1. [Flannel (vxlan 또는 host-gw) 클러스터에 Windows Server 노드 참가](#joining-a-flannel-cluster)
2. [연결 스위치를 사용 하 여 Windows Server 노드를 클러스터에 가입](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Flannel 클러스터 참가 ###
이 노드를 클러스터에 연결 하는 데 도움이 되는 [이 Microsoft 리포지토리에](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) 대 한 Flannel 배포 스크립트 컬렉션이 있습니다.

[Flannel 시작. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) 스크립트를 다운로드 하 고 다음에 `C:\k`대 한 콘텐츠를 추출 해야 합니다.

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

[Windows 노드를 준비한](#preparing-a-windows-node)후 다음과 같이 `c:\k` 디렉터리가 표시 되는 경우에는 노드에 가입할 준비가 된 것입니다.

![텍스트](./media/flannel-directory.png)

#### <a name="join-node"></a>조인 노드 #### 
Windows 노드를 조인 하는 프로세스를 간소화 하려면 단일 windows 스크립트만 실행 하 여 노드를 시작 `kubelet`하 고 `kube-proxy`,, `flanneld`그리고, 그에 참가 하기만 하면 됩니다.

> [!Note]
> [시작. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) 은 `flanneld` 실행 파일과 [인프라 포드에 대 한 dockerfile](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) 등의 추가 파일을 다운로드 *하 고 해당 사용자를 위해 설치*하는 [설치 ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1)을 참조 합니다. 네트워크 오버레이 모드의 경우 로컬 UDP 포트 4789에 대해 [방화벽이](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) 열립니다. Pod 네트워크에 대 한 새 외부 vSwitch를 처음으로 만드는 동안 여러 powershell 창이 열리거나 닫히는 동시에 몇 초 정도 네트워크 중단 될 수 있습니다.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
Windows 노드에 할당 된 IP 주소입니다. 를 사용 `ipconfig` 하 여이를 찾을 수 있습니다.

|  |  | 
|---------|---------|
|매개 변수     | `-ManagementIP`        |
|기본값    | n.A. **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
네트워크 솔루션으로 `l2bridge` 선택 된 네트워크 모드 (flannel host `overlay` -gw) 또는 (flannel vxlan [](./network-topologies.md))입니다.

> [!Important] 
> `overlay` 네트워킹 모드 (flannel vxlan)에는 Kubernetes v 1.14 이진 (이상) 및 [KB4489899](https://support.microsoft.com/help/4489899)이 필요 합니다.

|  |  | 
|---------|---------|
|매개 변수     | `-NetworkMode`        |
|기본값    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[클러스터 서브넷 범위](./getting-started-kubernetes-windows.md#cluster-subnet-def)

|  |  | 
|---------|---------|
|매개 변수     | `-ClusterCIDR`        |
|기본값    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
[서비스 서브넷 범위](./getting-started-kubernetes-windows.md#service-subnet-def)

|  |  | 
|---------|---------|
|매개 변수     | `-ServiceCIDR`        |
|기본값    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS 서비스 IP](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  | 
|---------|---------|
|매개 변수     | `-KubeDnsServiceIP`        |
|기본값    | `10.96.0.10`        |


# [<a name="interfacename"></a>InterfaceName](#tab/InterfaceName)
Windows 호스트의 네트워크 인터페이스 이름입니다. 를 사용 `ipconfig` 하 여이를 찾을 수 있습니다.

|  |  | 
|---------|---------|
|매개 변수     | `-InterfaceName`        |
|기본값    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Kubelet 및 kube-프록시 로그가 해당 출력 파일로 리디렉션되는 디렉터리입니다.

|  |  | 
|---------|---------|
|매개 변수     | `-LogDir`        |
|기본값    | `C:\k`        |


---

> [!tip]
> [앞서](./creating-a-linux-master.md#collect-cluster-information) Linux 마스터에서 클러스터 서브넷, 서비스 서브넷 및 KUBE-DNS IP를 이미 적어 둔 경우

이 작업을 실행 한 후에는 다음을 수행할 수 있습니다.
  * 을 사용 하 여 연결 된 Windows 노드 보기 `kubectl get nodes`
  * 3 개의 powershell 창을 열고, 하나씩 `kubelet`, 각각에 대해 `flanneld`하나씩 참조 하 고, `kube-proxy`
  * 노드에서 호스트 에이전트 프로세스 `flanneld`, `kubelet`및 `kube-proxy` 실행을 참조 하세요.

성공할 경우 [다음 단계](#next-steps)를 계속 합니다.

## <a name="joining-a-tor-cluster"></a>참여 클러스터 참가 ##
> [!NOTE]
> [이전에](./network-topologies.md#flannel-in-host-gateway-mode)네트워킹 솔루션으로 Flannel를 선택한 경우이 섹션을 건너뛸 수 있습니다.

이렇게 하려면 [Kubernetes에서 업스트림 L3 라우팅 토폴로지에 대해 Windows Server 컨테이너를 설정](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)하기 위한 지침을 따라야 합니다. 여기에는 노드에 할당 된 pod CIDR 접두사가 해당 노드 IP에 매핑되도록 업스트림 라우터를 구성 해야 합니다.

새 노드가 "준비"로 표시 되는 경우 `kubectl get nodes`, kubelet + kube는 프록시가 실행 중이 고 업스트림 사용자 라우터를 구성 하면 다음 단계에 대 한 준비가 된 것입니다.

## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 Windows 작업자를 Kubernetes 클러스터에 가입 하는 방법을 설명 했습니다. 이제 5 단계를 수행할 준비가 되었습니다.

> [!div class="nextstepaction"]
> [Linux 직원 가입](./joining-linux-workers.md)

또는 Linux를 사용 하지 않는 경우 6 단계까지 건너뛸 수 있습니다.

> [!div class="nextstepaction"]
> [Kubernetes 리소스 배포](./deploying-resources.md)
