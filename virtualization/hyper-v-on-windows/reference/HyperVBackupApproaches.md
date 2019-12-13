# <a name="hyper-v-backup-approaches"></a>Hyper-v 백업 방법
Hyper-v를 사용 하면 가상 컴퓨터 내에서 사용자 지정 백업 소프트웨어를 실행할 필요 없이 호스트 운영 체제에서 가상 컴퓨터를 백업할 수 있습니다.  개발자는 필요에 따라 활용할 수 있는 몇 가지 방법이 있습니다.
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS 기록기
Hyper-v는 Hyper-v가 지원 되는 모든 버전의 Windows Server에서 VSS 기록기를 구현 합니다.  이 VSS 기록기를 통해 개발자는 기존 VSS 인프라를 활용 하 여 가상 컴퓨터를 백업할 수 있습니다.  그러나 서버에 있는 모든 가상 컴퓨터를 동시에 백업 하는 작은 규모의 백업 작업을 위해 설계 되었습니다.

이 아키텍처를 보다 잘 이해 하려면 다음 프레젠테이션을 참조 하십시오. https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-v WMI 기반 백업
Windows Server 2016부터 hyper-v는 Hyper-v WMI API를 통한 백업 지원을 시작 했습니다.  이 방법은 백업 목적으로 가상 컴퓨터 내에서 VSS를 계속 사용 하지만 호스트 운영 체제에서 더 이상 VSS를 사용 하지 않습니다.  대신 참조 지점과 복원 력 (변경 내용 추적)의 조합을 사용 하 여 개발자가 백업 된 가상 컴퓨터에 대 한 정보를 효율적으로 액세스할 수 있도록 합니다.  이 방법은 호스트에서 VSS를 사용 하는 것 보다 확장성이 뛰어납니다. 그러나 Windows Server 2016 이상 에서만 사용할 수 있습니다.

이 아키텍처를 보다 잘 이해 하려면 다음 프레젠테이션을 참조 하십시오. https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

이러한 Api를 사용 하는 방법에 대 한 예제도 있습니다. https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI 기반 백업에서 백업을 읽는 방법
Hyper-v WMI를 사용 하 여 가상 머신 백업을 만들 때 백업에서 실제 데이터를 읽는 방법에는 세 가지가 있습니다.  각각에는 고유한 장점과 단점이 있습니다.
### <a name="wmi-export"></a>WMI 내보내기
개발자는 위의 예제에서 사용 된 것과 같이 Hyper-v WMI 인터페이스를 통해 백업 데이터를 내보낼 수 있습니다.  Hyper-v는 변경 내용을 가상 하드 드라이브로 컴파일하고 요청 된 위치에 파일을 복사 합니다.  이 메서드는 사용 하기 쉬우며 모든 시나리오에서 작동 하며 원격으로 사용할 수 있습니다.  그러나 생성 되는 가상 하드 드라이브는 네트워크를 통해 전송 하는 대량의 데이터를 만드는 경우가 많습니다.
### <a name="win32-apis"></a>Win32 Api
개발자는 여기에 설명 된 대로 Win32 API 설정 된 가상 하드 디스크의 GetVirtualDiskInformation 및 QueryChangesVirtualDisk Api를 사용할 수 있습니다. https://docs.microsoft.com/windows/desktop/api/_vhd/ 참고: 이러한 Api를 사용 하려면 Hyper-v WMI를 사용 하 여 연결 된 가상 컴퓨터에 대 한 참조 점을 만들어야 합니다.  그런 다음 이러한 Win32 Api를 통해 백업 된 가상 컴퓨터의 데이터에 효율적으로 액세스할 수 있습니다.  Win32 Api에는 다음과 같은 몇 가지 제한 사항이 있습니다.
* 로컬에만 액세스할 수 있습니다.
* 공유 가상 하드 디스크 파일에서 데이터 읽기를 지원 하지 않습니다.
* 가상 하드 디스크의 내부 구조를 기준으로 하는 데이터 주소를 반환 합니다.

### <a name="remote-shared-virtual-disk-protocol"></a>원격 공유 가상 디스크 프로토콜
마지막으로, 개발자가 공유 가상 하드 디스크 파일에서 백업 데이터 정보에 효율적으로 액세스 해야 하는 경우 원격 공유 가상 디스크 프로토콜을 사용 해야 합니다.  이 프로토콜은 [여기](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)에 설명 되어 있습니다.
