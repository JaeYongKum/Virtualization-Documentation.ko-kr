# 문제 해결

컴퓨터를 설정하거나 컨테이너를 실행하는 데 문제가 있나요? 일반적인 문제를 확인하는 PowerShell 스크립트를 만들었습니다. 이 스크립트를 먼저 사용해 보고 찾는 내용과 결과를 공유해 주세요.

```PowerShell
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/windows-server-container-tools/Debug-ContainerHost/Debug-ContainerHost.ps1 | Invoke-Expression
```
일반적인 솔루션과 함께 실행하는 모든 테스트 목록은 스크립트의 [Readme file](https://github.com/Microsoft/Virtualization-Documentation/blob/master/windows-server-container-tools/Debug-ContainerHost/README.md)(추가 정보 파일)에 있습니다.

문제의 원인을 찾는 데 도움이 되지 않는 경우 스크립트 출력을 [컨테이너 포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)에 게시하세요. 이 포럼은 Windows 참가자와 개발자를 비롯한 커뮤니티의 도움을 받을 수 있는 유용한 곳입니다.


## 로그 찾기
Windows 컨테이너를 관리하기 위해 여러 서비스가 사용됩니다. 다음 섹션에서는 각 서비스에 대한 로그를 확인할 수 있는 위치를 보여 줍니다.

### Docker 엔진
Docker 엔진은 파일 대신 Windows '응용 프로그램' 이벤트 로그에 기록합니다. 이러한 로그는 Windows PowerShell을 사용하여 쉽게 읽고 정렬하고 필터링할 수 있습니다.

예를 들어 가장 오래된 순서부터 시작하여 최근 5분의 Docker 엔진 로그가 표시됩니다.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

또한 다른 도구 또는 스프레드시트로 읽을 수 있도록 CSV 파일로 쉽게 파이프할 수 있습니다.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### 디버그 로깅 사용
Docker 엔진에서 디버그 수준 로깅을 사용하도록 설정할 수도 있습니다. 그러면 일반 로그에 정보가 부족한 경우의 문제 해결에 유용할 수 있습니다.

먼저 관리자 권한 명령 프롬프트를 연 다음 `sc.exe qc docker`를 실행하여 Docker 서비스에 대한 현재 명령줄을 가져옵니다.
예:
```none
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

현재 `BINARY_PATH_NAME`을 다음과 같이 수정합니다.
- 끝에 -D 추가
- 각 "를 \로 이스케이프
- 전체 명령 "로 묶기

그런 다음 새 문자열 다음에 오는 `sc.exe config docker binpath= `을(를) 실행합니다. 예를 들면 다음과 같습니다. 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


이제 Docker 서비스를 다시 시작합니다.
```none
sc.exe stop docker
sc.exe start docker
```

이렇게 하면 응용 프로그램 이벤트 로그에 더 많은 정보가 기록되므로 문제 해결을 마치면 `-D` 옵션을 제거하는 것이 좋습니다. 위의 단계를 `-D` 없이 수행하고 서비스를 다시 시작하여 디버그 로깅을 사용하지 않도록 설정합니다.


### 호스트 컨테이너 서비스
Docker 엔진은 Windows 관련 호스트 컨테이너 서비스를 사용합니다. 이 서비스에는 다음과 같은 별도의 로그가 있습니다. 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

이 로그는 이벤트 뷰어에서 볼 수 있고 PowerShell에서 쿼리할 수도 있습니다.

예를 들면 다음과 같습니다.
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```



<!--HONumber=Oct16_HO3-->


