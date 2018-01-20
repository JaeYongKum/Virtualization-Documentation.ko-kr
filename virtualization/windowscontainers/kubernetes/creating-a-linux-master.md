---
title: "처음부터 Kubernetes 마스터"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "처음부터 Kubernetes 클러스터 마스터를 만듭니다."
keywords: "kubernetes, 1.9, 마스터, linux"
ms.openlocfilehash: d5251b1a2dc06bef396820e324fb240eed04acc8
ms.sourcegitcommit: b0e21468f880a902df63ea6bc589dfcff1530d6e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/17/2018
---
# <a name="kubernetes-master--from-scratch"></a>처음부터 Kubernetes 마스터 #
이 페이지는 처음부터 끝까지 Kubernetes 마스터를 수동 배포하는 방법을 안내합니다.

최근에 업데이트된 Ubuntu 같은 Linux 컴퓨터는 다음 절차를 따라야 합니다. Windows는 전혀 상관이 없습니다. 바이너리는 Linux에서 크로스 컴파일됩니다.


> [!Warning]  
> Kubernetes는 버전 간에 많이 다르기 때문에 이 가이드에서는 이후에 해당하지 않는 내용을 포함할 수 있습니다.


## <a name="preparing-the-master"></a>마스터 준비 ##
먼저 모든 필수 구성 요소를 설치합니다.

```bash
sudo apt-get install curl git build-essential docker.io conntrack
```


[이 리포지토리](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux)에 설치 과정에 도움이 되는 스크립트 컬렉션이 있습니다. `~/kube/`에 시험해 보세요. 이 전체 디렉터리는 이후 단계에서 여러 Docker 컨테이너에 대해 마운트되므로 이 가이드에서 사용된 구조를 동일하게 유지하세요.

```bash
mkdir ~/kube
mkdir ~/kube/bin
git clone https://github.com/Microsoft/SDN /tmp/k8s 
cd /tmp/k8s/Kubernetes/linux
chmod -R +x *.sh
chmod +x manifest/generate.py
mv * ~/kube/
```


### <a name="installing-the-linux-binaries"></a>Linux 바이너리 설치 ###

> [!Note]  
> 패치를 포함하거나 사전 설치된 바이너리를 다운로드하는 대신 최신 Kubernetes 코드를 사용하려면 [이 페이지](./compiling-kubernetes-binaries.md)를 참조하세요.

[Kubernetes 메인 라인](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1)에서 공식 Linux 바이너리를 다운로드하고 다음과 같이 설치합니다.

```bash
wget -O kubernetes.tar.gz https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
tar -vxzf kubernetes.tar.gz 
cd kubernetes/cluster 
# follow the prompts from this command, the defaults are generally fine:
./get-kube-binaries.sh
cd ../server
tar -vxzf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin
cp hyperkube kubectl ~/kube/bin/
```

바이너리를 어디에서든 실행할 수 있도록 `$PATH`에 추가합니다. 이렇게 하면 세션에 대한 경로만 설정됩니다. 영구적으로 설정하려면 이 라인을 `~/.profile`에 추가합니다.

```bash
$ PATH="$HOME/kube/bin:$PATH"
```

