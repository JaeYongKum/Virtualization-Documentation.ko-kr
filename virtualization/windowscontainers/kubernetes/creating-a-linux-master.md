---
title: 처음부터 Kubernetes 마스터
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes 클러스터 마스터 합니다.
keywords: kubernetes, 1.12, 마스터, linux
ms.openlocfilehash: 2bbcf2d382f20d140c73d9b34cf0f13a74debdfa
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178856"
---
# <a name="creating-a-kubernetes-master"></a>Kubernetes 마스터 만들기 #
> [!NOTE]
> 이 가이드 Kubernetes v1.12에서 유효성이 검사 되었습니다. 많이 다르기 때문에 kubernetes 버전에서 버전으로,이 섹션 수 이후 버전의 경우도 마찬가지 보유 하지 않는 가정 합니다. Kubernetes 마스터 kubeadm를 사용 하 여 초기화에 대 한 공식 설명서를 찾을 수 [다음과 같습니다](https://kubernetes.io/docs/setup/independent/install-kubeadm/). [혼합 OS 일정 섹션](#enable-mixed-os-scheduling) 의 사용 하기만 하면 됩니다.

> [!NOTE]  
> 최근에 업데이트 Linux 컴퓨터 따르는; 해야 합니다. Kubernetes 마스터 [kube dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [kube 스케줄러](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)및 [kube apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) 가 되지 되었는지에 포트 Windows 아직와 같은 리소스입니다. 

> [!tip]
> Linux 지침 **Ubuntu 16.04**향해 맞게 조정 됩니다. Kubernetes를 실행 하려면 인증 다른 Linux 배포판 대체할 수 있는 해당 하는 명령을 제공 해야 합니다. 또한 성공적으로 상호 작용 Windows 합니다.


## <a name="initialization-using-kubeadm"></a>Kubeadm를 사용 하 여 초기화 ##
명시적으로 지정 하지 않으면 아래 명령이 **루트로**실행 합니다.

먼저 관리자 권한 루트 셸 갖도록:

```bash
sudo –s
```

컴퓨터가 최신 있는지 확인 합니다.

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Docker 설치 ###
컨테이너를 사용할 수 있도록 Docker와 같은 컨테이너 엔진을 해야 합니다. 최신 버전을 가져오려면 Docker 설치에 대 한 [다음이 지침](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 을 사용할 수 있습니다. 해당 docker이 올바르게 실행 하 여 설치를 확인할 수는 `hello-world` 컨테이너:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Kubeadm 설치 ###
다운로드 `kubeadm` 바이너리 Linux 배포판 및 클러스터를 초기화 합니다.

> [!Important]  
> Linux 배포판에 따라 대체 해야 할 수도 있습니다 `kubernetes-xenial` 아래 올바른 [코드](https://wiki.ubuntu.com/Releases)를 사용 하 여 합니다.

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>마스터 노드 준비 ###
Linux에서 Kubernetes 설정을 해제 스왑 공간에 필요 합니다.

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>마스터를 초기화 합니다. ###
클러스터 서브넷 (예: 10.244.0.0/16) 및 서비스 서브넷 (예: 10.96.0.0/12)를 기록 하 고 kubeadm를 사용 하 여 마스터를 초기화 합니다.

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

몇 분 정도 걸릴 수 있습니다. 완료 되 면이 확인 하면 마스터 초기화 된 같은 화면이 나타납니다.

![텍스트](media/kubeadm-init.png)

> [!tip]
> 주의 kubeadm 조인 명령 출력 *이제* 위 그림에서 손실 되기 전에 합니다.

> [!tip]
> 사용 하려는, 전달할 수 있는 원하는 Kubernetes 버전이 있는 경우는 `--kubernetes-version` 플래그를 kubeadm 합니다.

우리는 아직 완성 되지 않았습니다. 사용 하 여 `kubectl` 으로 일반 사용자는 다음 실행 __**unelevated, 비 루트 사용자 셸의**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
이제 클러스터에 대 한 정보를 보거나 편집 하려면 kubectl를 사용할 수 있습니다.

### <a name="enable-mixed-os-scheduling"></a>혼합 OS 일정 사용 ###
기본적으로 특정 Kubernetes 리소스는 모든 노드에서 예약 하는 방식으로 기록 됩니다. 그러나 multi-OS 환경에서는 Linux 리소스 방해 하거나 Windows 노드 및 그 반대의 경우로 이중 예약 수를 적용 하지 않습니다. 이러한 이유로 [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) 레이블을 적용 해야 합니다. 

이런 점과 하겠습니다 linux 패치 kube 프록시 [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) Linux 대상으로 합니다.

확인 업데이트 전략을 `kube-proxy` DaemonSet [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)로 설정 됩니다.

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

다음으로 [이 nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) 를 다운로드 하 여는 DaemonSet 패치를 Linux를 대상으로 적용:

```bash
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

되 면 성공적으로 표시 "노드 선택기"의 `kube-proxy` 및 다른 DaemonSets로 설정 `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![텍스트](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>클러스터 정보 수집 ###
마스터를 향후 노드를 가입 성공적으로, 다음 정보를 기록해 야 합니다.
  1. `kubeadm join` 출력 ([다음](#initialize-master))에서 명령
    * 예제: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. 클러스터 서브넷을 정의 하는 동안 `kubeadm init` ([다음](#initialize-master))
    * 예제: `10.244.0.0/16`
  3. 서비스 서브넷을 정의 하는 동안 `kubeadm init` ([다음](#initialize-master))
    * 예제: `10.96.0.0/12`
    * 또한 사용 하 여 찾을 수 있습니다. `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube dns 서비스 IP 
    * 예제: `10.96.0.10`
    * 사용 하 여 "클러스터 IP" 필드에서 찾을 수 있습니다. `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` 파일을 생성 한 후 `kubeadm init` ([다음](#initialize-master)). 지침을 따른 경우이 다음 경로에서 찾을 수 있습니다.
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>마스터 확인 ##
몇 분 후 시스템은 다음과 같은 상태여야 합니다.

  - 아래에서 `kubectl get pods -n kube-system`, 모든는 [Kubernetes 마스터 구성 요소에](https://kubernetes.io/docs/concepts/overview/components/#master-components) 대 한 포드 됩니다 `Running` 상태입니다.
  - 호출 `kubectl cluster-info` DNS 추가 기능 외에도 Kubernetes 마스터 API 서버에 대 한 정보가 표시 됩니다.

## <a name="next-steps"></a>다음 단계 ## 
이 섹션에서는 우리 kubeadm를 사용 하 여 Kubernetes 마스터를 설정 하는 방법을 설명 합니다. 이제 3 단계에 대 한 준비가:

> [!div class="nextstepaction"]
> [네트워크 솔루션을 선택합니다.](./network-topologies.md)