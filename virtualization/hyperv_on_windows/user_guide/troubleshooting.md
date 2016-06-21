---
title: &1493496636 Windows 10에서 Hyper-V 문제 해결
description: Windows 10에서 Hyper-V 문제 해결
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &407718909 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
---

# Windows 10에서 Hyper-V 문제 해결

## Windows 10으로 업데이트했고 지금 내 하위 수준(Windows 8.1 또는 Server 2012 R2) 호스트에 연결할 수 없습니다.

Windows 10에서 Hyper-V 관리자는 원격 관리를 위해 WinRM으로 이동했습니다. 이는 이제 Hyper-V 관리자를 사용하여 관리하려면 원격 호스트에서 원격 관리를 활성화해야 함을 의미합니다.

자세한 내용은 <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">원격 Hyper-V 호스트 관리</g><g id="2CapsExtId3" ctype="x-title"></g></g>를 참조하세요.

## 검사점 유형을 변경했지만 여전히 잘못된 유형의 검사점을 적용합니다.

VMConnect에서 검사점을 적용하고 Hyper-V 관리자에서 검사점 유형을 변경하는 경우 검사점은 VMConnect가 열렸을 때 지정된 검사점 유형과 관계 없이 적용됩니다.

VMConnect를 닫았다가 다시 열어 올바른 유형의 검사점을 적용하도록 합니다.

## 플래시 드라이브에서 가상 하드 디스크를 만들려고 할 때 오류 메시지가 표시됩니다.

이러한 파일 시스템은 액세스 제어 목록(ACL)을 제공하지 않으며 4GB보다 큰 파일을 지원하지 않으므로 Hyper-V는 FAT/FAT32 포맷된 디스크 드라이브를 지원하지 않습니다. ExFAT 포맷된 디스크는 제한된 ACL 기능만을 제공하므로 보안상의 이유로 지원되지 않습니다.
PowerShell에 표시된 오류 메시지는 "시스템을 만들지 못했습니다 '\[path to VHD\]': 파일 시스템 제한으로 인해 요청한 작업을 완료할 수 없습니다(0x80070299)."입니다.

NTFS 포맷 드라이브를 사용하세요.

## 설치하려고 할 때 다음과 같은 메시지가 표시됩니다.: "Hyper-V를 설치할 수 없습니다. 프로세서는 두 번째 수준 주소 변환(SLAT)을 지원하지 않습니다."

Hyper-V는 가상 컴퓨터를 실행하기 위해 SLAT가 필요합니다. 컴퓨터가 SLAT를 지원하지 않는 경우 가상 컴퓨터에 대한 호스트가 될 수 없습니다.

관리 도구만을 설치하려는 경우 <g id="6" ctype="x-strong">Windows 기능 설정 또는 해제</g>에서 <g id="4" ctype="x-strong">프로그램 및 기능</g>의 <g id="2" ctype="x-strong">Hyper-V 플랫폼</g>을 선택 취소합니다.






<!--HONumber=May16_HO1-->


