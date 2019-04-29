# <a name="known-issues-for-insider-builds"></a>참가자 빌드에 대 한 알려진된 문제

## <a name="build-16237"></a>빌드 16237

- Hyper-v 격리 제대로 작동 하지 않습니다. 빌드 16237에서에서 Hyper-v 격리를 사용 하려면이 해결 방법이 필요 합니다. PowerShell에서 다음 명령을 실행합니다.

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano 서버는 이제 사용자로 실행 되므로 관리자 권한이 필요한 명령은 실패 합니다. "RUN setx /M PATH" 같은 줄을 포함하면 빌드가 실패합니다. 이 경우 다음과 같은 대안을 사용할 수 있습니다.

```dockerfile
RUN setx PATH <path>
```
