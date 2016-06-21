---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
---

# Windows Server 컨테이너 관리

<g id="1" ctype="x-strong">이 예비 콘텐츠는 변경될 수 있습니다.</g>

컨테이너 수명 주기에는 컨테이너 시작, 중지 및 제거 등과 같은 작업이 포함됩니다. 이러한 작업을 수행할 때는 컨테이너 이미지 목록을 검색하고, 컨테이너 네트워킹을 관리하며 컨테이너 리소스를 제한해야 할 수도 있습니다. 이 문서에서는 Docker를 사용한 기본 컨테이너 관리 작업에 대해 자세히 설명하고 세부 정보 문서에 대한 링크도 제공합니다.

## 컨테이너 관리

### 컨테이너 만들기

<g id="2" ctype="x-code">docker run</g>을 사용하여 Docker가 있는 컨테이너를 만듭니다.

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Docker <g id="2" ctype="x-code">run</g> 명령에 대한 자세한 내용은 <g id="4CapsExtId1" ctype="x-link"><g id="4CapsExtId2" ctype="x-linkText">Docker run reference(Docker run 참조)</g><g id="4CapsExtId3" ctype="x-title"></g></g>를 참조하세요.

### 컨테이너 중지

<g id="2" ctype="x-code">docker stop</g> 명령을 사용하여 Docker가 있는 컨테이너를 중지합니다.

```none
PS C:\> docker stop tender_panini

tender_panini
```

이 예제에서는 Docker를 통해 실행 중인 모든 컨테이너를 중지합니다.

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### 컨테이너 제거

Docker가 있는 컨테이너를 제거하려면 <g id="2" ctype="x-code">docker rm</g> 명령을 사용합니다.

```none
PS C:\> docker rm prickly_pike

prickly_pike
```

Docker가 있는 모든 컨테이너를 제거합니다.

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

Docker rm 명령에 대한 자세한 내용은 <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Docker rm 참조</g><g id="2CapsExtId3" ctype="x-title"></g></g>를 참조하세요.






<!--HONumber=Apr16_HO5-->


