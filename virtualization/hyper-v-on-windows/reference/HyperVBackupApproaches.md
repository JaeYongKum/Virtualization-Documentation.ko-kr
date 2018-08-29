# <a name="hyper-v-backup-approaches"></a>Hyper-v 백업 방법
Hyper-v을 사용 하면 호스트 운영 체제, 가상 컴퓨터 내 사용자 지정 백업 소프트웨어를 실행 하지 않아도에서 백업 가상 컴퓨터를 할 수 있습니다.  자신의 요구에 따라 활용 하는 개발자를 위한 사용할 수 있는 여러 가지가 있습니다.
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS 기록기
Hyper-v에서 모든 버전의 Windows Server Hyper-v 지원 되는 VSS 기록기를 구현 합니다.  이 VSS 기록기 백업 가상 컴퓨터를 기존 VSS 인프라를 활용 하는 개발자 수 있습니다.  그러나 여기에서 서버에 있는 모든 가상 컴퓨터는 백업 동시에 작은 규모 백업 작업을 위해 설계 되었습니다.

이 아키텍처를 향상 – 이해 하려면이 프레젠테이션에 참조:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-v WMI 기반 백업
Windows Server 2016 부터는 Hyper-v Hyper-v WMI API를 통해 백업을 지원 하기 시작 합니다.  이 방법은 여전히 백업을 위해 가상 컴퓨터 내 VSS를 활용 하지만 더이상 VSS를 사용 하 여 호스트 운영 체제에서 합니다.  대신, 참조 (영문) 포인트와 (RCT)를 추적 하는 탄력적인 변경의 조합은 효율적인 방식으로 가상 컴퓨터를 백업한 개발자가 대 한 정보에 액세스할 수 있도록 하려면 사용 됩니다.  이 방법은 이것은 Windows Server 2016에서 사용할 수 있고 나중에 호스트에서 VSS를 사용 하 여 보다 확장성이 향상입니다.

이 아키텍처를 향상 – 이해 하려면이 프레젠테이션에 참조:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

여기 사용할 수 있는 이러한 Api를 사용 하는 방법에 대 한 예로:https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI 기반 백업에서 백업을 읽는 방법
Hyper-v WMI를 사용 하 여 가상 컴퓨터 백업을 만들 때 백업에서 실제 데이터를 읽는데는 세가지 방법이 있습니다.  각는 고유한 장단점이 있습니다.
### <a name="wmi-export"></a>WMI 내보내기
개발자 (으로 위의 예제에서 사용 되는) Hyper-v WMI 인터페이스를 통해 백업 데이터를 내보낼 수 있습니다.  Hyper-v 가상 하드 드라이브에 변경 내용을 컴파일하고 요청한 위치에 파일을 복사 합니다.  이 메서드는 쉽게 사용할 수, 모든 시나리오에 대해 작동 하며 원격입니다.  그러나 자주 생성 된 가상 하드 드라이브 많은 양의 네트워크를 통해 전송 하는 데이터를 만듭니다.
### <a name="win32-apis"></a>Win32 api (영문)
개발자는 SetVirtualDiskInformation GetVirtualDiskInformation를 사용할 수 있는 및 가상 하드 디스크 Win32 API에 QueryChangesVirtualDisk Api 여기에서 설정 명시: https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/ 이러한 Api를 사용 하려면 Hyper-v WMI 여전히 해야 함을 참조 (영문)를 만드는데 사용할 메모 연결 된 가상 컴퓨터에서 포인트입니다.  이러한 Win32 Api 백업한 가상 컴퓨터의 데이터에 대 한 효율적인 액세스에 대 한 다음 허용 합니다.  Win32 Api 수행 여러 제한 사항이 있습니다.
*   만 액세스할 수 로컬로
*   작업 수행에서 데이터를 읽는 지원 하지 가상 하드 디스크 파일 공유
*   가상 하드 디스크의 내부 구조를 기준으로 수 있는 데이터 주소를 반환

### <a name="remote-shared-virtual-disk-protocol"></a>원격 공유 가상 디스크 프로토콜
마지막으로, 개발자가 효율적으로 공유 가상 하드 디스크 파일-에서 백업 데이터 정보에 액세스 해야 하는 경우 원격 공유 가상 디스크 프로토콜을 사용 해야 합니다.  여기는이 프로토콜에 대 한 설명입니다.https://msdn.microsoft.com/en-us/library/dn393384.aspx
