---
title: "Windows 컨테이너 설명서"
description: "Windows 컨테이너 설명서"
keywords: "Docker, 컨테이너"
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 74c9d604-0915-4d89-bc69-0263b76bc66b
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 2c9821ef7ac414640790b3cfdb7fd457710a67f4

---

# Windows 컨테이너 설명서

Windows 컨테이너는 여러 개의 격리된 응용 프로그램을 단일 시스템에서 실행할 수 있는 운영 체제 수준 가상화를 제공합니다. 두 가지 형식의 컨테이너 런타임이 각각 다른 응용 프로그램 격리 수준을 가진 기능과 함께 포함됩니다. Windows Server 컨테이너는 네임스페이스 및 프로세스 격리를 통해 격리됩니다. Hyper-V 컨테이너는 간단한 가상 컴퓨터에서 각 컨테이너를 캡슐화합니다. 이 문서 집합은 관리 작업에서 빠른 시작 가이드, 배포 가이드 및 기술 세부 정보를 제공합니다.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:90%" cellpadding="25" cellspacing="5">
<tr>
<td ><center>![](media/try.png)</center></td>
<td>**빠른 시작**<br /><br />
다음 빠른 시작 가이드를 사용하여 Windows Server 및 Hyper-V 컨테이너를 사용해 봅니다.<br /><br />
<ul>
<li>[1 - 개념 및 용어](quick_start/quick_start.md)<br /><br /></li>
<li>[2 - Windows Server의 컨테이너](quick_start/quick_start_windows_server.md)<br /><br /></li>
<li>[3 - Windows Server의 컨테이너 이미지](quick_start/quick_start_images.md)<br /><br /></li>
<li>[4 - Windows 10의 컨테이너](quick_start/quick_start_windows_10.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td ><center>![](media/1.png)</center></td>
<td>**배포**<br /><br />
Windows Server 2016 및 Nano Server에서 Windows 컨테이너를 배포하는 방법을 알아봅니다.<br /><br />
<ul>
<li>[시스템 요구 사항](deployment/system_requirements.md)<br /><br /></li>
<li>[컨테이너 호스트 배포 - Windows Server](deployment/deployment.md)<br /><br /></li>
<li>[컨테이너 호스트 배포 - Nano Server](deployment/deployment_nano.md)<br /><br /></li>
<li>[바이러스 백신 최적화](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/explore.png)</center></td>
<td>**Windows의 Docker**<br /><br />
Windows에서 Docker를 관리하는 방법을 알아보세요.<br /><br />
<ul>
<li>[Windows의 Docker 엔진](docker/configure_docker_daemon.md)<br /><br /></li>
<li>[Windows의 Dockerfile](docker/manage_windows_dockerfile.md)<br /><br /></li>
<li>[컨테이너 데이터 관리](management/manage_data.md)<br /><br /></li>
<li>[Dockerfile 최적화](docker/optimize_windows_dockerfile.md)<br /><br /></li>
<li>[컨테이너 네트워킹](management/container_networking.md)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/video.png)</center></td>
<td>**보기**<br /><br />
Windows 컨테이너 팀의 데모 및 인터뷰에 관심이 있으십니까?<br /><br />
<ul>
<li>[컨테이너 채널](https://channel9.msdn.com/Blogs/containers)</li>
</ul>
<br />
</td>
</tr>

<tr>
<td ><center>![](media/question.png)</center></td>
<td>**커뮤니티**<br /><br />
커뮤니티와 상호 작용하고 샘플을 사용해 보며 추가 리소스를 찾습니다.<br /><br />
<ul>
<li>[컨테이너 포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)<br /><br /></li>
<li>[컨테이너 리소스](https://msdn.microsoft.com/virtualization/community/community_overview)<br /><br /></li>
</ul>
</td>
</tr>
</table>



<!--HONumber=Sep16_HO2-->


