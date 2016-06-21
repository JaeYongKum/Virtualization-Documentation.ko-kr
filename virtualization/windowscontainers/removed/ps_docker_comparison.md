---
author: scooley
redirect_url: ../quick_start/manage_docker
---


# Windows 컨테이너 관리를 위한 PowerShell과 Docker 비교

기본 Windows 도구(PowerShell,이 미리 보기 내)와 Docker와 같은 오픈 소스 관리 도구를 사용하여 Windows 컨테이너를 관리하는 방법은 여러 가지가 있습니다.  
개별적으로 사용할 수 있는 모든 개요 가이드:
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Docker를 사용하여 Windows 컨테이너 관리</g><g id="1CapsExtId3" ctype="x-title"></g></g>
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">PowerShell을 사용하여 Windows 컨테이너 관리</g><g id="1CapsExtId3" ctype="x-title"></g></g>

이 페이지는 Docker 도구 및 PowerShell 관리 도구를 더욱 자세히 비교합니다.

## 컨테이너에 대한 PowerShell과 Hyper-V VM

PowerShell cmdlet을 사용하여 Windows 컨테이너를 만들고 실행하고 상호 작용할 수 있습니다. 실행을 위해 필요한 것은 모두 기본 제공으로 사용할 수 있습니다.

Hyper-V PowerShell을 사용한 경우 cmdlet의 디자인은 친숙해야 합니다. 워크플로의 대부분은 Hyper-V 모듈을 사용하여 가상 컴퓨터를 관리하는 방법과 비슷합니다. <g id="2" ctype="x-code">NEW-VM</g>, <g id="4" ctype="x-code">GET-VM</g>, <g id="6" ctype="x-code">START-VM</g>, <g id="8" ctype="x-code">STOP-VM</g> 대신 <g id="10" ctype="x-code">New-Container</g>, <g id="12" ctype="x-code">Get-Container</g>, <g id="14" ctype="x-code">Start-Container</g>, <g id="16" ctype="x-code">Stop-Container</g>가 있습니다. 몇 가지 컨테이너별 cmdlet 및 매개 변수가 있지만 일반적인 수명 주기 및 Windows 컨테이너의 관리는 Hyper-V VM의 것과 유사하게 보입니다.

## Docker와 비교하여 PowerShell 관리는 어떻습니까?

컨테이너 PowerShell cmdlet은 일반적인 규칙으로 Docker의 것과 동일하지 않은 API를 공개하고 cmdlet은 작업에 더 세분화됩니다. 일부 Docker 명령은 PowerShell에서 상당히 간단한 병렬을 가지고 있습니다.

| Docker 명령| PowerShell Cmdlet|
|----|----|
| <g id="1" ctype="x-code">docker ps -a</g>| <g id="1" ctype="x-code">Get-Container</g>|
| <g id="1" ctype="x-code">docker images</g>| <g id="1" ctype="x-code">Get-ContainerImage</g>|
| <g id="1" ctype="x-code">docker rm</g>| <g id="1" ctype="x-code">Remove-Container</g>|
| <g id="1" ctype="x-code">docker rmi</g>| <g id="1" ctype="x-code">Remove-ContainerImage</g>|
| <g id="1" ctype="x-code">docker create</g>| <g id="1" ctype="x-code">New-Container</g>|
| <g id="1" ctype="x-code">docker commit <container ID></g>| <g id="1" ctype="x-code">New-ContainerImage -Container &lt;container&gt;</g>|
| <g id="1" ctype="x-code">docker load &lt;tarball&gt;</g>| <g id="1" ctype="x-code">Import-ContainerImage <AppX package></g>|
| <g id="1" ctype="x-code">docker save</g>| <g id="1" ctype="x-code">Export-ContainerImage</g>|
| <g id="1" ctype="x-code">docker start</g>| <g id="1" ctype="x-code">Start-Container</g>|
| <g id="1" ctype="x-code">docker stop</g>| <g id="1" ctype="x-code">Stop-Container</g>|

PowerShell cmdlet은 정확하게 완벽한 패리티가 아니며 PowerShell의 대체로 제공하지 않는 상당히 많은 수의 명령이 있습니다*(특히 <g id="2" ctype="x-code">docker build</g> 및 <g id="4" ctype="x-code">docker cp</g>). 하지만 눈에 들어오는 사실은 <g id="2" ctype="x-code">docker run</g>에 대한 단일 한 줄 대체가 없다는 것입니다.

\* 변경될 수 있습니다.

### 그러나 docker run이 필요합니다. 어떻게 진행되고 있나요?

