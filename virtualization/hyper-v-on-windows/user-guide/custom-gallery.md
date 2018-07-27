---
title: 사용자 지정 가상 컴퓨터 갤러리 만들기
description: Windows 10 크리에이터스 업데이트 이상에서 가상 컴퓨터 갤러리로 자체 항목을 빌드하세요.
keywords: windows 10, hyper-v, 빨리 만들기, 가상 컴퓨터, 갤러리
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: 2235201a56a238cbd5a75b0a6cae64cdb26108a2
ms.sourcegitcommit: edc153ffef01094c2324a0da2f9a301b31015a58
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/23/2018
ms.locfileid: "1928380"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>사용자 지정 가상 컴퓨터 갤러리 만들기

> Windows 10 Fall Creators Update 이상

가을 크리에이터스 업데이트에서 가상 컴퓨터 갤러리를 포함하도록 빨리 만들기가 확장되었습니다.

![사용자 지정 이미지를 통해 VM 갤러리 빨리 만들기](media/vmgallery.png)

Microsoft 및 Microsoft 파트너가 제공하는 이미지 집합이 있더라도 갤러리는 자체 이미지를 나열할 수 있습니다.

자세한 내용은 이 문서를 참조하세요.

* 갤러리와 호환 가능한 가상 컴퓨터를 빌드.
* 새 갤러리 소스 만들기.
* 갤러리에 사용자 지정 갤러리 소스 추가.

## <a name="gallery-architecture"></a>갤러리 아키텍처

가상 컴퓨터 갤러리는 Windows 레지스트리에 정의되어 있는 가상 컴퓨터 소스 집합을 위한 그래픽 보기입니다.  각 가상 컴퓨터 소스는 가상 컴퓨터를 목록 항목으로 하는 JSON 파일에 대한 경로(로컬 경로 또는 URI)입니다.

갤러리에 표시되는 가상 컴퓨터의 목록에는 첫 번째 소스의 전체 내용 다음에 두 번째 소스의 내용이 나오는 식으로 사용 가능한 모든 가상 컴퓨터가 나열될 때까지 내용이 표시됩니다.  갤러리를 시작할 때마다 목록이 동적으로 생성됩니다.

![갤러리 아키텍처](media/vmgallery-architecture.png)

레지스트리 키: `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

값 이름: `GalleryLocations`

유형: `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>갤러리와 호환되는 가상 컴퓨터 만들기

갤러리의 가상 컴퓨터는 디스크 이미지(.iso) 또는 가상 하드 드라이브(.vhdx)가 될 수 있습니다.

가상 하드 드라이브로 구성된 가상 컴퓨터에는 몇 가지 구성 요구 사항이 있습니다.

1. UEFI 펌웨어를 지원하도록 빌드해야 합니다. Hyper-V를 사용해 만들어질 경우 2세대 VM입니다.
1. 가상 하드 드라이브의 용량은 20GB 이상이어야 합니다. 이것이 최대 크기라는 것에 유의하세요.  Hyper-V는 VM이 자주 사용하지 않는 공간을 차지하지 않습니다.

### <a name="testing-a-new-vm-image"></a>새 VM 이미지 테스트

가상 컴퓨터 갤러리는 로컬 설치 원본에서 설치하는 것과 같은 메커니즘을 시용해 가상 컴퓨터를 만듭니다.

가상 컴퓨터 이미지의 유효성을 검사하려면 부팅하고 실행합니다.

1. VM 갤러리(Hyper-V 빨리 만들기)를 열고 **로컬 설치 원본**을 선택합니다.
  ![로컬 설치 원본을 사용하기 위한 단추](media/use-local-source.png)
1. **설치 원본 변경**을 선택합니다.
  ![로컬 설치 원본을 사용하기 위한 단추](media/change-source.png)
1. 갤러리에 사용될 .iso 또는 .vhdx를 선택합니다.
1. 이미지가 Linux 이미지 경우 보안 부팅 옵션을 선택 취소합니다.
  ![로컬 설치 원본을 사용하기 위한 단추](media/toggle-secure-boot.png)
1. 가상 컴퓨터를 만듭니다.  가상 컴퓨터가 제대로 부팅되면 갤러리를 사용할 준비가 됩니다.

## <a name="build-a-new-gallery-source"></a>새 갤러리 원본 빌드

다음 단계에서는 새 갤러리 원본을 만듭니다.  이 원본은 가상 컴퓨터를 나열하고 갤러리에 표시되는 모든 부가적인 정보를 추가하는 JSON 파일입니다.

텍스트 정보:

![레이블이 지정된 갤러리 텍스트 위치](media/gallery-text.png)

* **이름** - 필수 항목 - 가상 컴퓨터 보기의 왼쪽 열과 맨 위에 표시되는 이름입니다.
* **게시자** - 필수 항목
* **설명** - 필수 항목 - VM을 설명하는 문자열 목록입니다.
* **버전** - 필수 항목
* lastUpdated - 1월 1일 월요일, 0001이 기본값입니다.

  yyyy-mm-ddThh:mm:ssZ 형식이어야 합니다.

  다음 PowerShell 명령은 오늘자로 적절한 형식을 제공하며 이를 클립보드에 배치합니다.

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* 로캘 - 기본값은 빈 항목입니다.

사진:

![레이블이 지정된 갤러리 사진 위치](media/gallery-pictures.png)

* **로고** - 필수 항목
* 심볼
* 섬네일

그리고 물론 가상 컴퓨터(.iso 또는.vhdx)도 포함됩니다.

아래 JSON 템플릿은 시작 항목과 갤러리의 스키마를 가지고 있습니다.  VSCode에서 편집하는 경우, 자동으로 IntelliSense를 제공합니다.

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>VM 갤러리 UI에 갤러리 연결

VM 갤러리에 사용자 지정 갤러리 원본을 추가하는 가장 쉬운 방법은 regedit에 추가하는 것입니다.

1. **regedit.exe**를 엽니다.
1. 다음으로 이동합니다. `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`
1. `GalleryLocations` 항목을 찾습니다.

    이미 존재하는 경우에는 **편집** 메뉴로 이동하여 **수정**합니다.

    이미 존재하지 않는 경우에는 **편집** 메뉴의 **신규**부터 **다중 문자열 값**까지 탐색합니다.

1. `GalleryLocations` 레지스트리 키에 갤러리를 추가합니다.

    ![새 항목이 포함된 갤러리 레지스트리 키](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>문제 해결

### <a name="check-for-errors-loading-gallery"></a>갤러리 로드 오류에 대한 확인

가상 컴퓨터 갤러리는 Windows 이벤트 뷰어에 오류를 보고합니다.  오류를 확인하려면

1. 이벤트 뷰어를 엽니다.
1. **Windows 로그** -> **응용 프로그램**을 탐색합니다.
1. 원본 VMCreate에서 이벤트를 찾습니다.

## <a name="resources"></a>리소스

GitHub [링크](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery)에는 여러 갤러리 스크립트 및 도우미가 있습니다.

샘플 갤러리 항목은 [여기](https://go.microsoft.com/fwlink/?linkid=851584)를 참조하세요.  기본 제공 갤러리를 정의하는 JSON 파일입니다.
