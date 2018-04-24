# <a name="host-gateway-mode"></a>호스트 게스트웨이 모드 #
Kubernetes 네트워킹에서 사용 가능한 옵션 중 하나는 모든 노드에서 포드 서브넷 간 정적 경로를 구성해야 하는 *호스트 게이트웨이 모드*입니다.


## <a name="configuring-static-routes--linux"></a>정적 경로 구성 | Linux ##
이 경우 `iptables`를 사용합니다. `$CLUSTER_PREFIX` 변수를 모든 포드가 사용할 줄임 서브넷으로 교체(또는 설정)합니다.

```bash
CLUSTER_PREFIX="192.168"
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING ! -d $CLUSTER_PREFIX.0.0/16 \
              -m addrtype ! --dst-type LOCAL -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1
```

이렇게 하면 포드에 대해 기본 NATing만 설정됩니다. 이제 포드로 향하는 모든 트래픽이 주 인터페이스를 통하도록 만들어야 합니다. 다시 필요에 따라 `$CLUSTER_PREFIX` 변수를 교체하고 적용 가능한 경우 `eth0`을 교체합니다.

```bash
sudo route add -net $CLUSTER_PREFIX.0.0 netmask 255.255.0.0 dev eth0
```

마지막으로 **노드별**로 다음 홉 게이트웨이를 추가합니다. 예를 들어 `192.168.1.0/16`에서 첫 번째 노드가 Windows 노드인 경우:

```bash
sudo route add -net $CLUSTER_PREFIX.1.0 netmask 255.255.255.0 gw $CLUSTER_PREFIX.1.2 dev eth0
```

클러스터의 모든 노드*에 대해* 클러스터의 모든 노드*에* 유사한 경로를 추가해야 합니다.


<a name="explanation-2-suffix"></a>
> [!Important]  
> Windows 노드**만**인 경우 게이트웨이는 서브넷의 `.2`입니다. Linux의 경우 항상 `.1`일 확률이 높습니다. 이 변칙은 `.1` 주소가 호스트 네트워크와 가상 포드 네트워크를 브리징하는 네트워크 어댑터에 대한 게이트웨이로 예약되어 있기 때문입니다.


## <a name="configuring-static-routes--windows"></a>정적 경로 구성 | Windows ##
이 경우 `New-NetRoute`를 사용합니다. [이 리포지토리](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/AddRoutes.ps1)에 사용할 수 있는 자동화된 스크립드 `AddRoutes.ps1`이 있습니다. *Linux 마스터*의 IP 주소를 알아야 합니다.

```powershell
$url = "https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/windows/AddRoutes.ps1"
Invoke-WebRequest $url -o AddRoutes.ps1
./AddRoutes.ps1 -MasterIp 10.1.2.3
```
