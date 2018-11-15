---
title: Linux 노드를 가입
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes 클러스터 v1.12로 Linux 노드를 가입합니다.
keywords: kubernetes, 1.12, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 97c12d70db9679dbb85877f0985c6053f95fa500
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/15/2018
ms.locfileid: "6947982"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Linux 노드 클러스터에 가입

[Kubernetes 마스터 노드를 설정](creating-a-linux-master.md) 하 고 [원하는 네트워크 솔루션을 선택](network-topologies.md)했다면 Linux 노드 클러스터에 가입할 준비가 되었습니다. 이렇게 하려면 가입 하기 전에 일부 [Linux 노드 준비](joining-linux-workers.md#preparing-a-linux-node) 합니다.
> [!tip]
> Linux 지침 **Ubuntu 16.04**향해 맞게 조정 됩니다. Kubernetes를 실행 하려면 인증 다른 Linux 배포판 대체할 수 있는 해당 하는 명령을 제공 해야 합니다. 또한 성공적으로 상호 작용 Windows 합니다.

## <a name="preparing-a-linux-node"></a>Linux 노드 준비

> [!NOTE]
> 명시적으로 지정 하지 않으면 **권한 루트 사용자 셸**는에서 모든 명령을 실행 합니다.

먼저, 루트 셸에 가져옵니다.

```bash
sudo –s
```

컴퓨터가 최신 있는지 확인 합니다.

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Docker 설치

컨테이너를 사용할 수 있도록 Docker와 같은 컨테이너 엔진을 해야 합니다. 최신 버전을 가져오려면 Docker 설치에 대 한 [다음이 지침](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 을 사용할 수 있습니다. 해당 docker이 올바르게 실행 하 여 설치를 확인할 수 있습니다 `hello-world` 이미지:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Kubeadm 설치

다운로드 `kubeadm` 바이너리 Linux 배포판 및 클러스터를 초기화 합니다.

> [!Important]  
> Linux 배포판에 따라 대체 해야 할 수도 있습니다 `kubernetes-xenial` 아래 올바른 [코드](https://wiki.ubuntu.com/Releases)를 사용 하 여 합니다.

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>스왑 사용 안 함

Linux에서 Kubernetes 설정을 해제 스왑 공간에 필요 합니다.

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Flannel에만 해당) Iptables IPv4 트래픽을 브리지

Flannel 사용 하도록 설정 하는 것이 좋습니다. 네트워킹 솔루션으로 선택한 경우 iptables 체인에 대 한 IPv4 트래픽을 브리지입니다. [마스터 아직](network-topologies.md#flannel-in-host-gateway-mode) 있어야 하 고 참가 예정인 Linux 노드에 대 한 반복 해야 해야 합니다. 수행할 수 있습니다 다음 명령을 사용 합니다.

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Kubernetes 인증서를 복사 합니다.

**보통 (비 루트) 사용자로**, 다음 3 단계를 수행 합니다.

1. Linux 디렉터리에 대 한 Kubernetes를 만듭니다.

```bash
mkdir -p $HOME/.kube
```

1. Kubernetes 인증서 파일을 복사 (`$HOME/.kube/config`) [마스터에서](./creating-a-linux-master.md#collect-cluster-information) 다른 이름으로 저장 하 고 `$HOME/.kube/config` 작업자에 있습니다.

1. 다음과 같이 복사한 구성 파일의 파일 소유권을 설정 합니다.

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>노드 가입

마지막으로, 실행 클러스터에 가입 하는 `kubeadm join` **루트로** [아래로 앞서 설명한](./creating-a-linux-master.md#initialize-master) 명령:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

성공 하면이 유사한 출력이 표시 되어야 합니다.

![텍스트](./media/node-join.png)

## <a name="next-steps"></a>다음 단계

이 섹션에서는 Linux 작업자 우리의 Kubernetes 클러스터에 가입 하는 방법을 설명 했습니다. 이제 6 단계에 대 한 준비가:
> [!div class="nextstepaction"]
> [Kubernetes 리소스 배포](./deploying-resources.md)