---
title: Kubernetes 바이너리 컴파일
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 원본에서 Kubernetes 바이너리를 컴파일 및 크로스 컴파일합니다.
keywords: kubernetes, 1.12, linux, 컴파일
ms.openlocfilehash: 40bf7e65a8910cdab095abb269aa0a92508189cd
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178876"
---
# <a name="compiling-kubernetes-binaries"></a>Kubernetes 바이너리 컴파일 #
Kubernetes 컴파일에는 Go 환경 작업이 필요합니다. 이 페이지는 Linux 바이너리를 컴파일하고 Windows 바이너리를 크로스 컴파일하는 여러 방법에 대해 다룹니다.
> [!NOTE] 
> 이 페이지는 완전히 자유 번째이자 유일한 관심 Kubernetes 개발자에 게 최신 및 최고의 소스 코드를 사용 하 여 실험을 포함 합니다.

> [!tip]
> 구독할 수 최신 개발에 대 한 알림을 받도록 [@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce).

## <a name="installing-go"></a>Go 설치 ##
편의를 위해 임시 사용자 지정 위치에 Go를 설치합니다.

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.11.1.linux-amd64.tar.gz -O go1.11.1.tar.gz
tar -vxzf go1.11.1.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!Note]  
> 이렇게 하면 귀하의 세션에 대한 환경 변수가 설정됩니다. 영구 설정을 위해 `~/.profile`에 `export`를 추가합니다.

`go env`를 실행하여 경로가 올바르게 설정되었는지 확인합니다. Kubernetes 바이너리 구축에는 몇 가지 옵션이 있습니다.

  - [로컬로](#build-locally) 빌드합니다.
  - [Vagrant](#build-with-vagrant)를 사용하여 바이너리를 생성합니다.
  - Kubernetes 프로젝트에서 [컨테이너화된 표준 빌드 스크립트](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts)를 활용합니다. 이를 위해 `make` 단계까지 [로컬로 구축](#build-locally)하는 단계를 따라 수행한 다음 연결된 지침을 활용합니다.

각 노드에 Windows 바이너리를 복사하려면 [WinSCP](https://winscp.net/eng/download.php) 같은 시각 도구나 [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 같은 명령줄 도구를 사용하여 `C:\k` 디렉터리에 바이너리를 전송합니다.


## <a name="building-locally"></a>로컬로 빌드 ##
> [!Tip]  
> "권한이 거부되었습니다" 오류가 발생한 경우 먼저 [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176)의 메모에 따라 Linux `kubelet`을 빌드하여 이를 해결할 수 있습니다.
>  
> _Kubernetes Windows 빌드 시스템에서 버그처럼 보이는 것으로 인해 `_output/bin/deepcopy-gen`을 생성하기 위해 먼저 Linux 바이너리를 빌드해야 합니다. 이를 수행하지 않고 Windows에 빌드하면 빈 `deepcopy-gen`가 생성됩니다._

먼저 Kubernetes 리포지토리를 검색합니다.

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about 
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

이제 빌드할 분기를 확인하고 Linux `kubelet` 바이너리를 빌드합니다. 이는 위에서 언급한 Windows 빌드 오류가 발생하지 않도록 하기 위해 필요합니다. 여기에서는 `v1.12.2`을 사용하겠습니다. `git checkout` 이후에 보류 중인 PR, 패치를 적용하거나 사용자 지정 바이너리에 대해 다른 수정을 할 수 있습니다.

```bash
git checkout tags/v1.12.2
make clean && make WHAT=cmd/kubelet
```

마지막으로 필요한 Windows 클라이언트 바이너리를 빌드합니다(마지막 단계는 나중에 어디에서 Windows 바이너리를 검색할지에 따라 다릅니다).

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

Linux 바이너리를 구축하는 단계는 동일합니다. 명령에서 `KUBE_BUILD_PLATFORMS=windows/amd64` 접두사를 제외하면 됩니다. 이때 출력 디렉터리는 `_output/.../linux/amd64`입니다.


## <a name="build-with-vagrant"></a>Vagrant를 사용하여 빌드 ##
[여기](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant)에서 사용 가능한 Vagrant 설정이 있습니다. 이를 Vagrant VM을 준비하는 데 사용한 다음 이 내부에서 다음 명령을 실행합니다.

```bash
DIST_DIR="${HOME}/kube/"
SRC_DIR="${HOME}/src/k8s-main/"
mkdir ${DIST_DIR}
mkdir -p "${SRC_DIR}"

git clone https://github.com/kubernetes/kubernetes.git ${SRC_DIR}

cd ${SRC_DIR}
git checkout tags/v1.12.2
KUBE_BUILD_PLATFORMS=linux/amd64   build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kubelet 
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kube-proxy 
cp _output/dockerized/bin/windows/amd64/kube*.exe ${DIST_DIR}

ls ${DIST_DIR}
```

