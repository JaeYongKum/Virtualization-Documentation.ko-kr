# <a name="hyper-v-backup-approaches"></a>Hyper-V 백업 방법
Hyper-V를 사용하면 가상 머신 내에서 사용자 지정 백업 소프트웨어를 실행할 필요 없이 호스트 운영 체제에서 가상 머신을 백업할 수 있습니다.  개발자가 필요에 따라 활용할 수 있는 여러 가지 방법이 있습니다.
## <a name="hyper-v-vss-writer"></a>Hyper-V VSS 작성기
Hyper-V는 Hyper-V를 지원하는 모든 버전의 Windows Server에서 VSS 작성기를 구현합니다.  이 VSS 작성기를 통해 개발자는 기존 VSS 인프라를 활용하여 가상 머신을 백업할 수 있습니다.  그러나 이 작성기는 서버의 모든 가상 머신이 동시에 백업되는 소규모 백업 작업용으로 설계되었습니다.

이 아키텍처를 보다 정확하게 이해하려면 https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322 프레젠테이션을 참조하세요.
## <a name="hyper-v-wmi-based-backup"></a>Hyper-V WMI 기반 백업
Windows Server 2016부터 Hyper-V는 Hyper-V WMI API를 통해 백업 지원을 시작했습니다.  이 방법은 백업을 목적으로 한 가상 머신 내에서는 여전히 VSS를 사용하지만, 호스트 운영 체제에서는 더 이상 VSS를 사용하지 않습니다.  그 대신, 참조 지점과 RCT(복원 변경 추적) 조합을 사용하여 개발자가 백업된 가상 머신의 정보를 효율적으로 액세스할 수 있게 해줍니다.  이 방법은 호스트에서 VSS를 사용하는 것보다 확장성이 우수하지만, Windows Server 2016 이상에서만 사용할 수 있습니다.

이 아키텍처를 보다 정확하게 이해하려면 https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 프레젠테이션을 참조하세요. 

이러한 API를 사용하는 방법에 대한 예제는 https://www.powershellgallery.com/packages/xHyper-VBackup 에서 확인할 수 있습니다.
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI 기반 백업에서 백업을 읽는 방법
Hyper-V WMI를 사용하여 가상 머신 백업을 만들 때 백업에서 실제 데이터를 읽는 세 가지 방법이 있습니다.  방법마다 고유한 장점과 단점이 있습니다.
### <a name="wmi-export"></a>WMI 내보내기
개발자는 위의 예제에서 사용한 것처럼 Hyper-V WMI 인터페이스를 통해 백업 데이터를 내보낼 수 있습니다.  Hyper-V는 변경 내용을 가상 하드 드라이브로 컴파일하고 요청된 위치에 파일을 복사합니다.  이 방법은 사용하기 쉽고 모든 시나리오에서 작동하며 원격으로 사용할 수 있습니다.  그러나 생성되는 가상 하드 드라이브가 네트워크를 통해 전송할 대량의 데이터를 만드는 경우가 자주 있습니다.
### <a name="win32-apis"></a>Win32 API
개발자는 여기에 설명된 대로 가상 하드 디스크 Win32 API 세트에서 SetVirtualDiskInformation, GetVirtualDiskInformation 및 QueryChangesVirtualDisk API를 사용할 수 있습니다. https://docs.microsoft.com/windows/desktop/api/_vhd/ 이러한 API를 사용하려면 여전히 Hyper-V WMI를 사용하여 연결된 가상 머신에서 참조점을 만들어야 합니다.  그러면 Win32 API는 백업된 가상 머신의 데이터에 효율적으로 액세스할 수 있습니다.  Win32 API는 다음과 같은 제한이 있습니다.
* 로컬로만 액세스할 수 있습니다.
* 공유 가상 하드 디스크 파일의 데이터를 읽을 수 없습니다.
* 가상 하드 디스크 내부 구조의 상대 데이터 주소를 반환합니다.

### <a name="remote-shared-virtual-disk-protocol"></a>원격 공유 가상 디스크 프로토콜
마지막으로, 개발자가 공유 가상 하드 디스크 파일의 백업 데이터 정보에 효율적으로 액세스해야 하는 경우 원격 공유 가상 디스크 프로토콜을 사용해야 합니다.  이 프로토콜에 대한 내용은 [여기](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)에 설명되어 있습니다.