PowerShell을 이미 알고 있는 사용자에 대해 조금 더 친숙한 상호 작용 모델을 제공하기 위해 몇 가지 작업을 진행하고 있습니다. 물론 docker를 작동하는 방식에 익숙한 경우 약간의 시점 전환이 됩니다.

1.  PowerShell 모델의 컨테이너의 수명 주기는 약간 다릅니다. 컨테이너 PowerShell 모듈에서 <g id="2" ctype="x-code">New-Container</g>(중지된 새 컨테이너를 만듦) 및 <g id="4" ctype="x-code">Start-Container</g>의 더욱 세밀한 작업을 공개했습니다.

  컨테이너를 만들고 시작하는 사이에서 TP3에 대한 컨테이너의 설정을 구성할 수도 있으며 공개하려고 계획 중인 다른 구성은 컨테이너에 대한 네트워크 연결을 설정하는 기능입니다. (Add/Remove/Connect/Disconnect/Get/Set)-ContainerNetworkAdapter cmdlet 사용.

2.  현재 시작 시 컨테이너 내에서 실행되도록 명령을 전달할 수 없습니다. 그러나 <g id="2" ctype="x-code">Enter-PSSession -ContainerId <ID of a running container></g>를 사용하여 실행 중인 컨테이너에 대화형 PowerShell 세션을 가져올 수 있으며 <g id="4" ctype="x-code">Invoke-Command -ContainerId <container id> -ScriptBlock { code to run inside the container }</g> 또는 <g id="6" ctype="x-code">Invoke-Command -ContainerId <container id> -FilePath <path to script></g>를 사용하여 실행 중인 컨테이너 내에서 명령을 실행할 수 있습니다.  
이 두 명령은 모두 높은 권한 작업에 대해 선택적 <g id="2" ctype="x-code">-RunAsAdministrator</g> 플래그를 허용합니다.


## 제한 사항 및 알려진 문제

1.  현재 컨테이너 cmdlet은 Docker를 통해 만든 모든 컨테이너 또는 이미지에 대한 지식이 없으며 Docker는 PowerShell을 통해 만든 모든 컨테이너 및 이미지에 대해 모릅니다. Docker에서 만든 경우 Docker로 관리하고 PowerShell을 통해 만든 경우 PowerShell을 통해 관리합니다.

2.  더 나은 오류 메시지, 더 나은 진행률 보고, 잘못된 이벤트 문자열 등과 같은 최종 사용자 환경을 개선하기 위한 몇 가지 작업이 있습니다. 더 자세하거나 더 나은 정보를 얻으려고 하는 문제가 발생하는 경우 포럼에 제안을 보내 주시기 바랍니다.

## 빠른 실행

다음은 몇 가지 일반적인 워크플로를 살펴봅니다.

여기에서는 "ServerDatacenterCore"라는 OS 컨테이너 이미지를 설치하고 "Virtual Switch"(New-vmswitch 사용)라는 가상 스위치를 생성했다고 가정합니다.

``` PowerShell
### 1. Enumerating images
# At this point, you can enumerate the images on the system:
Get-ContainerImage

# Get-ContainerImage also accepts filters.
# For example, this will return all container images whose Name starts with S (case-insensitive):
Get-ContainerImage -Name S*

# You can save the results of this to a variable.
# (If you're not familiar with PowerShell, the "$" denotes a variable.)
$baseImage = Get-ContainerImage -Name ServerDatacenterCore
$baseImage

### 2. Creating and enumerating containers
# Now, we can create a new container using this image:
New-Container -Name "MyContainer" -ContainerImage $baseImage -SwitchName "Virtual Switch"

# Now we can enumerate all containers.
Get-Container

# Similarly, we can save this container to a variable:
$container1 = Get-Container -Name "MyContainer"

### 3. Starting containers, interacting with running containers, and stopping containers
# Now let's go ahead and start the container.
Start-Container -Name "MyContainer"

# (We could've also started this container using "Start-Container -Container $container1".)

# With the container now running, let's go ahead and enter an interactive PowerShell session:
Enter-PSSession -ContainerId $container1.Id

# This should eventually bring up a PowerShell prompt from inside the container.
# You can try all the things that you did in the interactive cmd prompt given by "docker run -it".
# For now, just to prove we've been here, we can create a new file:
cd \
mkdir Test
cd Test
echo "hello world" > hello.txt
exit

# Now we should be back in the outside world. Even though we've exited the PowerShell session,
# the container itself is still running, as you can see by printing out the container again:
$container1

# Before we can commit this container to a new image, we need to stop the container.
# Let's do that now.
Stop-Container -Container $container1

### 4. Creating a new container image
# And now let's commit it to a new image.
$image1 = New-ContainerImage -Container $container1 -Publisher Test -Name Image1 -Version 1.0

# Enumerate all the images again, for sanity's sake:
Get-ContainerImage

# Rinse and repeat! Make another container based on the new image.
$container2 = New-Container -Name "MySecondContainer" -ContainerImage $image1 -SwitchName "Virtual Switch"

# (If you like, you can start the second container and verify that the new file
# "\Test\hello.txt" is there as expected.)

### 5. Removing a container
# The first container we created is now stopped. Let's get rid of it:
Remove-Container -Container $container1

# And confirm that it's gone:
Get-Container

### 6. Exporting, removing, and importing images
# For images that aren't the base OS image, we can export them into an .appx package file.
Export-ContainerImage -Image $image1 -Path "C:\exports"
# This should create a .appx file in the C:\exports folder.
# If you've given your image the same publisher, name, and version we used earlier,
# you'd expect the resulting .appx to be named "CN=Test_Image1_1.0.0.0.appx".

# Before we can try importing the image again, we need to remove the image.
# (If you have any running containers that depend on this image, you'll want to stop them first.)
Remove-ContainerImage -Image $image1

# Now let's import the image again:
Import-ContainerImage -Path C:\exports\CN=Test_Image1_1.0.0.0.appx

# We'd previously created a container dependent on this image. You should be able to start it:
Start-Container -Container $container2 
```

