---
title: "Windows의 Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "v1.9 베타 Kubernetes 클러스터에 Windows 노드를 가입합니다."
keywords: "kubernetes, 1.9, windows, 시작"
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: d88ab46dc0046256ebed9c6696a99104a7197fad
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/05/2017
---
# <a name="kubernetes-on-windows"></a>Windows의 Kubernetes #
Kubernetes 1.9의 최신 버전 및 Windows Server [버전 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking)를 통해 사용자들은 Windows 네트워킹에서 최신 기능을 활용할 수 있습니다.

  - **공유 포드 구획**: 이제 인프라 및 작업자 포드가 네트워크 구획을 공유합니다(Linux 네임스페이스와 유사함).
  - **끝점 최적화**: 구획 공유 덕분에 컨테이너 서비스는 이전처럼 적어도 절반에서 최대한 많은 끝점을 추적해야 합니다.
  - **데이터 경로 최적화**: 가상 필터링 플랫폼 및 호스트 네트워킹 서비스가 개선되어 커널 기반 부하 분산이 가능합니다.


이 페이지는 완전히 새로운 Windows 노드를 기존 Linux 기반 클러스터에 가입하는 가이드를 제공합니다. 처음부터 시작하려면 [이 페이지](./creating-a-linux-master.md)에서 이전에 수행했던 것과 동일한 방식으로 처음부터 마스터를 설정하기 위해 Kubernetes 클러스터를 배포하는 데 사용할 수 있는 여러 리소스를 참조하세요.


<a name="definitions"></a> 다음은 이 가이드에서 참조되는 몇 가지 조건에 대한 정의입니다.

  - **외부 네트워크**는 노드가 이를 통해 통신하는 네트워크입니다.
  - <a name="cluster-subnet-def"></a>**클러스터 서브넷**은 라우팅할 수 있는 가상 네트워크입니다. 포드를 사용하기 위해 여기에서 노드에는 보다 작은 서브넷이 할당됩니다.
  - **서비스 서브넷**은 라우팅할 수 없으며 네트워크 토폴로지에 대한 고려 없이 동일하게 서비스에 액세스하기 위해 포드에 의해 사용되는 11.0/16의 완전히 가상 서브넷입니다. 노드에서 실행되는 `kube-proxy`에 의해 라우팅할 수 있는 주소 공간으로 또는 공간에서 변환됩니다.


## <a name="what-we-will-accomplish"></a>수행할 작업 ##
이 가이드에서는 다음을 수행합니다.

