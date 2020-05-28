# [Windows 기반 컨테이너 설명서](index.md) 

# 개요
## [Windows 컨테이너 정보](about/index.md)
## [컨테이너와 VM 비교](about/containers-vs-vm.md)
## [시스템 요구 사항](deploy-containers/system-requirements.md)
## [FAQ](about/faq.md)

# 시작
## [환경 설정](quick-start/set-up-environment.md)
## [첫 번째 컨테이너 실행](quick-start/run-your-first-container.md)
## [샘플 앱 컨테이너화](quick-start/building-sample-app.md)

# 자습서
## Windows 컨테이너 빌드
### [Dockerfile 작성](manage-docker/manage-windows-dockerfile.md)
### [Dockerfile 최적화](manage-docker/optimize-windows-dockerfile.md)
## Azure Kubernetes Service에서 실행
### [AKS에서 Windows 컨테이너 클러스터 만들기](/azure/aks/windows-container-cli)
### [현재 제한 사항](/azure/aks/windows-node-limitations)
## Service Fabric에서 실행
### [첫 번째 컨테이너 배포](/azure/service-fabric/service-fabric-quickstart-containers)
### [Windows 컨테이너에서 .NET 애플리케이션 배포](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Azure App Service에서 실행
### [Azure App Service 빠른 시작](/azure/app-service/app-service-web-get-started-windows-container)
### [Windows 컨테이너 및 Azure App Service로 ASP.NET 앱 마이그레이션](/azure/app-service/app-service-web-tutorial-windows-containers-custom-fonts)
## Windows의 Linux 컨테이너
### [개요](deploy-containers/linux-containers.md)
### [첫 번째 LCOW 컨테이너 실행](quick-start/quick-start-windows-10-linux.md)
## Windows 참가자 프로그램에서 컨테이너 사용
### [개요](deploy-containers/insider-overview.md)

# 개념
## Windows 컨테이너 기본 정보
### [컨테이너 기본 이미지](manage-containers/container-base-images.md)
### [격리 모드](manage-containers/hyperv-container.md)
### [버전 호환성](deploy-containers/version-compatibility.md)
### [컨테이너 업데이트](deploy-containers/update-containers.md)
### [리소스 컨트롤](manage-containers/resource-controls.md)
## Docker
### [Windows의 Docker 엔진](manage-docker/configure-docker-daemon.md)
### [Windows Docker 호스트 원격 관리](management/manage_remotehost.md)
## 컨테이너 오케스트레이션
### [개요](about/overview-container-orchestrators.md)
### Windows의 Kubernetes
#### [Windows의 Kubernetes](kubernetes/getting-started-kubernetes-windows.md)
#### [Kubernetes 마스터 만들기](kubernetes/creating-a-linux-master.md)
#### [네트워크 솔루션 선택](kubernetes/network-topologies.md)
#### [Windows 작업자 조인](kubernetes/joining-windows-workers.md)
#### [Linux 작업자 조인](kubernetes/joining-linux-workers.md)
#### [Kubernetes 리소스 배포](kubernetes/deploying-resources.md)
#### [문제 해결](kubernetes/common-problems.md)
#### [Windows 서비스형 Kubernetes](kubernetes/kube-windows-services.md)
#### [Kubernetes 이진 파일 컴파일](kubernetes/compiling-kubernetes-binaries.md)
### Service Fabric
#### [Service Fabric 및 컨테이너](/azure/service-fabric/service-fabric-containers-overview)
#### [리소스 관리](/azure/service-fabric/service-fabric-resource-governance)
### Docker Swarm
#### [Swarm 모드](manage-containers/swarm-mode.md)
## 작업
### Group Managed Service Accounts
#### [gMSA 만들기](manage-containers/manage-serviceaccounts.md)
#### [gMSA를 사용하도록 앱 구성](manage-containers/gmsa-configure-app.md)
#### [gMSA를 사용하여 컨테이너 실행](manage-containers/gmsa-run-container.md)
#### [gMSA를 사용하여 컨테이너 오케스트레이션](manage-containers/gmsa-orchestrate-containers.md)
#### [gMSA 문제 해결](manage-containers/gmsa-troubleshooting.md)
### [프린터 서비스](deploy-containers/print-spooler.md)
## 네트워킹
### [개요](container-networking/architecture.md)
### [네트워크 토폴로지 및 드라이버](container-networking/network-drivers-topologies.md)
### [네트워크 격리 및 보안](container-networking/network-isolation-security.md)
### [고급 네트워킹 옵션 구성](container-networking/advanced.md)
## 스토리지
### [개요](manage-containers/container-storage.md)
### [영구 스토리지](manage-containers/persistent-storage.md)
## 디바이스
### [하드웨어 디바이스](deploy-containers/hardware-devices-in-containers.md)
### [GPU 가속](deploy-containers/gpu-acceleration.md)

# 참조
## [기본 이미지 서비스 수명 주기](deploy-containers/base-image-lifecycle.md)
## [바이러스 백신 최적화](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [컨테이너 플랫폼 도구](deploy-containers/containerd.md)
## [컨테이너 OS 이미지 EULA](Images_EULA.md)

# 리소스
## [알려진 문제](manage-containers/known-issues.md)
## [컨테이너 샘플](samples.md)
## [문제 해결](troubleshooting.md)
## [컨테이너 포럼](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [커뮤니티 동영상 및 블로그](communitylinks.md)
