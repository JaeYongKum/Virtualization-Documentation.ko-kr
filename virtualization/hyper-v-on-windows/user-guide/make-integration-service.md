---
title: 고유한 통합 서비스 만들기
description: Windows 10 통합 서비스
keywords: windows 10, Hyper-V, HVSocket, AF_HYPERV
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
ms.openlocfilehash: 966ca3ff267e03e8c380391281c8dde723e4b1dd
ms.sourcegitcommit: f1a08252087e72791ac4d12517c02e39f41c33f0
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/18/2018
ms.locfileid: "1723639"
---
# <a name="make-your-own-integration-services"></a>고유한 통합 서비스 만들기

Windows 10 1주년 업데이트부터 누구나 대상 가상 컴퓨터에 대한 새로운 주소 집합과 특수 끝점을 제공하는 Windows 소켓인 Hyper-V 소켓을 사용하여 Hyper-V 호스트 및 해당 가상 컴퓨터와 통신하는 응용 프로그램을 만들 수 있습니다.  Hyper-V 소켓을 통한 통신은 모두 네트워킹을 사용하지 않고 실행되며 모든 데이터가 동일한 물리적 메모리에 머물게 됩니다.   Hyper-V 소켓을 사용하는 응용 프로그램은 Hyper-V의 통합 서비스와 비슷합니다.

이 문서에서는 Hyper-V 소켓에 구축되는 간단한 프로그램을 만드는 방법을 안내합니다.

**지원되는 호스트 OS**
* Windows 10 이상
* Windows Server 2016 이상

**지원되는 게스트 OS**
* Windows 10 이상
* Windows Server 2016 이상
* Linux 통합 서비스를 포함하는 Linux 게스트([Supported Linux and FreeBSD virtual machines for Hyper-V on Windows(Windows의 Hyper-V에 대해 지원되는 Linux 및 FreeBSD 가상 컴퓨터)](https://technet.microsoft.com/library/dn531030.aspx) 참조)
> **참고:** 지원되는 Linux 게스트는 다음에 대해 커널을 지원해야 합니다.
> ```bash
> CONFIG_VSOCKET=y
> CONFIG_HYPERV_VSOCKETS=y
> ```

**기능 및 제한 사항**
* 커널 모드 또는 사용자 모드 동작 지원
* 데이터 스트림만
* 블록 메모리 없음(백업/비디오에는 적합하지 않음)

--------------

## <a name="getting-started"></a>시작

요구 사항:
* C/C++ 컴파일러.  없으면 [Visual Studio 커뮤니티](https://aka.ms/vs)를 확인하세요.
* [Windows 10 SDK](https://developer.microsoft.com/windows/downloads/windows-10-sdk) -- Visual Studio 2015(업데이트 3 포함) 이상에 미리 설치되어 있습니다.
* 한 대 이상의 가상 컴퓨터와 함께 위의 호스트 운영 체제 중 하나를 실행하는 컴퓨터. -- 응용 프로그램 테스트용입니다.

> **참고:** Hyper-V 소켓용 API는 얼마 뒤 Windows 10에서 공개되었습니다.  HVSocket을 사용하는 응용 프로그램은 Windows 10 호스트와 게스트에서 실행되지만 빌드 14290 이후의 Windows SDK로만 개발할 수 있습니다.

## <a name="register-a-new-application"></a>새 응용 프로그램 등록
Hyper-V 소켓을 사용하려면 응용 프로그램을 Hyper-V 호스트의 레지스트리에 등록해야 합니다.

레지스트리에 서비스를 등록하면 다음이 가능합니다.
*  WMI 관리를 통해 서비스 사용, 사용 안 함 및 사용 가능한 서비스 나열
*  가상 컴퓨터와 직접 통신할 수 있는 권한

다음 PowerShell은 이름이 "HV Socket Demo"인 새 응용 프로그램을 등록합니다.  관리자 권한으로 실행해야 합니다.  수동 지침은 다음과 같습니다.

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```


**레지스트리 위치 및 정보:**
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
이 레지스트리 위치에는 여러 GUID가 있습니다.  모두 기본 제공 서비스입니다.

서비스별 레지스트리 정보:
* `Service GUID`
    * `ElementName (REG_SZ)` -- 이 서비스의 친근한 이름

고유 서비스를 등록하려면 자체 GUID 및 친근한 이름을 사용하여 새 레지스트리 키를 만듭니다.

친근한 이름이 새 응용 프로그램과 연결됩니다.  이 이름은 성능 카운터와 GUID가 적합하지 않은 다른 위치에 표시됩니다.

레지스트리 항목은 다음과 같이 표시됩니다.
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

> **참고:** Linux 게스트용 서비스 GUID는 GUID가 아니라 `svm_cid`및 `svm_port`를 통해 주소 지정이 되는 VSOCK 프로토콜을 사용합니다. Windows에서 이러한 비일관성을 해결하기 위해 잘 알려진 GUID가 호스트의 서비스 플랫폼으로 사용되고, 이는 게스트에서 포트로 변환됩니다. 서비스 GUID를 사용자 지정하려면 첫 번째 "00000000"을 원하는 포트 번호로 변경하면 됩니다. 예를 들어 "00000ac9"는 포트 2761입니다.
> ```C++
> // Hyper-V Socket Linux guest VSOCK template GUID
> struct __declspec(uuid("00000000-facb-11e6-bd58-64006a7986d3")) VSockTemplate{};
>
>  /*
>   * GUID example = __uuidof(VSockTemplate);
>   * example.Data1 = 2761; // 0x00000AC9
>   */
> ```
>

> ** 팁: **  PowerShell에서 GUID를 생성하고 클립보드에 복사하려면 다음을 실행합니다.
>``` PowerShell
>(New-Guid).Guid | clip.exe
>```

## <a name="create-a-hyper-v-socket"></a>Hyper-V 소켓 만들기

대부분의 기본적인 경우 소켓을 정의하려면 주소 집합, 연결 유형 및 프로토콜이 필요합니다.

간단한 [소켓 정의](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
)

``` C
// Windows
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);

// Linux guest
int socket(int domain, int type, int protocol);
```

Hyper-V 소켓의 경우:
* 주소 집합 - `AF_HYPERV`(Windows) 또는 `AF_VSOCK`(Linux 게스트)
* 유형 - `SOCK_STREAM`
* 프로토콜 - `HV_PROTOCOL_RAW`(Windows) 또는 `0`(Linux 게스트)


선언/인스턴스화 예제는 다음과 같습니다.
``` C
// Windows
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);

// Linux guest
int sock = socket(AF_VSOCK, SOCK_STREAM, 0);
```

## <a name="bind-to-a-hyper-v-socket"></a>Hyper-V 소켓에 바인딩

소켓을 연결 정보에 바인딩합니다.

편의를 위해 아래에 함수 정의를 복사해 두었으므로 바인딩에 대한 자세한 내용은 [여기](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx)를 참조하세요.

``` C
// Windows
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);

// Linux guest
int bind(int sockfd, const struct sockaddr *addr,
         socklen_t addrlen);
```

호스트 컴퓨터의 IP 주소와 해당 호스트의 포트 번호로 구성된 표준 인터넷 프로토콜 주소 집합(`AF_INET`)의 소켓 주소와는 달리, `AF_HYPERV`는 위에서 정의한 가상 컴퓨터의 ID와 응용 프로그램 ID를 사용하여 연결을 수립합니다. Linux 게스트 `AF_VSOCK`에서의 바인딩에서 `svm_cid` 및 `svm_port`가 사용되는 경우입니다.

Hyper-V 소켓은 네트워킹 스택, TCP/IP, DNS 등에 종속되지 않으므로, 소켓 끝점에서 호스트 이름이 아니라 연결을 분명히 설명하는 비 IP 형식이 필요합니다.

Hyper-V 소켓의 소켓 주소 정의는 다음과 같습니다.

``` C
// Windows
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};

