# <a name="hyper-v-backup-approaches"></a>Hyper-v 백업 방법
Hyper-v에서는 가상 컴퓨터 내에서 사용자 지정 백업 소프트웨어를 실행할 필요 없이 호스트 운영 체제에서 백업 가상 컴퓨터를 허용 합니다.  개발자가 자신의 요구에 따라 활용 하는 데 사용할 수 있는 방법은 여러 가지가 있습니다.
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS 작성기
Hyper-v는 모든 버전의 Hyper-v 지원 되는 Windows Server에서 VSS 글쓰기를 구현 합니다.  이 VSS 작성기 백업 가상 컴퓨터에 기존 VSS 인프라를 활용 하는 개발자 수 있습니다.  그러나 서버에서 모든 가상 컴퓨터는 백업 되 동시에 작 백업 작업을 위해 설계 되었습니다.

이 아키텍처를 – 더 잘 이해 하려면이 프레젠테이션에서를 참조 하세요.https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-v WMI 기반 백업
Windows Server 2016 부터는 Hyper-v Hyper-v WMI API를 통해 백업 지 원하는 시작 합니다.  이 접근 방식은 여전히 VSS 백업을 목적에 대 한 가상 컴퓨터 내에서 사용 하지만 더 이상 호스트 운영 체제에서 VSS를 사용 합니다.  대신, 기준점 및 복원 변경 (RCT) 추적의 조합에서 효율적으로 가상 컴퓨터를 백업 개발자에 대 한 정보에 액세스할 수 있도록 사용 됩니다.  이 방법은 Windows Server 2016에서 사용할 수 있는 이상만 호스트에서 VSS를 사용 하 여 보다 더 확장성 합니다.

이 아키텍처를 – 더 잘 이해 하려면이 프레젠테이션에서를 참조 하세요.https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

여기에 사용할 수 있는 이러한 Api를 사용 하는 방법에도 예제가입니다.https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI 기반 백업에서 백업 읽기 위한 메서드
Hyper-v WMI를 사용 하 여 가상 컴퓨터 백업을 만들 때 백업에서 실제 데이터 읽기에 대 한 세 가지 방법이 있습니다.  마다 고유한 장점과 단점이 있습니다.
### <a name="wmi-export"></a>WMI 내보내기
개발자 (위의 예제에서 사용) 되는 Hyper-v WMI 인터페이스를 통해 백업 데이터를 내보낼 수 있습니다.  Hyper-v는 가상 하드 드라이브에 변경 내용을 컴파일하고 요청 된 위치에 파일을 복사 합니다.  이 방법을 사용 하기 쉽고 모든 시나리오 작동 하며 원격으로 사용할 수는 합니다.  그러나 자주 생성 된 가상 하드 드라이브 많은 양의 네트워크를 통해 전송 하는 데이터를 만듭니다.
### <a name="win32-apis"></a>Win32 Api
개발자가 SetVirtualDiskInformation, GetVirtualDiskInformation를 사용할 수 있으며 QueryChangesVirtualDisk Api 가상 하드 디스크 Win32 API에 설명 된 대로 설정 여기: https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/ 는 이러한 Api를 사용 하려면 Hyper-v WMI 해야 참조를 만드는 데 사용할 참고 연결 된 가상 컴퓨터에서 점입니다.  이러한 Win32 Api 백업된 가상 컴퓨터의 데이터에 대 한 효율적인 액세스에 대 한 다음 수 있습니다.  Win32 Api에 몇 가지 제한이 합니다.
*   로컬로 액세스할만 수 있습니다.
*   안 함에서 데이터 읽기를 지원 하지 가상 하드 디스크 파일 공유
*   가상 하드 디스크의 내부 구조와 관련 된 데이터 주소 반환

### <a name="remote-shared-virtual-disk-protocol"></a>원격 공유 가상 디스크 프로토콜
마지막으로, 개발자 효율적으로 공유 가상 하드 디스크 파일 –에서 백업 데이터 정보에 액세스 해야 할 경우 원격 공유 가상 디스크 프로토콜을 사용 하 여 필요한 됩니다.  이 프로토콜은 여기에서 설명 합니다.https://msdn.microsoft.com/en-us/library/dn393384.aspx
