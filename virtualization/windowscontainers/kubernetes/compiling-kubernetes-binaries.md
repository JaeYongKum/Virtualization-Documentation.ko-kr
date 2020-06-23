---
title: Kubernetes 이진 파일 컴파일
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: how-to
ms.prod: containers
description: Kubernetes 이진 파일을 소스에서 컴파일하고 크로스 컴파일합니다.
keywords: kubernetes, 1.12, linux, 컴파일
ms.openlocfilehash: a0c9ed6ef0872e19de49fa97f4727b6e0e09ed43
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192350"
---
# <a name="compiling-kubernetes-binaries"></a>Kubernetes 이진 파일 컴파일 #
Kubernetes을 컴파일하려면 작동 중인 Go 환경이 필요 합니다. 이 페이지에서는 Linux 이진 파일을 컴파일하고 Windows 바이너리를 크로스 컴파일하는 여러 가지 방법을 살펴봅니다.
> [!NOTE]
> 이 페이지는 완전히 자발적 이며 최신 & 가장 많은 소스 코드로 시험해 보려는 관심이 있는 Kubernetes 개발자 에게만 포함 되어 있습니다.

> [!tip]
> 최신 개발에 대 한 알림을 받으려면 구독할 수 있습니다 [@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce) .

## <a name="installing-go"></a>Go 설치 ##
간단 하 게 하기 위해 임시, 사용자 지정 위치를 설치 하는 과정을 진행 합니다.

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
> 세션에 대 한 환경 변수를 설정 합니다. 에 `export` 을 (를) 영구적으로 설정 하기 위해에 추가 합니다 `~/.profile` .

`go env`를 실행 하 여 경로가 올바르게 설정 되었는지 확인 합니다. Kubernetes 이진 파일을 빌드하기 위한 몇 가지 옵션이 있습니다.

  - [로컬](#build-locally)에서 빌드합니다.
  - [Vagrant](#build-with-vagrant)를 사용 하 여 이진 파일을 생성 합니다.
  - Kubernetes 프로젝트에서 [표준 컨테이너 화 된 빌드 스크립트](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts) 를 활용 합니다. 이렇게 하려면 [로컬로](#build-locally) 구성 된 단계를 수행 하는 단계를 수행 하 `make` 고 연결 된 지침을 사용 합니다.

Windows 이진 파일을 해당 노드에 복사 하려면 [Winscp](https://winscp.net/eng/download.php) 와 같은 시각적 도구 또는 [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 와 같은 명령줄 도구를 사용 하 여 디렉터리에 전송 `C:\k` 합니다.


## <a name="building-locally"></a>로컬로 빌드 ##
> [!Tip]
> "사용 권한이 거부 되었습니다." 오류가 발생 하는 경우 다음의 참고 사항에 따라 Linux를 먼저 빌드하여 이러한 오류를 방지할 수 있습니다 `kubelet` [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176) .
>
> _Kubernetes Windows 빌드 시스템에서 버그로 표시 되는 것으로 인해 먼저 생성할 Linux 이진 파일을 빌드해야 `_output/bin/deepcopy-gen` 합니다. Windows w/o로 빌드하면 빈이 생성 됩니다 `deepcopy-gen` ._

먼저 Kubernetes 리포지토리를 검색합니다.

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

이제 분기를 체크 아웃 하 여 Linux 이진 파일에서 빌드하고 빌드합니다 `kubelet` . 위에서 언급 한 Windows 빌드 오류를 방지 하기 위해이 작업이 필요 합니다. 여기서는를 사용 `v1.12.2` 합니다. 이후에는 `git checkout` 보류 중인 pr 또는 패치를 적용 하거나 사용자 지정 이진 파일에 대 한 기타 수정 작업을 수행할 수 있습니다.

```bash
git checkout tags/v1.12.2
make clean && make WHAT=cmd/kubelet
```

마지막으로 필요한 Windows 클라이언트 이진 파일을 빌드합니다 (마지막 단계는 나중에 Windows 이진 파일을 검색 해야 하는 위치에 따라 달라질 수 있음).

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

Linux 이진 파일을 빌드하는 단계는 동일 합니다. `KUBE_BUILD_PLATFORMS=windows/amd64`명령에 대 한 접두사를 그냥 둡니다. 대신 출력 디렉터리를 사용할 수 `_output/.../linux/amd64` 있습니다.


## <a name="build-with-vagrant"></a>Vagrant를 사용 하 여 빌드 ##
[여기](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant)에서 사용할 수 있는 Vagrant 설정이 있습니다. 이를 사용 하 여 Vagrant VM을 준비한 다음, 내에서 다음 명령을 실행 합니다.

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