// Linux guest
// See include/uapi/linux/vm_sockets.h for more information.
struct sockaddr_vm {
    __kernel_sa_family_t svm_family;
    unsigned short svm_reserved1;
    unsigned int svm_port;
    unsigned int svm_cid;
    unsigned char svm_zero[sizeof(struct sockaddr) -
                   sizeof(sa_family_t) -
                   sizeof(unsigned short) -
                   sizeof(unsigned int) - sizeof(unsigned int)];
};
```

IP 또는 호스트 이름 대신 AF_HYPERV 끝점은 다음 두 GUID에 크게 의존합니다.
* VM ID – VM마다 할당된 고유의 ID입니다.  VM ID는 다음 PowerShell 코드 조각을 사용하여 찾을 수 있습니다.
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* 서비스 ID – [위에서 설명한](#RegisterANewApplication) GUID로, Hyper-V 호스트 레지스트리에 응용 프로그램과 함께 등록되어 있습니다.

특정 가상 컴퓨터에 대한 연결이 아닌 경우 VMID 와일드 카드 집합도 사용할 수 있습니다.

### <a name="vmid-wildcards"></a>VMID 와일드카드

| 이름 | GUID | 설명 |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | 수신기를 이 VMID에 등록해야 모든 파티션으로부터의 연결을 수락할 수 있습니다. |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | 수신기를 이 VMID에 등록해야 모든 파티션으로부터의 연결을 수락할 수 있습니다. |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | 자식에 대한 와일드 카드 주소입니다. 수신기를 이 VMID에 등록해야 모든 자식으로부터의 연결을 수락할 수 있습니다. |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | 루프백 주소 이 VMID 연결을 사용하면 커넥터와 동일한 파티션에 연결합니다. |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | 부모 주소입니다. 이 VMID 연결을 사용하면 커넥터와 동일한 부모 파티션에 연결합니다.* |


\* `HV_GUID_PARENT` 가상 컴퓨터의 부모는 해당 호스트입니다.  컨테이너의 부모는 컨테이너의 호스트입니다.
가상 컴퓨터에서 실행 중인 컨테이너로부터의 연결은 해당 컨테이너를 호스팅하는 VM에 연결합니다.
이 VmId에서 수신은 다음으로부터의 연결을 수락합니다. (컨테이너 내): 컨테이너 호스트
(VM 내: 컨테이너 호스트/컨테이너 없음): VM 호스트
(VM 내부 아님: 컨테이너 호스트/컨테이너 없음): 지원 안 함

## <a name="supported-socket-commands"></a>지원되는 소켓 명령

Socket() Bind() Connect() Send() Listen() Accept()

## <a name="useful-links"></a>유용한 링크
[WinSock API 완료](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

[Hyper-V 통합 서비스 참조](../reference/integration-services.md)