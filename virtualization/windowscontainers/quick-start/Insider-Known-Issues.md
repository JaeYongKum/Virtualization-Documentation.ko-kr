# 참가자 빌드에 대해 알려진 문제

## 빌드 16237

- Hyper-V 컨테이너가 제대로 작동하지 않습니다. 빌드 16237에서 Hyper-V 컨테이너를 사용하려면 이 해결 방법이 필요합니다. PowerShell에서 다음 명령을 실행합니다.

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- 이제 Nano 서버가 사용자로 실행되므로 관리자 권한이 필요한 명령은 실패합니다. "RUN setx /M PATH" 같은 줄을 포함하면 빌드가 실패합니다. 이 경우 다음과 같은 대안을 사용할 수 있습니다.

```dockerfile
RUN setx PATH <path>
```
