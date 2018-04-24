---
title: Windows의 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: v1.9 베타 Kubernetes 클러스터에 Windows 노드를 가입합니다.
keywords: kubernetes, 1.9, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6309ca8c0fd50e1b8e926776bef6dfe82bb815f0
ms.sourcegitcommit: ee86ee093b884c79039a8ff417822c6e3517b92d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/03/2018
---
# <a name="kubernetes-on-windows"></a>Windows의 Kubernetes #

Kubernetes 1.9의 최신 버전 및 Windows Server [버전 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking)를 통해 사용자들은 Windows 네트워킹에서 최신 기능을 활용할 수 있습니다.

  - **공유 포드 구획**: 이제 인프라 및 작업자 포드가 네트워크 구획을 공유합니다(Linux 네임스페이스와 유사함).
  - **끝점 최적화**: 구획 공유 덕분에 컨테이너 서비스는 이전처럼 적어도 절반에서 최대한 많은 끝점을 추적해야 합니다.
  - **데이터 경로 최적화**: 가상 필터링 플랫폼 및 호스트 네트워킹 서비스가 개선되어 커널 기반 부하 분산이 가능합니다.


이 페이지는 완전히 새로운 Windows 노드를 기존 Linux 기반 클러스터에 가입하는 가이드를 제공합니다. 처음부터 시작하려면 [이 페이지](./creating-a-linux-master.md)에서 이전에 수행했던 것과 동일한 방식으로 처음부터 마스터를 설정하기 위해 Kubernetes 클러스터를 배포하는 데 사용할 수 있는 여러 리소스를 참조하세요.

> [!TIP] 
> Azure에서 클러스터를 배포하려는 경우 오픈 소스 ACS-Engine 도구를 사용하면 간편합니다. 단계별 [연습](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md)을 볼 수 있습니다.

<a name="definitions"></a> 다음은 이 가이드에서 참조되는 몇 가지 조건에 대한 정의입니다.

  - **외부 네트워크**는 노드가 이를 통해 통신하는 네트워크입니다.
  - <a name="cluster-subnet-def"></a>**클러스터 서브넷**은 라우팅할 수 있는 가상 네트워크입니다. 포드를 사용하기 위해 여기에서 노드에는 보다 작은 서브넷이 할당됩니다.
  - **서비스 서브넷**은 라우팅할 수 없으며 네트워크 토폴로지에 대한 고려 없이 동일하게 서비스에 액세스하기 위해 포드에 의해 사용되는 11.0/16의 완전히 가상 서브넷입니다. 노드에서 실행되는 `kube-proxy`에 의해 라우팅할 수 있는 주소 공간으로 또는 공간에서 변환됩니다.

## <a name="what-you-will-accomplish"></a>수행할 작업 ##

이 가이드에서는 다음을 수행합니다.

