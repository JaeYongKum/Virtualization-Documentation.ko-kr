# 관리 상호 운용성

**이 예비 콘텐츠는 변경될 수 있습니다.**

대부분의 경우 PowerShell을 통해 만든 Windows 컨테이너는 PowerShell로 관리해야 하며 Docker로 만든 컨테이너는 Docker로 관리합니다. 즉 호스트 컴퓨팅 PowerShell 모듈은 만든 방법에 관계없이 컨테이너의 **실행**을 확인 및 중지하는 기능을 제공합니다. 이 모듈은 컨테이너 호스트에서 실행되는 컨테이너의 '작업 관리자' 같은 역할을 합니다.

## 모든 컨테이너 표시

컨테이너 목록을 반환하려면 `Get-ComputeProcess` 명령을 사용합니다.

```powershell
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## 컨테이너 중지

만든 방법이 PowerShell 또는 Docker이든 관계없이 컨테이너를 중지하려면 `Stop-ComputeProcess` 명령을 사용합니다.

> 작성 시점 현재 `Get-Container` 명령을 사용할 때 컨테이너가 중지된 것으로 표시되게 하려면 VMMS 서비스를 다시 시작해야 합니다.

```powershell
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```




