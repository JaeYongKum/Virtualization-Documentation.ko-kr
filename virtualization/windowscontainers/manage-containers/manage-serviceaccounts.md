---
title: "Windows 컨테이너용 Active Directory 서비스 계정"
description: "Windows 컨테이너용 Active Directory 서비스 계정"
keywords: "docker, 컨테이너, active directory"
author: PatrickLang
ms.date: 11/04/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 0e692f7521e4a15e3e56d4b98f7ca15fe94ee167
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/21/2017
---
# Windows 컨테이너용 Active Directory 서비스 계정

데이터를 안전하게 보호하고 무단 사용을 방지할 수 있도록 사용자와 다른 서비스가 응용 프로그램 및 서비스에 인증된 연결을 만들어야 할 수 있습니다. Windows AD(Active Directory) 도메인은 기본적으로 암호 및 인증서 인증을 모두 지원합니다. Windows 도메인에 가입된 호스트에서 응용 프로그램 또는 서비스를 빌드하는 경우 응용 프로그램 및 서비스는 로컬 시스템 또는 네트워크 서비스로 실행되는 경우 기본적으로 호스트의 ID를 사용합니다. 그렇지 않은 경우 인증을 위해 다른 AD 계정을 대신 구성할 수 있습니다.

Windows 컨테이너는 도메인에 가입될 수는 없지만 장치가 영역에 가입된 경우와 유사하게 Active Directory 도메인 ID를 활용할 수도 있습니다. Windows Server 2012 R2 도메인 컨트롤러에서는 서비스에서 공유하도록 설계된 gMSA(그룹 관리 서비스 계정)라는 새로운 도메인 계정을 도입했습니다. gMSA(그룹 관리 서비스 계정)를 사용하면 Windows 컨테이너 자체 및 컨테이너에 호스트되는 서비스가 특정 gMSA를 도메인 ID로 사용하도록 구성할 수 있습니다. 로컬 시스템 또는 네트워크 서비스로 실행되는 모든 서비스는 도메인에 가입된 호스트의 ID를 사용하는 지금의 경우처럼 앞으로 Windows 컨테이너의 ID를 사용합니다. 컨테이너 이미지에는 실수로 노출될 수 있는 암호나 인증서 개인 키가 저장되지 않으며 저장된 암호나 인증서를 변경하기 위해 다시 빌드할 필요 없이 컨테이너를 개발, 테스트 및 프로덕션 환경으로 다시 배포할 수 있습니다. 


# 용어 및 참조
- [Active Directory](http://social.technet.microsoft.com/wiki/contents/articles/1026.active-directory-services-overview.aspx)는 Windows에서 사용자, 컴퓨터 및 서비스 계정 정보의 검색 및 복제에 사용되는 서비스입니다. 
  - [Active Directory Domain Services](https://technet.microsoft.com/en-us/library/dd448614.aspx)는 컴퓨터와 사용자를 인증하는 데 사용되는 Windows Active Directory 도메인을 제공합니다. 
  - 장치가 Active Directory의 구성원인 경우 _도메인에 가입된_ 것입니다. 도메인에 가입됨은 장치 상태로, 장치에 도메인 컴퓨터 ID를 제공할 뿐 아니라 다양한 도메인에 가입된 서비스도 제공하는 상태입니다.
  - [그룹 관리 서비스 계정](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)(종종 약식으로 gMSA로 사용)은 암호를 공유하지 않고 Active Directory 사용 서비스를 간단히 보호할 수 있도록 지원하는 Active Directory 계정 유형입니다. 여러 컴퓨터나 컨테이너가 필요에 따라 동일한 gMSA를 공유하여 서비스 간 연결을 인증합니다.
- _CredentialSpec_ PowerShell 모듈 - 이 모듈은 컨테이너에서 사용할 그룹 관리 서비스 계정을 구성하는 데 사용됩니다. 스크립트 모듈 및 예제 단계는 [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools)의 ServiceAccount를 참조하세요.

# 작동 방법

현재 그룹 관리 서비스 계정은 컴퓨터나 서비스 간의 연결을 보호하는 데 주로 사용됩니다. 이 계정을 사용하는 일반적인 단계는 다음과 같습니다.

1. gMSA를 만듭니다.
2. gMSA로 실행할 서비스를 구성합니다.
3. 서비스를 실행하는 도메인에 가입된 호스트에 Active Directory에서 gMSA 암호에 액세스할 수 있는 권한을 부여합니다.
4. 데이터베이스나 파일 공유와 다른 서비스에서 gMSA에 액세스하도록 허용합니다.

서비스가 시작되면 도메인에 가입된 호스트는 Active Directory에서 gMSA 암호를 자동으로 가져오고 해당 계정을 사용하여 서비스를 실행합니다. 서비스가 gMSA로 실행되므로 gMSA가 액세스할 수 있는 모든 리소스에 액세스할 수 있습니다.

Windows 컨테이너도 유사한 프로세스를 수행합니다.

1. gMSA를 만듭니다. 기본적으로 도메인 관리자 또는 계정 운영자가 해야 합니다. 그러지 않으면 gMSA를 만들고 관리하는 권한을 이 계정을 사용하는 서비스를 관리하는 관리자에게 위임할 수 있습니다. [gMSA 시작하기](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)를 참조하세요.
2. 도메인에 가입된 컨테이너 호스트에 gMSA 액세스 권한을 부여합니다.
3. 데이터베이스나 파일 공유와 다른 서비스에서 gMSA에 액세스하도록 허용합니다.
4. [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools)의 CredentialSpec PowerShell 모듈을 사용하여 gMSA를 사용하는 데 필요한 설정을 저장합니다.
5. 추가 옵션을 사용하여 컨테이너를 시작합니다. `--security-opt "credentialspec=..."`

컨테이너가 시작되면 로컬 시스템 또는 네트워크 서비스로 실행되는 설치된 시스템이 gMSA로 실행되는 것으로 표시됩니다. 이는 이러한 계정을 도메인에 가입된 호스트에서 사용하는 방법과 동일하지만, 컴퓨터 계정 대신 gMSA를 사용하는 점만 다릅니다. 

![다이어그램 - 서비스 계정](media/serviceaccount_diagram.png)


# 사용 예


## SQL 연결 문자열
서비스가 컨테이너에서 로컬 시스템 또는 네트워크 서비스로 실행되는 경우 Windows 통합 인증을 사용하여 Microsoft SQL Server에 연결할 수 있습니다.

예제:

```none
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Microsoft SQL Server에서 도메인과 gMSA 이름 다음에 $를 사용하여 로그인을 만듭니다. 만든 로그인을 데이터베이스의 사용자에 추가하고 적절한 액세스 권한을 부여할 수 있습니다.

예제: 

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

작동을 확인하려면 Microsoft Ignite 2016의 "Walk the Path to Containerization - transforming workloads into containers"(컨테이너화 안내 - 워크로드를 컨테이너로 변환) 세션에서 사용 가능한 [녹화된 데모](https://youtu.be/cZHPz80I-3s?t=2672)를 참조하세요.