> [!div class="checklist"]  
> * [Linux 마스터](#preparing-the-linux-master) 노드 구성  
> * 이에 [Windows 작업자 노드](#preparing-a-windows-node) 가입  
> * [네트워크 토폴로지](#network-topology) 준비  
> * [샘플 Windows 서비스](#running-a-sample-service) 배포  
> * [일반적인 문제 및 실수](./common-problems.md) 검토  

## <a name="preparing-the-linux-master"></a>Linux 마스터 준비 ##

[지침](./creating-a-linux-master.md)을 따랐는지 또는 이미 기존 클러스터가 있는지에 관계 없이 Linux 마스터에서 필요한 것은 Kubernetes의 인증서 구성뿐입니다. 설정에 따라 `/etc/kubernetes/admin.conf`, `~/.kube/config`, 또는 다른 곳에 있을 수 있습니다.

## <a name="preparing-a-windows-node"></a>Windows 노드 준비 ##

> [!NOTE]  
> Windows 섹션의 모든 코드 조각을 _관리자 권한_ PowerShell로 실행해야 합니다.

Kubernetes는 [Docker](https://www.docker.com/)를 컨테이너 조정자로 사용하므로 설치해야 합니다. [공식 Docs 지침](../manage-docker/configure-docker-daemon.md#install-docker), [Docker 지침](https://store.docker.com/editions/enterprise/docker-ee-server-windows)을 따르거나 또는 다음 단계를 시도할 수 있습니다.

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

[이 Microsoft 리포지토리](https://github.com/Microsoft/SDN)에 이 노드를 클러스터에 가입시키는 데 도움이 되는 스크립트 컬렉션이 있습니다. [여기](https://github.com/Microsoft/SDN/archive/master.zip)에서 ZIP 파일을 직접 다운로드할 수 있습니다. `Kubernetes/windows` 폴더만 필요하며 해당 폴더의 내용을 `C:\k\`로 이동해야 합니다.

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

[이전에 식별된](#preparing-the-linux-master) 인증서 파일을 이 새 `C:\k` 디렉터리에 복사합니다.

## <a name="network-topology"></a>네트워크 토폴로지 ##

여러 가지 방법으로 가상 [서브넷 클러스터](#cluster-subnet-def)를 라우팅할 수 있도록 만들 수 있습니다. 다음을 수행할 수 있습니다.

  - [호스트 게이트웨이 모드](./configuring-host-gateway-mode.md)를 구성하여 정적 다음 홉 경로를 설정하여 포드 간 통신을 활성화할 수 있습니다.
  - 스마트 ToR(Top-of-Rack) 스위치를 구성하여 서브넷을 라우팅합니다.
  - [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html)과 같은 타사 오버레이 플러그인을 사용합니다(Flannel에 대한 Windows 지원은 베타 단계임).

### <a name="creating-the-pause-image"></a>"일시 중지" 이미지 만들기 ###

`docker`를 설치했으므로 이제 인프라 포드를 준비하기 위해 Kubernetes에 의해 사용되는 "일시 중지" 이미지를 준비해야 합니다.

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!NOTE]
> 나중에 배포하는 샘플 서비스가 실제로 사용 가능한 최신 Windows Server Core 이미지가 _아니어도_ 해당 이미지에 따라 달라지므로 샘플 서비스를 `:latest` 태그로 지정합니다. 충돌하는 컨테이너 이미지를 조심하는 것이 중요합니다. 예상되는 태그가 없으면 호환되지 않는 컨테이너 이미지의 `docker pull`이 발생하여 [배포 문제](./common-problems.md#when-deploying-docker-containers-keep-restarting)가 발생할 수 있습니다. 


### <a name="downloading-binaries"></a>이진 파일 다운로드 ###
그 동안에 `pull`이 발생하는 동안 Kubernetes에서 다음 클라이언트 측 바이너리를 다운로드합니다.

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

최신 1.9 릴리스의 `CHANGELOG.md` 파일에 있는 링크에서 이를 다운로드할 수 있습니다. 이 문서를 작성한 시점을 기준으로 [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1)이며 Windows 바이너리는 [여기](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz)에 있습니다. [7-Zip](http://www.7-zip.org/)과 같은 도구를 사용하여 아카이브 압축을 풀고 `C:\k\`에 바이너리를 놓습니다.

`kubectl` 명령을 `C:\k\` 디렉터리 외부에서 사용할 수 있도록 하려면 `PATH` 환경 변수를 다음과 같이 수정하십시오.

```powershell
$env:Path += ";C:\k"
```

이 변경 사항을 영구적으로 유지하려면 다음과 같이 대상 컴퓨터에서 변수를 수정하십시오.

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

### <a name="joining-the-cluster"></a>클러스터 가입 ###
다음을 사용하여 클러스터 구성이 올바른지 확인하십시오.

```powershell
kubectl version
```

연결 오류가 나타나는 경우

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

구성이 적절하게 발견되었는지 확인합니다.

```powershell
kubectl config view
```

`kubectl`이 구성 파일을 찾는 위치를 변경하기 위해 `--kubeconfig` 매개 변수를 전달하거나 `KUBECONFIG` 환경 변수를 수정할 수 있습니다. 예를 들어 구성 파일이 `C:\k\config`에 있는 경우 다음과 같습니다.

```powershell
$env:KUBECONFIG="C:\k\config"
```

이 설정을 현재 사용자 범위에서 영구적으로 유지하려면 다음을 수행하십시오.

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

노드가 이제 클러스터에 가입할 준비가 되었습니다. 별개의 *관리자 권한* PowerShell 창에서 이 스크립트를 실행합니다(이 순서대로). 첫 번째 스크립트의 `-ClusterCidr` 매개 변수는 구성된 [클러스터 서브넷](#cluster-subnet-def)이며 여기에서는 `192.168.0.0/16`입니다.

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

1분 안에 Windows 노드가 `kubectl get nodes` 아래의 Linux 마스터에서 표시됩니다.


### <a name="validating-your-network-topology"></a>네트워크 토폴로지 확인 ###

적절한 네트워크 구성을 확인하는 몇 가지 기본 테스트가 있습니다.

  - **노드 간 연결**: 마스터 및 Windows 작업자 노드 간에 ping이 양방향에서 성공해야 합니다.

  - **포드 서브넷 및 노드 간 연결**: 가상 포드 인터페이스와 노드 간에 ping합니다. 각각 Linux 및 Windows의 `route -n` 및 `ipconfig` 아래의 게이트웨이 주소를 찾고 `cbr0` 인터페이스를 찾습니다.

이런 기본 테스트 중 하나라도 작동하지 않는 경우 문제 해결을 위해 [문제 해결 페이지](./common-problems.md#common-networking-errors)를 사용해 볼 수 있습니다.


## <a name="running-a-sample-service"></a>샘플 서비스 실행 ##

클러스터에 성공적으로 가입되고 네트워크가 적절히 구성되었는지 확인하기 위해 매우 간단한 [PowerShell 기반 웹 서비스](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)를 배포할 것입니다.

Linux 마스터에서 서비스를 다운로드하여 실행합니다.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

이렇게 하면 배포 및 서비스가 생성되며 이들의 상태를 추적하기 위해 무기한으로 포드를 감시합니다. 관찰을 완료했으면 `Ctrl+C`를 눌러 `watch` 명령을 종료하면 됩니다.

모든 작업이 제대로 진행되면 다음을 수행할 수 있습니다.

  - Windows 노드의 `docker ps` 명령 아래에서 컨테이너 4개를 확인합니다.
  - Linux 마스터의 `kubectl get pods` 명령 아래에서 포드 2개를 확인합니다.
  - `curl` 포트 80에 있는 *포드* IP에서 Linux 마스터로부터 웹 서버 응답을 가져옵니다. 이는 네트워크에 걸친 적절한 노드에서 포드 통신을 보여줍니다.
  - `docker exec`를 통해 *포드 간* Ping(Windows 노드가 여러 개 있는 경우 호스트 간 포함). 이는 적절한 포드 간 통신을 나타냅니다.
  - `curl` Linux 마스터 및 개별 포드에서 가상 *서비스* IP(`kubectl get services` 아래에 표시).
  - `curl` *서비스 이름*에 Kubernetes [기본 DNS 접미사](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)가 있음. 이는 DNS 기능을 보여줍니다.

> [!Warning]  
> Windows 노드는 서비스 IP에 액세스할 수 없습니다. 이를 [알려진 플랫폼 제한](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)이라고 하며, 다음 Windows Server 업데이트에서 개선될 예정입니다.


### <a name="port-mapping"></a>포트 매핑 ### 
또한 노드의 포트를 매핑하여 각 노드를 통해 포드에 호스팅되는 서비스에 액세스할 수도 있습니다. 이 기능을 보여주기 위해 노드의 포트 4444를 포드의 포트 80에 매핑하는 [다른 샘플 YAML](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml)도 볼 수 있습니다. 배포하려면 전과 동일한 단계를 따르세요.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

이제 포트 4444의 *노드* IP에 대해 `curl`을 수행할 수 있습니다. 이는 일 대 일 매핑을 적용해야 하므로 노드당 단일 포드로의 확장이 제한된다는 점에 유의하세요.