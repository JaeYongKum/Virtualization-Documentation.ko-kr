---
title: Linux 노드를 가입
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Linux 노드 v1.13 사용 하 여 Kubernetes 클러스터에 가입 합니다.
keywords: kubernetes, 1.13, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c32cc300fd97eb53605e2f51e6a83e5889747561
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120471"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Linux 노드 클러스터에 가입

[Kubernetes 마스터 노드를 설정](creating-a-linux-master.md) 하 고 [원하는 네트워크 솔루션을 선택](network-topologies.md)했다면 Linux 노드 클러스터에 가입 준비가 되었습니다. 이렇게 하려면 조인 하기 전에 일부 [Linux 노드 준비](joining-linux-workers.md#preparing-a-linux-node) 합니다.
> [!tip]
> **Ubuntu 16.04**방향으로 Linux 지침에 맞게 조정 됩니다. Kubernetes를 실행 하려면 인증 다른 Linux 배포판 대체할 수 있는 해당 하는 명령을 제공 해야 합니다. 또한 성공적으로 상호 작용 Windows 합니다.

## <a name="preparing-a-linux-node"></a>Linux 노드 준비

> [!NOTE]
> 명시적으로 지정 하지 않으면는 **권한 루트 사용자 셸**명령을 실행 합니다.

먼저, 루트 셸 갖도록:

```bash
sudo –s
```

컴퓨터에 최신 상태 인지 확인 합니다.

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Docker 설치

컨테이너를 사용할 수, Docker와 같은 컨테이너 엔진을 해야 합니다. 최신 버전을 가져오려면 Docker 설치에 대 한 [다음이 지침](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 을 사용할 수 있습니다. 해당 docker를 실행 하 여 올바르게 설치 되었는지 확인할 수 있습니다 `hello-world` 이미지:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Kubeadm 설치

다운로드 `kubeadm` 바이너리 Linux 배포판 및 클러스터 초기화 합니다.

> [!Important]  
> Linux 배포판에 따라 대체 해야 할 수 `kubernetes-xenial` 아래 올바른 [코드](https://wiki.ubuntu.com/Releases)를 사용 하 여 합니다.

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>스왑 사용 안 함

Linux에서 Kubernetes 스왑 공간을 해제할 수에 필요 합니다.

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Flannel에만 해당) Iptables IPv4 트래픽을 브리지

Flannel 사용 하도록 설정 하는 것이 좋습니다. 네트워킹 솔루션으로 선택한 경우 iptables 체인에 대 한 IPv4 트래픽이 브리지입니다. [마스터 아직](network-topologies.md#flannel-in-host-gateway-mode) 있는 하며 가입 예정인 Linux 노드에 대 한 반복 해야 합니다. 수행할 수 있습니다 다음 명령을 사용 하 여:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Kubernetes 인증서를 복사 합니다.

**보통 (비 루트) 사용자로**, 다음 3 단계를 수행 합니다.

1. Linux 디렉터리에 대 한 Kubernetes를 만듭니다.

```bash
mkdir -p $HOME/.kube
```

2. Kubernetes 인증서 파일을 복사 (`$HOME/.kube/config`) [마스터에서](./creating-a-linux-master.md#collect-cluster-information) 다른 이름으로 저장 하 고 `$HOME/.kube/config` 작업자에 있습니다.

> [!tip]
> 노드 간에 config 파일을 전송 [WinSCP](https://winscp.net/eng/download.php) 같은 scp 기반 도구를 사용할 수 있습니다.

3. 다음과 같이 복사한 구성 파일의 파일 소유권을 설정 합니다.

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>노드 가입

마지막으로, 클러스터에 가입 하려면 실행은 `kubeadm join` **루트로** [아래로 앞서 설명한](./creating-a-linux-master.md#initialize-master) 명령:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

성공 하면이 유사한 출력이 표시 되어야 합니다.

![텍스트](./media/node-join.png)

## <a name="next-steps"></a>다음 단계

이 섹션에서는 Linux 작업자 우리의 Kubernetes 클러스터에 가입 하는 방법을 설명 했습니다. 이제 6 단계에 대 한 준비가:
> [!div class="nextstepaction"]
> [Kubernetes 리소스 배포](./deploying-resources.md)