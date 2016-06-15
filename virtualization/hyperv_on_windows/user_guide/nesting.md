---
title: 중첩된 가상화
description: 중첩된 가상화
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
---

# 중첩된 가상화

중첩된 가상화를 사용하면 Hyper-V 호스트를 가상화된 환경에서 실행할 수 있습니다. 즉, 중첩된 가상화를 통해 Hyper-V 호스트 자체를 가상화할 수 있습니다. 중첩된 가상화의 몇 가지 사용 사례로 가상화된 환경에서 Hyper-V 랩 실행, 개별 하드웨어 없이 다른 사용자에게 가상 서비스 제공 등을 들 수 있으며 Windows 컨테이너 기술에서는 Hyper-V 컨테이너를 가상화된 컨테이너 호스트에서 실행할 때 중첩된 가상화를 사용합니다. 이 문서에서는 소프트웨어 및 하드웨어 필수 조건 및 구성 단계를 자세히 설명하고 문제 해결 정보를 제공합니다.

> 중첩된 가상화는 시험판 릴리스이므로 프로덕션에서 사용할 수 없습니다.

## 필수 구성 요소

- 빌드 10565 이상을 실행 중인 Windows 참가자 빌드(Windows Server 2016, Nano 서버 또는 Windows 10).
- 두 하이퍼바이저(부모 및 자식)가 모두 동일한 Windows 빌드(10565 이상)를 실행하고 있어야 합니다.
- 4GB RAM을 최소값으로 사용할 수 있습니다.
- Intel VT-x 기술을 사용하는 Intel 프로세서

## 중첩된 가상화 구성

먼저 호스트와 동일한 빌드를 실행하는 가상 컴퓨터를 만듭니다. **가상 컴퓨터를 켜지 마세요**. 자세한 내용은 [가상 컴퓨터 만들기](../quick_start/walkthrough_create_vm.md)를 참조하세요.

가상 컴퓨터를 만들었으면 부모 하이퍼바이저에서 다음 명령을 실행합니다. 그러면 가상 컴퓨터에서 중첩된 가상화를 사용할 수 있습니다.

```none
Set-VMProcessor -VMName <virtual machine> -ExposeVirtualizationExtensions $true -Count 2
```

중첩된 Hyper-V 호스트를 실행할 때는 가상 컴퓨터에서 동적 메모리를 사용하지 않도록 설정해야 합니다. 이 설정은 가상 컴퓨터의 속성에서 하거나 다음 PowerShell 명령을 사용하여 구성할 수 있습니다.

```none
Set-VMMemory <virtual machine> -DynamicMemoryEnabled $false
```

중첩된 가상 컴퓨터에서 IP 주소를 수신하려면, MAC 주소 스푸핑을 사용하도록 설정해야 합니다. 이 작업은 다음 PowerShell 명령으로 수행합니다.

```none
Get-VMNetworkAdapter -VMName <virtual machine> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

이러한 단계를 완료했으면 가상 컴퓨터를 시작하고 Hyper-V를 설치할 수 있습니다. Hyper-V 설치에 대한 자세한 내용은 [Hyper-V 설치]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)를 참조하세요.

## 구성 스크립트

필요에 따라 다음 스크립트 사용하여 중첩된 가상화를 사용하도록 설정하고 구성할 수 있습니다. 스크립트를 다운로드하고 실행하는 명령은 다음과 같습니다.
  
```none
# download script
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Enable-NestedVm.ps1 -OutFile .\Enable-NestedVm.ps1 

# run script
.\Enable-NestedVm.ps1 -VmName "DemoVM"
```

## 알려진 문제

- 장치 가드가 활성화된 호스트는 가상화 확장을 게스트에 공개할 수 없습니다.
- 가상화 기반 보안(VBS)이 활성화된 호스트는 가상화 확장을 게스트에 공개할 수 없습니다. 중첩된 가상화를 사용하려면 먼저 VBS를 사용하지 않도록 설정해야 합니다.
- 빈 암호를 사용할 경우 가상 컴퓨터 연결이 끊길 수 있습니다. 암호를 변경하고 문제를 해결해야 합니다.
- 중첩된 가상화가 가상 컴퓨터에서 활성화되면 다음과 같은 기능을 해당 VM과 더 이상 호환할 수 없습니다.  
  * 런타임 메모리 크기 조정
  * 실행 중인 가상 컴퓨터에 검사점 적용
  * 다른 가상 컴퓨터를 호스트하는 가상 컴퓨터를 실시간 마이그레이션할 수 없습니다.
  * 저장 및 복원이 작동하지 않습니다.

## FAQ 및 문제 해결

가상 컴퓨터가 시작되지 않으면 어떻게 해야 합니까?

1. 동적 메모리가 OFF인지 확인합니다.
2. 관리자 권한 프롬프트로 호스트 컴퓨터에서 [이 PowerShell 스크립트](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1)를 실행합니다.
  
## 사용자 의견

Windows 피드백 앱, [가상화 포럼](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv) 또는 [GitHub](https://github.com/Microsoft/Virtualization-Documentation)를 통해 추가 문제를 보고하세요.



<!--HONumber=Jun16_HO2-->


