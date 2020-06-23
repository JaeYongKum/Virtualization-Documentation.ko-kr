---
title: Kubernetes 마스터 처음부터
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: how-to
ms.prod: containers
description: Kubernetes 클러스터 마스터 만들기
keywords: kubernetes, 1.14, 마스터, linux
ms.openlocfilehash: a46c8e996162891cc596946d8601bcb590b2b8eb
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192390"
---
# <a name="creating-a-kubernetes-master"></a>Kubernetes 마스터 만들기 #
> [!NOTE]
> Kubernetes v 1.14에서이 가이드의 유효성을 검사 했습니다. 변동성 버전에서 버전으로의 Kubernetes 때문에이 섹션에서는 모든 이후 버전에 대해 true를 포함 하지 않는 가정을 만들 수 있습니다. Kubeadm를 사용 하 여 Kubernetes 마스터를 초기화 하는 공식 설명서는 [여기](https://kubernetes.io/docs/setup/independent/install-kubeadm/)에서 찾을 수 있습니다. 위에서 [혼합 된 OS 일정 섹션](#enable-mixed-os-scheduling) 을 사용 하도록 설정 하기만 하면 됩니다.

> [!NOTE]
> 업데이트를 수행 하려면 최근에 업데이트 된 Linux 컴퓨터가 필요 합니다. [Kube-](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)Kubernetes, [kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)및 [kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) 와 같은 마스터 리소스는 아직 Windows로 이식할 수 없습니다.

> [!tip]
> Linux 지침은 **Ubuntu 16.04**에 맞게 조정 됩니다. 또한 Kubernetes를 실행 하도록 인증 된 다른 Linux 배포판은 대체할 수 있는 동일한 명령도 제공 해야 합니다. Windows와도 상호 운용 됩니다.


## <a name="initialization-using-kubeadm"></a>Kubeadm를 사용 하 여 초기화 ##
달리 명시적으로 지정 하지 않는 한, 아래 명령을 **root**로 실행 합니다.

먼저 승격 된 루트 셸을 가져옵니다.

```bash
sudo –s
```

컴퓨터가 최신 상태 인지 확인 합니다.

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Docker 설치 ###
컨테이너를 사용할 수 있으려면 Docker와 같은 컨테이너 엔진이 필요 합니다. 최신 버전을 얻으려면 Docker 설치에 [이러한 지침](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 을 사용할 수 있습니다. 컨테이너를 실행 하 여 docker가 올바르게 설치 되었는지 확인할 수 있습니다 `hello-world` .

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Kubeadm 설치 ###
`kubeadm`Linux 배포에 대 한 이진 파일을 다운로드 하 고 클러스터를 초기화 합니다.

> [!Important]
> Linux 배포에 따라 `kubernetes-xenial` 아래를 올바른 [코드명](https://wiki.ubuntu.com/Releases)로 바꾸어야 할 수 있습니다.

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl
```

### <a name="prepare-the-master-node"></a>마스터 노드 준비 ###
Linux에서 Kubernetes를 사용 하려면 스왑 공간이 꺼져 있어야 합니다.

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

### <a name="initialize-master"></a>Master 초기화 ###
클러스터 서브넷 (예: 10.244.0.0/16) 및 서비스 서브넷 (예: 10.96.0.0/12)을 적어 두고 kubeadm를 사용 하 여 마스터를 초기화 합니다.

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

몇 분이 걸릴 수 있습니다. 완료 되 면 마스터가 초기화 되었는지 확인 하는 것과 같은 화면이 표시 됩니다.

![text](media/kubeadm-init.png)

> [!tip]
> 이 kubeadm join 명령을 기록해 두어야 합니다. Kubeadm 토큰이 만료 길이가을 사용 `kubeadm token create --print-join-command` 하 여 새 토큰을 만들 수 있습니다.

> [!tip]
> 원하는 Kubernetes 버전을 사용 하려는 경우 플래그를 kubeadm에 전달할 수 있습니다 `--kubernetes-version` .

아직 완료 되지 않았습니다. `kubectl`일반 사용자로 사용 하려면 __ **권한이 없는 비 루트 사용자 셸에서** 다음을 실행 합니다.__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
이제 kubectl을 사용 하 여 클러스터에 대 한 정보를 편집 하거나 볼 수 있습니다.

### <a name="enable-mixed-os-scheduling"></a>혼합 OS 일정 사용 ###
기본적으로 특정 Kubernetes 리소스는 모든 노드에 예약 된 방식으로 작성 됩니다. 그러나 다중 OS 환경에서는 Windows 노드에 대해 Linux 리소스를 방해 하거나 두 번 예약 하지 않으려는 경우, 그 반대의 경우도 가능 합니다. 이러한 이유로 [Nodeselector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) 레이블을 적용 해야 합니다.

이와 관련 하 여 linux kube [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) 를 대상 linux에만 패치 하겠습니다.

먼저. yaml 매니페스트 파일을 저장할 디렉터리를 만들어 보겠습니다.
```bash
mkdir -p kube/yaml && cd kube/yaml
```

DaemonSet의 업데이트 전략이 `kube-proxy` [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)로 설정 되어 있는지 확인 합니다.

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

다음으로, [nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 를 다운로드 하 고 대상 Linux에만 적용 하 여 DaemonSet을 패치 합니다.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

성공적으로 완료 되 면의 "노드 선택기" `kube-proxy` 와 기타 DaemonSets 집합을 표시 해야 합니다.`beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![text](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>클러스터 정보 수집 ###
이후 노드를 마스터에 성공적으로 연결 하려면 다음 정보를 추적 해야 합니다.
  1. `kubeadm join`출력에서 명령 ([여기](#initialize-master))
    * 예: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. 에서 정의 된 클러스터 서브넷 `kubeadm init` ([여기](#initialize-master))
    * 예: `10.244.0.0/16`
  3. 에서 정의 된 서비스 서브넷 `kubeadm init` ([여기](#initialize-master))
    * 예: `10.96.0.0/12`
    * 다음을 사용 하 여 찾을 수도 있습니다.`kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube-dns 서비스 IP
    * 예: `10.96.0.10`
    * 을 사용 하 여 "클러스터 IP" 필드에서 찾을 수 있습니다.`kubectl get svc/kube-dns -n kube-system`
  5. `config`다음에 생성 된 Kubernetes 파일 `kubeadm init` ([여기](#initialize-master)) 지침을 따르는 경우 다음 경로에서 찾을 수 있습니다.
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>마스터를 확인 하는 중 ##
몇 분 후 시스템이 다음과 같은 상태 여야 합니다.

  - 에서 `kubectl get pods -n kube-system` [Kubernetes 마스터 구성 요소](https://kubernetes.io/docs/concepts/overview/components/#master-components) 에 대 한 pod가 있습니다 `Running` .
  - 를 호출 `kubectl cluster-info` 하면 DNS addons 외에도 Kubernetes 마스터 API 서버에 대 한 정보가 표시 됩니다.

> [!tip]
> Kubeadm는 네트워킹을 설정 하지 않으므로 DNS pod는 여전히 또는 상태에 있을 수 있습니다 `ContainerCreating` `Pending` . `Running` [네트워크 솔루션을 선택한](./network-topologies.md)후 상태로 전환 됩니다.

## <a name="next-steps"></a>다음 단계 ##
이 섹션에서는 kubeadm을 사용 하 여 Kubernetes 마스터를 설정 하는 방법을 설명 했습니다. 이제 3 단계를 수행할 준비가 되었습니다.

> [!div class="nextstepaction"]
> [네트워크 솔루션 선택](./network-topologies.md)