### 사용자 고유의 샘플 빌드

<g id="2" ctype="x-code">Get-Command -Module Containers</g>를 사용하여 모든 컨테이너 cmdlet을 볼 수 있습니다. 여기에서 설명되지 않은 스스로 학습하도록 남겨둔 몇 가지 다른 cmdlet이 있습니다.    
<g id="1" ctype="x-strong">참고</g> 코어 PowerShell의 일부인 <g id="3" ctype="x-code">Enter-PSSession</g> 및 <g id="5" ctype="x-code">Invoke-Command</g> cmdlet을 반환하지 않습니다.

<g id="2" ctype="x-code">Get-Help [cmdlet name]</g>, 또는 동등한 <g id="4" ctype="x-code">[cmdlet name] -?</g>을 사용하여 모든 cmdlet에 대한 도움말을 얻을 수 있습니다. 현재 도움말 출력은 자동으로 생성되며 명령에 대한 구문을 알려 줍니다. cmdlet 설계를 완료하는 대로 추가 설명서를 추가할 예정입니다.

구문을 검색하는 더 좋은 방법은 PowerShell ISE입니다. PowerShell을 거의 사용하지 않은 경우 이전에 보지 못했을 수 있습니다. 이를 허용하는 SKU에서 실행 중인 경우 ISE를 시작하고 명령 창을 열고 cmdlet 및 해당 매개 변수 집합의 그래픽 표현을 보여 주는 "컨테이너" 모듈을 선택합니다.

PS: 완료할 수 있는지 증명하려면 ersatz <g id="2" ctype="x-code">docker run</g>에서 이미 살펴본 cmdlet 중 일부를 구성하는 PowerShell 함수가 여기에 있습니다. (명확히 하자면 이는 현재 개발에 포함되지 않은 개념 증명입니다.)

``` PowerShell
function Run-Container ([string]$ContainerImageName, [string]$Name="fancy_name", [switch]$Remove, [switch]$Interactive, [scriptblock]$Command) {
    $image = Get-ContainerImage -Name $ContainerImageName
    $container = New-Container -Name $Name -ContainerImage $image
    Start-Container $container

    if ($Interactive) {
         Start-Process powershell ("-NoExit", "-c", "Enter-PSSession -ContainerId $($container.Id)") -Wait
    } else {
        Invoke-Command -ContainerId $container.Id -ScriptBlock $Command
    }

    Stop-Container $container

    if ($Remove) {
        Remove-Container $container -Force
    }
} 
```

## Docker

Docker 명령을 사용하여 Windows 컨테이너를 관리할 수 있습니다. Windows 컨테이너는 해당 Linux 상대와 비교해야 하며 Docker를 통해 동일한 관리 경험을 가지지만 Windows 컨테이너와 의미가 통하지 않는 일부 Docker 명령이 있습니다. 다른 것은 테스트되지 않았습니다. (진행 중입니다.)

Docker에서 사용 가능한 API 문서를 복제하지 않기 위해 해당 관리 API에 대한 링크를 <g id="2" ctype="x-html"></g>여기<g id="4" ctype="x-html"></g>에서 제공합니다. 해당 연습은 매우 유용합니다.

진행 중인 작업 문서에서 Docker API에서 해야 할 작업과 하지 말아야 할 작업에 대해 추적하고 있습니다.






<!--HONumber=Apr16_HO4-->