### <a name="install-cni-plugins"></a>CNI 플러그 인 설치 ###
Kubernetes 네트워킹에 기본 CNI 플러그 인이 필요합니다. [여기](https://github.com/containernetworking/plugins/releases)에서 다운로드할 수 있으며 `/opt/cni/bin/`으로 압축을 풀어야 합니다.

```bash
DOWNLOAD_DIR="${HOME}/kube/cni-plugins"
CNI_BIN="/opt/cni/bin/"
mkdir ${DOWNLOAD_DIR}
cd $DOWNLOAD_DIR
curl -L $(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep browser_download_url | grep 'amd64.*tgz' | head -n 1 | cut -d '"' -f 4) -o cni-plugins-amd64.tgz
tar -xvzf cni-plugins-amd64.tgz
sudo mkdir -p ${CNI_BIN}
sudo cp -r !(*.tgz) ${CNI_BIN}
ls ${CNI_BIN}
```


### <a name="certificates"></a>인증서 ###
먼저 `ifconfig` 또는 다음을 통해 로컬 IP 주소를 얻습니다.

```bash
$ ip addr show dev eth0
```

(인터페이스 이름이 알려진 경우) 이 주소는 이 프로세스 전체에서 많이 참조되므로 환경 변수로 설정하면 더욱 유용합니다. 다음 조각은 주소를 일시적으로 설정합니다. 세션이 끝났거나 종료된 경우 다시 설정해야 합니다.

```bash
$ MASTER_IP=10.123.45.67   # example! replace
```

클러스터에서 통신하기 위해 노드에 사용할 수 있는 인증서를 준비합니다.

```bash
cd ~/kube/certs
./generate-certs.sh $MASTER_IP
```

### <a name="prepare-manifests--addons"></a>매니페스트 및 추가 기능 준비 ###
마스터 IP 주소 및 *전체* 클러스터 CIDR를 `manifest` 폴더에 있는 Python 스크립트에 전달하여 Kubernetes 시스템 포드를 지정하는 YAML 파일 집합을 생성합니다.

```bash
cd ~/kube/manifest
./generate.py $MASTER_IP --cluster-cidr 192.168.0.0/16
```

Kubernetes가 이를 매니페스트로 혼동하지 않도록 Python 스크립트를 이동(제거)합니다. 이렇게 하지 않으면 나중에 문제가 발생할 수 있습니다.

> [!Important]  
> Kubernetes 버전이 이 가이드에서와 다른 경우 `--api-version`과 같은 스크립트에서 여러 버전 지정 플래그를 사용하여 포드가 배포하는 [이미지를 사용자 지정](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/hyperkube-amd64)합니다. 모든 매니페스트가 동일한 이미지를 사용하고 여러 버전 스키마를 가지는 것은 아닙니다(특히 `etcd` 및 추가 기능 관리자).


#### <a name="manifest-customization"></a>매니페스트 사용자 지정 ####
이 시점에서 특정 설정 관련 변경이 필요할 수 있습니다. 예를 들어 자동으로 서브넷을 Kubernetes에서 관리하도록 하는 것보다 수동으로 서브넷을 노드에 지정해야 할 수 있습니다. 스크립트에서 이 특정 구성에는 옵션이 있습니다(`--help` 매개 변수에 대한 설명은 `--im-sure` 참조).

```bash
./generate.py $MASTER_IP --im-sure
```

다른 모든 사용자 지정 구성 옵션에서는 생성된 매니페스트를 수동으로 수정해야 합니다.


### <a name="configure--run-kubernetes"></a>Kubernetes 구성 및 실행 ###
생성된 인증서를 사용하기 위해 Kubernetes를 구성합니다. 이렇게 하면 `~/.kube/config`에서 구성이 만들어집니다.

```bash
./configure-kubectl.sh $MASTER_IP
```

이제 나중에 포드가 있기를 원하는 위치에 파일을 복사합니다.

```bash
mkdir ~/kube/kubelet
sudo cp ~/.kube/config ~/kube/kubelet/
```

Kubernetes "클라이언트" `kubelet`을 시작할 준비가 됩니다. 다음 스크립트는 모두 무기한 실행됩니다. 작업을 계속하려면 한 터미널 세션 후에 다른 터미널 세션을 엽니다.

```bash
cd ~/kube
sudo ./start-kubelet.sh
```

Kubeproxy 스크립트를 실행하여 부분 클러스터 CIDR을 전달합니다.

```bash
cd ~/kube
sudo ./start-kubeproxy.sh 192.168
```


> [!Important]  
> 이는 기대되는 *전체* /16 CIDR이 될 것이며 *해당 CIDR에 비Kubernetes 트래픽이 있더라도* 이 아래에 노드가 위치할 것입니다. Kubeproxy는 *서비스* 서브넷에 대한 Kubernetes 트래픽에*만* 적용되므로 다른 호스트 트래픽을 방해하지 않아야 합니다.

> [!Note]  
> 이러한 스크립트는 데몬화할 수 있습니다. 이 가이드는 오류를 탐지하기 위한 설정 중에 가장 중요한 스크립트 수동 실행에 대해서만 다룹니다.


## <a name="verifying-the-master"></a>마스터 확인 ##
몇 분 후 시스템은 다음과 같은 상태여야 합니다.

  - `docker ps` 아래에 ~23 작업자 및 포드 컨테이너가 있을 것입니다.
  - `kubectl cluster-info`를 호출하면 DNS 및 Heapster 추가 기능뿐만 아니라 Kubernetes 마스터 API 서버에 대한 정보가 표시됩니다.
  - `ifconfig` 선택된 클러스터 CIDR과 새 인터페이스 `cbr0`이 표시됩니다.

