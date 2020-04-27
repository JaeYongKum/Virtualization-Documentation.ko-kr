---
title: 사용자 지정 가상 머신 갤러리 만들기
description: Windows 10 크리에이터스 업데이트 이상에서 가상 머신 갤러리에 나만의 항목을 빌드하세요.
keywords: windows 10, hyper-v, 빨리 만들기, 가상 머신, 갤러리
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: 1348b9923d9de1314818f13414abdacee2cb9735
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439718"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>사용자 지정 가상 머신 갤러리 만들기

> Windows 10 Fall Creators Update 이상

Fall Creators Update에서 가상 머신 갤러리를 포함하도록 빨리 만들기가 확장되었습니다.

![사용자 지정 이미지로 VM 갤러리 빨리 만들기](media/vmgallery.png)

갤러리에 Microsoft 및 Microsoft 파트너가 제공한 이미지 집합이 있지만 나만의 이미지도 나열할 수 있습니다.

이 문서에서는 다음 사항에 대해 자세히 설명합니다.

* 갤러리와 호환되는 가상 머신 빌드
* 새로운 갤러리 소스 만들기
* 갤러리에 사용자 지정 갤러리 소스 추가

## <a name="gallery-architecture"></a>갤러리 아키텍처

가상 머신 갤러리는 Windows 레지스트리에 정의되어 있는 일련의 가상 머신 원본에 대한 그래픽 보기입니다.  각 가상 머신 소스는 가상 머신을 목록 항목으로 하는 JSON 파일에 대한 경로(로컬 경로 또는 URI)입니다.

갤러리에 보이는 가상 머신 목록에는 첫 번째 원본의 전체 내용 다음에 두 번째 원본의 내용이 나오는 방식으로 사용 가능한 가상 머신이 모두 나열될 때까지 표시됩니다.  이 목록은 갤러리를 시작할 때마다 동적으로 생성됩니다.

![갤러리 아키텍처](media/vmgallery-architecture.png)

레지스트리 키: `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

값 이름: `GalleryLocations`

형식: `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>갤러리와 호환되는 가상 머신 만들기

갤러리의 가상 머신은 디스크 이미지(.iso) 또는 가상 하드 드라이브(.vhdx)일 수 있습니다.

가상 하드 드라이브로 만든 가상 머신에는 몇 가지 구성 요구 사항이 있습니다.

1. UEFI 펌웨어를 지원하도록 빌드합니다. Hyper-V를 사용하여 만든 경우 2세대 VM입니다.
1. 가상 하드 드라이브의 용량은 20GB 이상이어야 하며, 이것이 최대 크기입니다.  VM이 활발히 사용하지 않는 공간은 Hyper-V가 차지하지 않습니다.

### <a name="testing-a-new-vm-image"></a>새 VM 이미지 테스트

가상 머신 갤러리는 로컬 설치 원본에서 설치하는 것과 동일한 메커니즘을 사용하여 가상 머신을 만듭니다.

가상 머신 이미지가 부팅되고 실행되는지 확인하려면 다음을 수행합니다.

1. VM 갤러리(Hyper-V 빨리 만들기)를 열고 **로컬 설치 원본**을 선택합니다.
  ![로컬 설치 원본을 사용하기 위한 단추](media/use-local-source.png)
1. **설치 원본 변경**을 선택합니다.
  ![로컬 설치 원본을 사용하기 위한 단추](media/change-source.png)
1. 갤러리에서 사용할 .iso 또는 .vhdx를 선택합니다.
1. 이미지가 Linux 이미지인 경우 보안 부팅 옵션을 선택 취소합니다.
  ![로컬 설치 원본을 사용하기 위한 단추](media/toggle-secure-boot.png)
1. 가상 머신을 만듭니다.  가상 머신이 제대로 부팅되면 갤러리를 사용할 준비가 된 것입니다.

## <a name="build-a-new-gallery-source"></a>새 갤러리 소스 빌드

다음 단계에서는 새 갤러리 소스를 만듭니다.  이것은 가상 머신을 나열하고 갤러리에 보이는 모든 추가 정보를 추가하는 JSON 파일입니다.

텍스트 정보:

![레이블이 지정된 갤러리 텍스트 위치](media/gallery-text.png)

* **이름** - 필수 - 왼쪽 열과 가상 머신 보기의 맨 위에 표시되는 이름입니다.
* **게시자** - 필수
* **설명** - 필수 - VM을 설명하는 문자열 목록입니다.
* **버전** - 필수
* lastUpdated - 기본값은 0001년 1월 1일 월요일입니다.

  yyyy-mm-ddThh:mm:ssZ 형식이어야 합니다.

  다음 PowerShell 명령은 오늘 날짜를 올바른 형식으로 제공하여 클립보드에 넣습니다.

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* 로캘 - 기본값은 빈 항목입니다.

그림:

![레이블이 지정된 갤러리 그림 위치](media/gallery-pictures.png)

* **로고** - 필수
* symbol
* 축소판 그림

물론 가상 머신(.iso 또는.vhdx)도 필요합니다.

해시를 생성하려면 다음 PowerShell 명령을 사용하면 됩니다.

  ``` PowerShell
  Get-FileHash -Path .\TMLogo.jpg -Algorithm SHA256
  ```

아래 JSON 템플릿에는 시작 항목과 갤러리의 스키마가 있습니다.  VSCode에서 편집하면 IntelliSense가 자동으로 제공됩니다.

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>갤러리를 VM 갤러리 UI에 연결

VM 갤러리에 사용자 지정 갤러리 소스를 추가하는 가장 쉬운 방법은 regedit에 추가하는 것입니다.

1. **regedit.exe**를 엽니다.
1. `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`으로 이동합니다.
1. `GalleryLocations` 항목을 찾습니다.

    이미 존재하는 경우에는 **편집** 메뉴로 이동하여 **수정**합니다.

    이미 존재하지 않는 경우에는 **편집** 메뉴로 이동하여 **신규**부터 **다중 문자열 값**까지 탐색합니다.

1. 갤러리를 `GalleryLocations` 레지스트리 키에 추가합니다.

    ![새 항목이 포함된 갤러리 레지스트리 키](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>문제 해결

### <a name="check-for-errors-loading-gallery"></a>갤러리 로딩 오류 확인

가상 머신 갤러리는 Windows 이벤트 뷰어에 오류를 보고합니다.  오류를 확인하려면 다음을 수행하세요.

1. 이벤트 뷰어를 엽니다.
1. **Windows 로그** -> **애플리케이션**으로 이동합니다.
1. 소스 VMCreate에서 이벤트를 찾습니다.

## <a name="resources"></a>리소스

GitHub [링크](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery)에는 몇 가지 갤러리 스크립트와 도우미가 있습니다.

샘플 갤러리 항목은 [여기](https://go.microsoft.com/fwlink/?linkid=851584)를 참조하세요.  기본 제공 갤러리를 정의하는 JSON 파일입니다.
