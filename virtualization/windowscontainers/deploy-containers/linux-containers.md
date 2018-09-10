# <a name="linux-containers"></a>Linux 컨테이너

이 기능은 [Hyper-V 격리](../manage-containers/hyperv-container.md)를 사용하여 충분한 OS에서 Linux 커널을 실행해 컨테이너를 지원합니다. 이를 구축하기 위한 Windows 및 Hyper-V 변경 사항은 _Windows 10 Fall Creators Update_ 및 _Windows Server(버전 1709)_ 에서 시작되었지만 , Docker 기술이 구축된 오픈 소스 [Moby 프로젝트](https://www.github.com/moby/moby) 및 Linux 커널에서 사용하도록 이를 모아야 했습니다. 

[!VIDEO https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4]

이 작업을 수행하려면 다음 사항이 필요합니다.

- Windows 10 또는 Windows Server Insider Preview 빌드 16267 이상
- Moby 마스터 분기를 기반으로 하는 Docker 디먼 빌드(`--experimental`플래그로 실행)
- 호환되는 Linux 이미지 선택

이 미리 보기에 사용 가능한 시작 가이드가 있습니다.

- [Docker Enterprise Edition 미리 보기](https://blog.docker.com/2017/09/docker-windows-server-1709/)에는 LinuxKit 시스템과, Linux 컨테이너를 실행할 수 있는 Docker EE의 미리 보기가 포함되어 있습니다. 배경 정보를 알아보려면 [LinuxKit를 사용하여 Windows에서 Linux 컨테이너 미리 보기](https://go.microsoft.com/fwlink/?linkid=857061)를 확인하세요.
- [Windows 10 및 Windows Server에서 Hyper-V 격리를 통해 Ubuntu 컨테이너 실행](https://go.microsoft.com/fwlink/?linkid=857067)


## <a name="work-in-progress"></a>작업 진행 중

Moby 프로젝트의 현재 진행 상황은 [GitHub](https://github.com/moby/moby/issues/33850)에서 추적할 수 있습니다.


### <a name="known-app-issues"></a>알려진 앱 문제

이러한 응용 프로그램 모두에는 볼륨 매핑이 요구되며 여기에는 [마운트 바인딩](#Bind-mounts)에 따라 적용되는 일부 제한 사항이 있습니다. 응용 프로그램이 올바르게 시작되거나 실행되지 않습니다.

- MySQL
- PostgreSQL
- WordPress
- Jenkins
- MariaDB
- RabbitMQ


### <a name="bind-mounts"></a>마운트 바인딩

`docker run -v ...`으로 마운팅 볼륨을 바인딩하면 Windows NTFS 파일 시스템에서 파일을 저장하므로 POSIX 작업을 하려면 변환이 필요합니다. 일부 파일 시스템 작동이 현재 부분적으로 구현되거나 구현되지 않습니다. 이에 따라 일부 앱이 호환되지 않을 수 있습니다.

바인딩되어 탑재된 볼륨에 대해 이러한 작업은 현재 작동되지 않습니다.

- MkNod
- XAttrWalk
- XAttrCreate
- Lock
- Getlock
- Auth
- Flush
- INotify

완전히 구현되지 않는 다음과 같은 사항도 있습니다.

- GetAttr – Nlink 개수가 항상 2개로 보고됩니다.
- Open – ReadWrite, WriteOnly, 및 ReadOnly 플래그만 구현됩니다.