> [!div class="checklist"]  
> * [네트워크 토폴로지](#network-topology) 준비  
> * [Linux 마스터](#preparing-the-linux-master) 노드 구성  
> * 이에 [Windows 작업자 노드](#preparing-a-windows-node) 가입  
> * [샘플 Windows 서비스](#running-a-sample-service) 배포  
> * [일반적인 문제 및 실수](./common-problems.md) 검토  


## <a name="network-topology"></a>네트워크 토폴로지 ##
여러 가지 방법으로 가상 [서브넷 클러스터](#cluster-subnet-def)를 라우팅할 수 있도록 만들 수 있습니다. 다음을 수행할 수 있습니다.

  - [호스트 게이트웨이 모드](./configuring-host-gateway-mode.md)를 구성하여 정적 다음 홉 경로를 설정하여 포드 간 통신을 활성화할 수 있습니다.
  - 스마트 ToR(Top-of-Rack) 스위치를 구성하여 서브넷을 라우팅합니다.
  - [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html)과 같은 타사 오버레이 플러그 인을 사용합니다(Flannel에 대한 Windows 지원 베타 단계입니다).


## <a name="preparing-the-linux-master"></a>Linux 마스터 준비 ##
[우리의 지침](./creating-a-linux-master.md)을 따랐든지 또는 이미 기존 클러스터가 있든지 상관 없이 Linux 마스터에서 필요한 것은 Kubernetes의 인증서 구성뿐입니다. 설정에 따라 `/etc/kubernetes/admin.conf`, `~/.kube/config`, 또는 다른 곳에 있을 수 있습니다.


## <a name="preparing-a-windows-node"></a>Windows 노드 준비 ##
> [!Note]  
> Windows 섹션의 모든 코드 조각을 관리자 권한 PowerShell로 실행해야 합니다.

Kubernetes는 [Docker](https://www.docker.com/)를 컨테이너 조정자로 사용하므로 설치해야 합니다. [공식 MSDN 지침](virtualization/windowscontainers/manage-docker/configure-docker-daemon.md#install-docker), [Docker 지침](https://store.docker.com/editions/enterprise/docker-ee-server-windows)을 따르거나 또는 다음 단계를 시도할 수 있습니다.

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

[이 Microsoft 리포지토리](https://github.com/Microsoft/SDN)에 클러스터에 이 노드를 가입하는 데 도움이 되는 스크립트 컬랙션이 있습니다. [여기](https://github.com/Microsoft/SDN/archive/master.zip)에서 ZIP 파일을 직접 다운로드할 수 있습니다. `Kubernetes/windows` 폴더만 필요하며 해당 폴더의 내용을 `C:\k\`로 이동해야 합니다.

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

[이전에 식별된](#preparing-the-linux-master) 인증서 파일을 이 새 `C:\k` 디렉터리에 복사합니다.


### <a name="creating-the-pause-image"></a>"일시 중지" 이미지 만들기 ###
`docker`를 설치했으므로 이제 인프라 포드를 준비하기 위해 Kubernetes에 의해 사용되는 "일시 중지" 이미지를 준비해야 합니다.

```powershell
docker pull microsoft/windowsservercore:1709
docker tag $(docker images -q) microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!Note]  
> 나중에 배포할 샘플 서비스에 쓰기 위해 이를 `:latest`로 태그 지정합니다.


### <a name="downloading-binaries"></a>바이너리 다운로드 ###
그 동안에 `pull`이 발생하는 동안 Kubernetes에서 다음 클라이언트 측 바이너리를 다운로드합니다.

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

최신 1.9 릴리스의 `CHANGELOG.md` 파일에 있는 링크에서 이를 다운로드할 수 있습니다. 이 문서를 작성한 시점을 기준으로 [1.9.0-beta.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.0-beta.1)이며 Windows 바이너리는 [여기](https://dl.k8s.io/v1.9.0-beta.1/kubernetes-node-windows-amd64.tar.gz)에 있습니다. [7-Zip](http://www.7-zip.org/)과 같은 도구를 사용하여 아카이브 압축을 풀고 `C:\k\`에 바이너리를 놓습니다.

> [!Warning]  
> 이 문서 작성 시점을 기준으로 `kube-proxy.exe`가 제대로 작동하려면 보류 중인 Kubernetes [풀 요청](https://github.com/kubernetes/kubernetes/pull/56529)이 필요합니다. 이를 수행하기 위해 [바이너리를 직접 빌드](./compiling-kubernetes-binaries.md)해야 할 수도 있습니다.


### <a name="joining-the-cluster"></a>클러스터 가입 ###
노드가 이제 클러스터에 가입할 준비가 되었습니다. 별개의 *관리자 권한* PowerShell 창에서 이 스크립트를 실행합니다(이 순서대로). 첫 번째 스크립트의 `-ClusterCidr` 매개 변수는 구성된 [클러스터 서브넷](#cluster-subnet-def)이며 여기에서는 `192.168.0.0/16`입니다.

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

1분 안에 Windows 노드가 `kubectl get nodes` 아래의 Linux 마스터에서 표시됩니다.


### <a name="validating-your-network-topology"></a>네트워크 토폴로지 유효성 검사 ###
적절한 네트워크 구성을 검사하는 몇 가지 기본 테스트가 있습니다.

  - **노드 간 연결**: 마스터 및 Windows 작업자 노드 간에 ping이 양방향에서 성공해야 합니다.

  - **포드 서브넷 및 노드 간 연결**: 가상 포드 인터페이스와 노드 간에 ping합니다. 각각 Linux 및 Windows의 `route -n` 및 `ipconfig` 아래의 게이트웨이 주소를 찾고 `cbr0` 인터페이스를 찾습니다.

이런 기본 테스트 중 하나라도 작동하지 않는 경우 문제 해결을 위해 [문제 해결 페이지](./common-problems.md#network-connectivity)를 사용해 볼 수 있습니다.


## <a name="running-a-sample-service"></a>샘플 서비스 실행 ##
클러스터가 성공적으로 가입되고 네트워크가 적절히 구성되었는지 확인하기 위해 매우 간단한 [PowerShell 기반 웹 서비스](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)를 배포해보겠습니다.


Linux 마스터에서 서비스를 다운로드하여 실행합니다.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

이렇게 하면 배포 및 서비스가 생성되며 이들의 상태를 추적하기 위해 무기한으로 포드를 감시합니다. 관찰을 완료했으면 `Ctrl+C`를 눌러 `watch` 명령을 종료하면 됩니다.


모든 것이 순조롭다면 다음이 가능한지 확인하기 위해 유효성 검사를 시행할 수 있습니다.

  - Windows 측 `docker ps` 명령 아래에서 컨테이너 4개를 확인합니다.
  - `curl` 포트 80에 있는 *포드* IP에서 Linux 마스터로부터 웹 서버 응답을 가져옵니다. 이는 네트워크에 걸친 적절한 노드에서 포드 통신을 보여줍니다.
  - `curl` 포트 4444에 있는 *노드* IP에서 웹 서버 응답을 가져옵니다. 이는 적절한 호스트에서 컨테이너 포트 매핑을 보여 줍니다.
  - `docker exec`를 통해 *포드 간* Ping(Windows 노드가 여러 개 있는 경우 호스트 간 포함). 이는 적절한 포드 간 통신을 나타냅니다.
  - `curl` Linux 마스터 및 개별 포드에서 가상 *서비스* IP(`kubectl get services` 아래에 표시).
  - `curl` *서비스 이름*에 Kubernetes [기본 DNS 접미사](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)가 있음. 이는 DNS 기능을 보여줍니다.

> [!Warning]  
> Windows 노드는 서비스 IP에 액세스할 수 없습니다. 이는 [잘 알려진 제한 사항](./common-problems.md#common-windows-errors)입니다.
