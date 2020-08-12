---
title: Linux 노드 클러스터 조인
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
description: V 1.14를 사용 하 여 Linux 노드를 Kubernetes 클러스터에 조인 합니다.
keywords: kubernetes, 1.14, windows, 시작
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: ababeda847badc2058739c8cd2de36d7f46581d8
ms.sourcegitcommit: bb18e6568393da748a6d511d41c3acbe38c62668
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/12/2020
ms.locfileid: "88161892"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>클러스터에 Linux 노드 조인

[Kubernetes 마스터 노드를 설정](creating-a-linux-master.md) 하 고 [원하는 네트워크 솔루션을 선택한](network-topologies.md)후에는 Linux 노드를 클러스터에 조인할 준비가 된 것입니다. 이를 위해서는 먼저 [Linux 노드에 대해 준비](joining-linux-workers.md#preparing-a-linux-node) 해야 합니다.

> [!TIP]
> Linux 지침은 **Ubuntu 16.04**에 맞게 조정 됩니다. 또한 Kubernetes를 실행 하도록 인증 된 다른 Linux 배포판은 대체할 수 있는 동일한 명령도 제공 해야 합니다. Windows와도 상호 운용 됩니다.

## <a name="preparing-a-linux-node"></a>Linux 노드 준비

> [!NOTE]
> 달리 명시적으로 지정 하지 않는 한, **관리자 권한으로 루트 사용자 셸에서**명령을 실행 합니다.

먼저 루트 셸로 이동 합니다.

```bash
sudo –s
```

컴퓨터가 최신 상태 인지 확인 합니다.

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Docker 설치

컨테이너를 사용할 수 있으려면 Docker와 같은 컨테이너 엔진이 필요 합니다. 최신 버전을 얻으려면 Docker 설치에 [이러한 지침](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 을 사용할 수 있습니다. 이미지를 실행 하 여 docker가 올바르게 설치 되었는지 확인할 수 있습니다 `hello-world` .

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Kubeadm 설치

`kubeadm`Linux 배포에 대 한 이진 파일을 다운로드 하 고 클러스터를 초기화 합니다.

> [!IMPORTANT]
> Linux 배포에 따라 `kubernetes-xenial` 아래를 올바른 [코드명](https://wiki.ubuntu.com/Releases)로 바꾸어야 할 수 있습니다.

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl
```

## <a name="disable-swap"></a>교환 사용 안 함

Linux에서 Kubernetes를 사용 하려면 스왑 공간이 꺼져 있어야 합니다.

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Flannel에만 해당) Iptables에 대 한 브리지 IPv4 트래픽 사용

Flannel를 네트워킹 솔루션으로 선택한 경우 iptables 체인에 대해 브리지 된 IPv4 트래픽을 사용 하도록 설정 하는 것이 좋습니다. [마스터에 대해이 작업을 이미 수행](network-topologies.md#flannel-in-host-gateway-mode) 했 고, 이제는 Linux 노드를 조인 하기 위해이 작업을 반복 해야 합니다. 다음 명령을 사용 하 여이 작업을 수행할 수 있습니다.

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Kubernetes certificate 복사

**일반 (루트가 아닌) 사용자**는 다음 3 단계를 수행 합니다.

1. Linux 디렉터리에 대해 Kubernetes를 만듭니다.

```bash
mkdir -p $HOME/.kube
```

2. Master에서 Kubernetes certificate 파일 ( `$HOME/.kube/config` ) [from master](./creating-a-linux-master.md#collect-cluster-information) 을 복사 하 고 작업자에 다른 이름으로 저장 `$HOME/.kube/config` 합니다.

> [!TIP]
> [Winscp](https://winscp.net/eng/download.php) 와 같은 scp 기반 도구를 사용 하 여 노드 간에 구성 파일을 전송할 수 있습니다.

3. 복사 된 구성 파일의 파일 소유권을 다음과 같이 설정 합니다.

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>노드 조인

마지막으로 클러스터를 조인 하려면 `kubeadm join` [이전에 언급](./creating-a-linux-master.md#initialize-master) 한 명령을 **root로**실행 합니다.

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

성공 하면 다음과 유사한 출력이 표시 됩니다.

![Bash의 노드 조인 전체 출력의 스크린샷](./media/node-join.png)

## <a name="next-steps"></a>다음 단계

이 섹션에서는 Kubernetes 클러스터에 Linux 작업자를 조인 하는 방법을 살펴보았습니다. 이제 6 단계를 수행할 준비가 되었습니다.
> [!div class="nextstepaction"]
> [Kubernetes 리소스 배포](./deploying-resources.md)