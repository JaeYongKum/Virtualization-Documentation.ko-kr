---
title: "Windows Docker 호스트의 원격 관리"
description: "Windows Server를 실행하는 원격 Docker 호스트를 안전하게 관리하는 방법"
keywords: "Docker, 컨테이너"
author: taylorb-microsoft
ms.date: 02/14/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 0cc1b621-1a92-4512-8716-956d7a8fe495
ms.openlocfilehash: 75d19646dd41a4f73dfb9cdd09808b61fba8e4ab
ms.sourcegitcommit: 1c7e94089646f3db31e033f0909a10ce5077d05e
ms.translationtype: HT
ms.contentlocale: ko-KR
---
# <a name="remote-management-of-a-windows-docker-host"></a>Windows Docker 호스트의 원격 관리

`docker-machine`이 없더라도 원격으로 액세스할 수 있는 Docker 호스트를 Windows Server 2016 VM에 만들 수 있습니다.

단계는 매우 간단합니다.

* [dockertls](https://hub.docker.com/r/stefanscherer/dockertls-windows/)를 사용하여 서버에 인증서를 만듭니다. IP 주소로 인증서를 만들 경우 IP 주소가 바뀔 때 인증서를 다시 만들 필요가 없도록 정적 IP 사용을 고려할 수 있습니다.

* Docker 서비스 다시 시작 `Restart-Service Docker`
* 인바운드 트래픽을 허용하는 NSG 규칙을 만들어 포트 Docker의 TLS 포트 2375 및 2376을 사용할 수 있게 합니다. 보안 연결의 경우 2376만 허용해야 합니다.  
  포털에 NSG 구성이 다음과 같이 표시되어야 합니다.  
  ![NGS](media/nsg.png)  
  
* Windows 방화벽을 통해 인바운드 연결을 허용합니다. 
```
New-NetFirewallRule -DisplayName 'Docker SSL Inbound' -Profile @('Domain', 'Public', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2376
```
* 컴퓨터에 있는 사용자의 Docker 폴더(예: `c:\users\chris\.docker`)에서 로컬 컴퓨터로 `ca.pem`, 'cert.pem' 및 'key.pem' 파일을 복사합니다. 예를 들어, RDP 세션에서 파일을 복사하여 붙여 넣을 수 있습니다(Ctrl+C, Ctrl+V). 
* 원격 Docker 호스트에 연결할 수 있는지 확인합니다. 실행
```
docker -D -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify --tlscacert=c:\
users\foo\.docker\client\ca.pem --tlscert=c:\users\foo\.docker\client\cert.pem --tlskey=c:\users\foo\.doc
ker\client\key.pem ps
```


## <a name="troubleshooting"></a>문제 해결
### <a name="try-connecting-without-tls-to-determine-your-nsg-firewall-settings-are-correct"></a>TLS 없이 연결을 시도하여 NSG 방화벽 설정이 올바른지 확인합니다.
연결 오류는 대개 오류에서 다음과 같이 나타납니다.
```
error during connect: Get https://wsdockerhost.southcentralus.cloudapp.azure.com:2376/v1.25/version: dial tcp 13.85.27.177:2376: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

다음을 
```
{
    "tlsverify":  false,
}
```
`c"\programdata\docker\config\daemon.json`에 추가하여 암호화되지 않은 연결을 허용한 다음 서비스를 다시 시작합니다.

다음과 같이 명령줄로 원격 호스트에 연결합니다.
```
docker -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify=0 version
```

### <a name="cert-problems"></a>인증서 문제
IP 주소나 DNS 이름에 대한 인증서를 만들지 않고 Docker 호스트에 액세스하면 다음과 같은 오류가 발생합니다.
```
error during connect: Get https://w.x.y.c.z:2376/v1.25/containers/json: x509: certificate is valid for 127.0.0.1, a.b.c.d, not w.x.y.z
```
w.x.y.z가 호스트의 공용 IP에 대한 DNS 이름이고, DNS 이름이 인증서의 [일반 이름](https://www.ssl.com/faqs/common-name/)(`SERVER_NAME` 환경 변수) 또는 dockertls에 제공된 `IP_ADDRESSES` 변수의 IP 주소 중 하나와 일치하는지 확인합니다.

### <a name="cryptox509-warning"></a>crypto/x509 경고
경고가 발생할 수 있습니다. 
```
level=warning msg="Unable to use system certificate pool: crypto/x509: system root pool is not available on Windows"
```
경고는 무해합니다.
