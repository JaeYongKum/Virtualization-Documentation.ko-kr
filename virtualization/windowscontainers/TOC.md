# [Windows 문서의 컨테이너](index.md) 

# 개요
## [Windows 컨테이너 정보](about/index.md)
## [시스템 요구 사항](deploy-containers/system-requirements.md)
## [FAQ](about/faq.md)

# 시작
## [환경 설정](quick-start/set-up-environment.md)
## [첫 번째 컨테이너 실행](quick-start/run-your-first-container.md)
## [샘플 앱 Containerize](quick-start/building-sample-app.md)

# 자습서
## Windows 컨테이너 빌드
### [Dockerfile 작성](manage-docker/manage-windows-dockerfile.md)
### [Dockerfile 최적화](manage-docker/optimize-windows-dockerfile.md)
## Windows의 Kubernetes
### [Windows의 Kubernetes](kubernetes/getting-started-kubernetes-windows.md)
### [Kubernetes 마스터 만들기](kubernetes/creating-a-linux-master.md)
### [네트워크 솔루션 선택](kubernetes/network-topologies.md)
### [Windows 작업자 참가](kubernetes/joining-windows-workers.md)
### [Linux 직원 가입](kubernetes/joining-linux-workers.md)
### [Kubernetes 리소스 배포](kubernetes/deploying-resources.md)
### [문제 해결](kubernetes/common-problems.md)
### [Kubernetes의 Windows 서비스](kubernetes/kube-windows-services.md)
### [Kubernetes 이진 컴파일](kubernetes/compiling-kubernetes-binaries.md)
## Windows의 서비스 패브릭
### [첫 번째 컨테이너 배포](/azure/service-fabric/service-fabric-quickstart-containers)
### [Windows 컨테이너에서 .NET 응용 프로그램 배포](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Windows의 Linux 컨테이너
### [개요](deploy-containers/linux-containers.md)
### [첫 번째 LCOW 컨테이너를 실행 합니다.](quick-start/quick-start-windows-10-linux.md)

# 개념
## Windows 컨테이너 Essentials
### [컨테이너 기본 이미지](manage-containers/container-base-images.md)
### [격리 모드](manage-containers/hyperv-container.md)
### [버전 호환성](deploy-containers/version-compatibility.md)
### [자원 컨트롤](manage-containers/resource-controls.md)
## Docker
### [Windows의 Docker 엔진](manage-docker/configure-docker-daemon.md)
### [Docker swarm](manage-containers/swarm-mode.md)
### [Windows Docker 호스트의 원격 관리](management/manage_remotehost.md)
## 컨테이너 오케스트레이션
### [개요](about/overview-container-orchestrators.md)
## 작업량
### 그룹 관리 서비스 계정
#### [gMSA를 만듭니다.](manage-containers/manage-serviceaccounts.md)
#### [앱을 구성 하 여 gMSA 사용](manage-containers/gmsa-configure-app.md)
#### [GMSA를 사용 하 여 컨테이너 실행](manage-containers/gmsa-run-container.md)
#### [GMSA를 사용 하 여 컨테이너 조정](manage-containers/gmsa-orchestrate-containers.md)
#### [GMSAs 문제 해결](manage-containers/gmsa-troubleshooting.md)
### [프린터 서비스](deploy-containers/print-spooler.md)
## 네트워킹
### [개요](container-networking/architecture.md)
### [네트워크 토폴로지 및 드라이버](container-networking/network-drivers-topologies.md)
### [네트워크 격리 및 보안](container-networking/network-isolation-security.md)
### [고급 네트워킹 옵션 구성](container-networking/advanced.md)
## 저장소
### [개요](manage-containers/container-storage.md)
## 장치
### [하드웨어 장치](deploy-containers/hardware-devices-in-containers.md)
### [GPU 가속](deploy-containers/gpu-acceleration.md)

# 참조
## [기본 이미지 서비스 수명 주기](deploy-containers/base-image-lifecycle.md)
## [바이러스 백신 최적화](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [컨테이너 플랫폼 도구](deploy-containers/containerd.md)
## [컨테이너 OS 이미지 EULA](Images_EULA.md)

# 리소스
## [컨테이너 샘플](samples.md)
## [문제 해결](troubleshooting.md)
## [컨테이너 포럼](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [커뮤니티 비디오 및 블로그](communitylinks.md)