# Windows 컨테이너에서의 응용 프로그램 호환성

미리 보기입니다. Windows에서 실행되는 응용 프로그램은 결과적으로 컨테이너에서도 실행되어야 하지만 이 시점에서 최신 응용 프로그램 호환성 상태를 확인해 보는 것이 좋겠습니다.

이 설명서는 오직 경험을 공유할 목적으로 제공됩니다.

이 목록에 없는 항목이 있다면 [포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)을 통해 해당 환경에서의 실패 및 성공 경험을 알려 주세요.

## Windows Server 컨테이너

Windows Server 컨테이너에서 다음과 같은 응용 프로그램의 실행을 테스트해 보았습니다. 이 결과가 해당 응용 프로그램이 반드시 제대로 작동한다고는 보장하지 않습니다.

| **이름**| **버전**| **Windows Server Core 기본 이미지**| **Nano Server 기본 이미지**| **설명**|
|:-----|:-----|:-----|:-----|:-----|
| .NET| 3.5| 예| Unknown| |
| .NET| 4.6| 예| Unknown| |
| .NET CLR| 5 beta 6| 예| 예| x64 및 x86 모두|
| Active Python| 3.4.1| 예| 예| |
| Apache Cassandra| | 예| Unkown|
| Apache CouchDB| 1.6.1| 아니요| 아니요| |
| Apache Hadoop| | 예| 아니요| |
| Apache HTTPD| 2.4| 예| 예| 중복 제거 필터를 로드하면 VC++ 런타임이 설치되지 않습니다.`fltmc 언로드 중복 제거`를 사용하여 중복 제거 언로드|
| Apache Tomcat| 8.0.24 x64| 예| Unknown| |
| ASP.NET| 4.6| 예| Unkown| |
| ASP.NET| 5 beta 6| 예| 예| x64 및 x86 모두|
| Django| | 예| 예| |
| Go| 1.4.2| 예| 예| |
| IIS(인터넷 정보 서비스) | 10.0| 예| 예| 중복 제거 필터를 로드하면 VC++ 런타임이 설치되지 않습니다.`fltmc 언로드 중복 제거`를 사용하여 중복 제거 언로드|
| Java| 1.8.0_51| 예| 예| 서버 버전을 사용합니다.클라이언트 버전이 제대로 설치되지 않습니다.|
| MongoDB| 3.0.4| 예| Unkown| |
| MySQL| 5.6.26| 예| 예| |
| NGinx| 1.9.3| 예| 예| |
| Node.js| 0.12.6| 부분적으로| 부분적으로| NPM이 패키지 다운로드에 실패합니다.|
| Perl| | 예| Unkown| |
| PHP| 5.6.11| 예| 예| 중복 제거 필터를 로드하면 VC++ 런타임이 설치되지 않습니다.`fltmc 언로드 중복 제거`를 사용하여 중복 제거 언로드|
| PostgreSQL| 9.4.4| 예| Unknown| 중복 제거 필터를 로드하면 VC++ 런타임이 설치되지 않습니다.`fltmc 언로드 중복 제거`를 사용하여 중복 제거 언로드|
| Python| 3.4.3| 예| 예| |
| R| 3.2.1| 아니요| 아니요| |
| RabbitMQ| 3.5.x| 예| Unknown| |
| Redis| 2.8.2101| 예| 예| |
| Ruby| 2.2.2| 예| 예| x64 및 x86 모두|
| Ruby on Rails| 4.2.3| 예| 예| |
| SQLite| 3.8.11.1| 예| 아니요| |
| SQL Server Express| 2014 LocalDB| 아니요| 아니요| |
| Sysinternals 도구| *| 예| 예| GUI를 필요로 하지 않는 항목만 테스트했습니다.현재 설계에서는 PsExec가 작동하지 않습니다.|

## Hyper-V 컨테이너

Hyper-V 컨테이너에서 다음과 같은 응용 프로그램의 실행을 테스트해 보았습니다. 이 결과가 해당 응용 프로그램이 반드시 제대로 작동한다고는 보장하지 않습니다.

| **이름**| **버전**| **Nano Server 기본 이미지**| **설명**|
|:-----|:-----|:-----|:-----|
| Apache Hadoop| | 아니요| |
| Apache HTTPD| 2.4| 예| 중복 제거 필터를 로드하면 VC++ 런타임이 설치되지 않습니다.`fltmc 언로드 중복 제거`를 사용하여 중복 제거 언로드|
| ASP.NET| 5 beta 6| 예| x64 및 x86 모두|
| Django| | 예| 이미지가 DockerFile로 만들어졌고 Python 바이너리가 그 일부로 복사된 경우 Python이 작동하지 않습니다.컨테이너를 시작한 다음 Python 바이너리를 복사합니다.|
| Go| 1.4.2| 예| |
| IIS(인터넷 정보 서비스) | 10.0| 예| IIS는 dism을 직접 설치하지 않습니다.dism 명령을 사용하여 IIS 무인 설치를 수행합니다.|
| Java| 1.8.0_51| 예| 서버 버전을 사용합니다.클라이언트 버전이 제대로 설치되지 않습니다.|
| MySQL| 5.6.26| 예| |
| NGinx| 1.9.3| 예| |
| Node.js| 0.12.6| 부분적으로| NPM이 패키지 다운로드에 실패합니다.|
| Python| 3.4.3| 예| 이미지가 DockerFile로 만들어졌고 Python 바이너리가 그 일부로 복사된 경우 Python이 작동하지 않습니다.컨테이너를 시작한 다음 Python 바이너리를 복사합니다.|
| Redis| 2.8.2101| 예| |
| Ruby| 2.2.2| 예| x64 및 x86 모두|
| Ruby on Rails| 4.2.3| 예| |
| Sysinternals 도구| | 예| GUI를 필요로 하지 않는 항목만 테스트했습니다.현재 설계에서는 PsExec가 작동하지 않습니다.|

## 내 경험 입력하기

이 목록에 없는 항목이 있다면 [포럼](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)을 통해 해당 환경에서의 실패 및 성공 경험을 알려 주세요.




