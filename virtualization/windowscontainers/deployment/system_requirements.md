# Windows의 컨테이너 요구 사항

**이 예비 콘텐츠는 변경될 수 있습니다.**

이 가이드는 Windows 컨테이너 호스트에 대한 요구 사항을 나열합니다.

## 지원되는 OS 이미지

Windows Server 기술 미리 보기 4는 두 개의 컨테이너 OS 이미지, Windows Server Core 및 Nano Server를 통해 제공됩니다. 일부 구성은 두 개의 OS 이미지를 모두 지원하지 않습니다. 이 테이블은 지원되는 구성을 자세히 설명합니다.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>호스트 운영 체제</center></th>
<th><center>Windows Server 컨테이너</center></th>
<th><center>Hyper-V 컨테이너</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 전체 UI</center></td>
<td><center>Core OS 이미지</center></td>
<td><center>Nano OS 이미지</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Core OS 이미지</center></td>
<td><center> Nano OS 이미지</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Nano OS 이미지</center></td>
<td><center>Nano OS 이미지</center></td>
</tr>
</tbody>
</table>

## Hyper-V 컨테이너 요구 사항

Windows 컨테이너 호스트가 Hyper-V 가상 컴퓨터에서 실행되며 Hyper-V 호스트 컨테이너를 호스팅할 경우 중첩된 가상화를 사용해야 합니다 중첩된 가상화에는 다음과 같은 요구 사항이 있습니다.

- 가상화된 Hyper-V 호스트에 사용할 수 있는 4GB 이상의 RAM.
- 실제 및 가상화된 호스트 모두에 Windows Server 2016 기술 미리 보기 4 또는 Windows 10 빌드 10565.
- Intel VT-x가 포함된 프로세서.(이 기능은 현재 Intel 프로세서에 대해서만 사용할 수 있습니다)
- 또한 컨테이너 호스트 VM에는 적어도 2개의 가상 프로세서가 필요합니다.





<!--HONumber=Jan16_HO1-->
