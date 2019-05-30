# <a name="hyper-v-backup-approaches"></a>Hyper-v 백업 접근 방법
Hyper-v에서는 가상 컴퓨터 내에서 사용자 지정 백업 소프트웨어를 실행할 필요 없이 호스트 운영 체제에서 가상 컴퓨터를 백업할 수 있습니다.  개발자는 필요에 따라 활용할 수 있는 몇 가지 접근 방식이 있습니다.
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS 작성기
Hyper-v는 Hyper-v가 지원 되는 모든 버전의 Windows Server에서 VSS 기록기를 구현 합니다.  이 VSS 작성기를 통해 개발자는 기존 VSS 인프라를 활용 하 여 가상 컴퓨터를 백업할 수 있습니다.  그러나 서버의 모든 가상 컴퓨터가 동시에 백업 되는 소규모 규모의 백업 작업을 위해 디자인 되었습니다.

이 아키텍처를 더 잘 이해 하려면 다음 프레젠테이션을 참조 하세요.https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-v WMI 기반 백업
Windows Server 2016부터 Hyper-v WMI API를 통한 백업 지원이 시작 되었습니다.  이 접근 방법은 백업을 위해 가상 컴퓨터 내에서 VSS를 계속 사용 하지만 호스트 운영 체제에서는 더 이상 VSS를 사용 하지 않습니다.  대신 개발자가 효율적인 방식으로 백업 된 가상 컴퓨터에 대 한 정보에 액세스할 수 있도록 참조 지점 및 회복성 있는 변경 내용 추적 (.RCT)의 조합을 사용 하는 것이 좋습니다.  이 접근 방식은 호스트에서 VSS를 사용 하는 것 보다 확장성이 향상 되지만, Windows Server 2016 이상 에서만 사용할 수 있습니다.

이 아키텍처를 더 잘 이해 하려면 다음 프레젠테이션을 참조 하세요.https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

여기에는 이러한 Api를 사용 하는 방법에 대 한 예도 나와 있습니다.https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI 기반 백업에서 백업을 읽는 방법
Hyper-v WMI를 사용 하 여 가상 컴퓨터 백업을 만드는 경우 백업에서 실제 데이터를 읽는 세 가지 방법이 있습니다.  각각 고유의 장점과 단점이 있습니다.
### <a name="wmi-export"></a>WMI 내보내기
개발자는 위의 예제에 사용 된 것 처럼 Hyper-v WMI 인터페이스를 통해 백업 데이터를 내보낼 수 있습니다.  Hyper-v는 변경 내용을 가상 하드 드라이브로 컴파일하고 파일을 요청 된 위치에 복사 합니다.  이 방법은 사용이 간편 하며, 모든 시나리오에서 작동 하며 원격화 가능 합니다.  그러나 생성 되는 가상 하드 드라이브는 네트워크를 통해 전송 하는 데 많은 양의 데이터를 만드는 경우가 많습니다.
### <a name="win32-apis"></a>Win32 Api
개발자는 다음과 같이 가상 하드 디스크 Win32 API 집합에서 SetVirtualDiskInformation, GetVirtualDiskInformation 및 QueryChangesVirtualDisk Api를 사용할 수 있습니다. https://docs.microsoft.com/windows/desktop/api/_vhd/ 이러한 api를 사용 하려면 참조를 만드는 데 hyper-v WMI를 사용 해야 합니다. 연결 된 가상 컴퓨터의 점  그런 다음이 Win32 Api를 통해 백업 된 가상 컴퓨터의 데이터에 효율적으로 액세스할 수 있습니다.  Win32 Api에는 다음과 같은 몇 가지 제한 사항이 있습니다.
*   로컬 에서만 액세스할 수 있습니다.
*   공유 가상 하드 디스크 파일의 데이터 읽기는 지원 되지 않습니다.
*   가상 하드 디스크의 내부 구조에 상대적인 데이터 주소를 반환 합니다.

### <a name="remote-shared-virtual-disk-protocol"></a>원격 공유 가상 디스크 프로토콜
마지막으로, 개발자가 공유 가상 하드 디스크 파일에서 백업 데이터 정보에 효율적으로 액세스 해야 하는 경우에는 원격 공유 가상 디스크 프로토콜을 사용 해야 합니다.  [여기서](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)는이 프로토콜에 대해 설명 합니다.
