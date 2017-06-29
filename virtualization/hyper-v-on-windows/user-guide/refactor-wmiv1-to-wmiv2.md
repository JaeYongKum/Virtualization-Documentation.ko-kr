---
title: "Hyper-V WMIv1을 WMIv2로 이식"
description: "Hyper-V WMIv1을 WMIv2로 이식하는 방법을 알아보세요."
keywords: windows 10, hyper-v, WMIv1, WMIv2, WMI, Msvm_VirtualSystemGlobalSettingData, root\virtualization
author: scooley
ms.date: 04/13/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: b13a3594-d168-448b-b0a1-7d77153759a8
ms.openlocfilehash: e2d6faabe77346199a5d292fcfd92cdfd63909b8
ms.sourcegitcommit: a424c11258e47f224e14c4349b852b9e37b7604f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/10/2017
---
# <a name="move-from-hyper-v-wmi-v1-to-wmi-v2"></a>Hyper-V WMI v1에서 WMI v2로 이동

WMI(Windows Management Instrumentation)는 기본 Hyper-V 관리자 및 Hyper-V PowerShell cmdlet의 관리 인터페이스입니다.  대부분의 사람들이 PowerShell cmdlet 또는 Hyper-V 관리자를 사용하지만 일부 개발자는 WMI를 직접 사용해야 합니다.  

두 가지 Hyper-V WMI 네임스페이스(또는 Hyper-V WMI API 버전)가 있습니다.
* Windows Server 2008에서 도입되어 Windows Server 2012까지 제공된 WMI v1 네임스페이스(root\virtualization)
* Windows Server 2012에서 도입된 WMI v2 네임스페이스(root\virtualization\v2)

이 문서는 이전 WMI 네임스페이스와 통신하는 코드를 새 네임스페이스로 변환하는 것과 관련된 리소스 참조를 포함하고 있습니다.  이 문서는 처음에는 API 정보 및 샘플 코드의 리포지토리/Hyper-V WMI API를 사용하는 프로그램 또는 스크립트를 v1 네임스페이스에서 v2 네임스페이스로 이식하는 데 사용할 수 있는 스크립트 역할을 할 것입니다.

## <a name="msdn-samples"></a>MSDN 샘플

[Hyper-V 가상 컴퓨터 마이그레이션 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-machine-aef356ee)  
[Hyper-V 가상 파이버 채널 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-Fiber-35d27dcd)  
[Hyper-V 계획된 가상 컴퓨터 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-planned-virtual-8c7b7499)  
[Hyper-V 응용 프로그램 상태 모니터링 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-application-health-dc0294f2)  
[가상 하드 디스크 관리 샘플](http://code.msdn.microsoft.com/windowsdesktop/Virtual-hard-disk-03108ed3)  
[Hyper-V 복제 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-replication-sample-d2558867)  
[Hyper-V 메트릭 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-metrics-sample-2dab2cb1)  
[Hyper-V 동적 메모리 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-dynamic-memory-9b0b1d05)  
[Hyper-V 확장 가능 스위치 확장 필터 드라이버](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-Extensible-Virtual-e4b31fbb)  
[Hyper-V 네트워킹 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-networking-sample-7c47e6f5)  
[Hyper-V 리소스 풀 관리 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-resource-pool-df906d95)  
[Hyper-V 복구 스냅숏 샘플](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-recovery-snapshot-ea72320c)  

## <a name="samples-from-blogs"></a>블로그의 샘플

[Hyper-V WMI V2 네임스페이스를 사용하여 VM에 네트워크 어댑터 추가](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/adding-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-V WMI V2 네임스페이스를 사용하여 VM 네트워크 어댑터를 스위치에 연결](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/connecting-a-vm-network-adapter-to-a-switch-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-V WMI V2 네임스페이스를 사용하여 NIC의 MAC 주소 변경](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/changing-the-mac-address-of-nic-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-V WMI V2 네임스페이스를 사용하여 VM에서 네트워크 어댑터 제거](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-V WMI V2 네임스페이스를 사용하여 VHD를 VM에 연결](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/attaching-a-vhd-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-V WMI V2 네임스페이스를 사용하여 VM에서 VHD 제거](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-vhd-from-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Hyper-V WMI V2 네임스페이스를 사용하여 VM 만들기](http://blogs.msdn.com/b/virtual_pc_guy/archive/2013/06/20/creating-a-virtual-machine-with-wmi-v2.aspx)

