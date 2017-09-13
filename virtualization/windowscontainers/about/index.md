---
title: About Windows Containers
description: Learn about Windows containers.
keywords: docker, containers
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 2be7a06c7b7b154e392c30981cdf954d2d1b796e
ms.sourcegitcommit: 8e193d8c274a549aef497f16dcdb00d7855e9fa7
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 08/02/2017
---
# Windows 컨테이너

## 컨테이너란?

컨테이너는 응용 프로그램을 자체 격리 상자 안에 래핑하는 방법입니다. 컨테이너 속에 있는 응용 프로그램은 상자 외부에 있는 다른 응용 프로그램 또는 프로세스에 대한 정보를 전혀 갖고 있지 않습니다. 이 응용 프로그램이 실행에 사용하는 모든 것 역시 이 컨테이너 안에 있습니다.  상자가 어디로 이동하든 응용 프로그램 실행에 필요한 모든 것이 번들에 포함되기 때문에 응용 프로그램이 항상 충족됩니다.

부엌을 상상해 보세요. 모든 기구와 가구, 냄비와 팬, 주방 세제와 핸드 타월을 패키지로 구성합니다. 이것이 컨테이너입니다.

<center style="margin: 25px">![](media/box1.png)</center>

이 컨테이너를 가져와서 우리가 원하는 호스트 아파트에 배치하면 똑같은 부엌이 됩니다. 필요한 기구가 모두 있으니 전기와 수도만 연결하면 바로 요리를 시작할 수 있습니다.

<center style="margin: 25px">![](media/apartment.png)</center>

컨테이너는 이 부엌과 거의 동일합니다. 서로 다른 종류의 방도 있을 것이고 같은 종류의 방도 있을 것입니다. 중요한 점은 필요한 모든 것이 컨테이너에 패키징된다는 것입니다.

[Windows 기반 컨테이너: 엔터프라이즈급 제어 기능을 통한 최신 앱 개발](https://youtu.be/Ryx3o0rD5lY)에 대한 간략한 개요를 확인하세요.

## 컨테이너의 기본 사항

컨테이너는 격리되고 리소스로 제어되는 휴대용 런타임 환경으로, 호스트 컴퓨터 또는 가상 컴퓨터에서 실행됩니다. 컨테이너에서 실행되는 응용 프로그램 또는 프로세스는 필요한 모든 종속성 및 구성 파일이 패키지로 포함되기 때문에 컨테이너 외부에서 다른 프로세스가 전혀 실행되지 않는 것처럼 보입니다.

컨테이너의 호스트는 컨테이너용 리소스 집합을 프로비전하고 컨테이너는 이러한 리소스만 사용합니다. 컨테이너는 자신에게 제공된 리소스 이외의 리소스에 대해서는 전혀 알지 못하며, 따라서 이웃하는 컨테이너에 프로비전된 리소스를 사용할 수 없습니다.

Windows 컨테이너의 만들기 및 작업을 시작할 때 다음 주요 개념이 유용할 것입니다.

**Container Host:** Physical or Virtual computer system configured with the Windows Container feature. 컨테이너 호스트는 하나 이상의 Windows 컨테이너를 실행합니다.

**컨테이너 이미지:** 샌드박스에서 캡처한 소프트웨어 설치 등 컨테이너 파일 시스템이나 레지스트리에 대한 수정 내용입니다. 대부분의 경우 이러한 변경 내용을 상속하여 새 컨테이너를 만들 수 있게 이 상태를 캡처하고자 할 수 있습니다. That’s what an image is – once the container has stopped you can either discard that sandbox or you can convert it into a new container image. For example, let’s imagine that you have deployed a container from the Windows Server Core OS image. You then install MySQL into this container. Creating a new image from this container would act as a deployable version of the container. This image would only contain the changes made (MySQL), however would work as a layer on top of the Container OS Image.

**Sandbox:** Once a container has been started, all write actions such as file system modifications, registry modifications or software installations are captured in this ‘sandbox’ layer.

**Container OS Image:** Containers are deployed from images. The container OS image is the first layer in potentially many image layers that make up a container. 이 이미지는 운영 체제 환경을 제공합니다. 컨테이너 OS 이미지는 변경할 수 없습니다. 즉, 수정할 수 없습니다.

**컨테이너 리포지토리:** 컨테이너 이미지가 생성될 때마다 컨테이너 이미지와 해당 종속성이 로컬 리포지토리에 저장됩니다. 이러한 이미지는 컨테이너 호스트에서 여러 번 다시 사용할 수 있습니다. 컨테이너 이미지를 공용 또는 개인 레지스트리(예: DockerHub)에 저장하여 여러 다른 컨테이너 호스트에서 사용할 수도 있습니다.

<center>![](media/containerfund.png)</center>

가상 컴퓨터에 익숙한 분들의 눈에는 컨테이너가 가상 컴퓨터와 매우 유사하게 보일 수 있습니다. A container runs an operating system, has a file system and can be accessed over a network just as if it was a physical or virtual computer system. 그렇긴 하지만 컨테이너의 바탕이 되는 기술 및 개념은 가상 컴퓨터와는 상당한 차이가 있습니다.

Microsoft Azure 전문가 Mark Russinovich의 [유용한 블로그 게시물](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/)을 통해 이러한 차이점에 대해 자세히 알아볼 수 있습니다.

## Windows 컨테이너 형식

Windows Containers include two different container types, or runtimes.

**Windows Server Containers** – provide application isolation through process and namespace isolation technology. A Windows Server Container shares a kernel with the container host and all containers running on the host. 이러한 컨테이너는 적대적 보안 경계를 제공하지 않으므로 신뢰할 수 없는 코드 격리에 사용하면 안 됩니다. 공유 커널 공간 때문에 이러한 컨테이너에 동일한 커널 버전과 구성이 필요합니다.

**Hyper-V 격리** – 고도로 최적화된 가상 컴퓨터에서 각 컨테이너를 실행하여 Windows Server 컨테이너에서 제공하는 격리를 확장합니다. In this configuration, the kernel of the container host is not shared with other containers on the same host. 이러한 컨테이너는 가상 컴퓨터와 보안 보장 수준이 같은 적대적 다중 테넌트 호스팅을 위해 디자인되었습니다. 이러한 컨테이너는 호스트 또는 호스트의 다른 컨테이너와 커널을 공유하지 않으므로 버전과 구성이 다른 커널을 실행할 수 있습니다(지원되는 버전에서). 예를 들어 Windows 10의 모든 Windows 컨테이너는 Windows Server 커널 버전과 구성을 활용하기 위해 Hyper-V 격리를 사용합니다.

Hyper-V 격리를 사용하여 또는 사용하지 않고 Windows에서 컨테이너를 실행하는 것은 런타임 결정입니다. 처음에 Hyper-V 격리를 사용하여 컨테이너를 만들고 나중에 런타임에 컨테이너를 Windows Server 컨테이너 대신 실행하도록 선택할 수 있습니다.

## Docker란?

컨테이너에 대해 공부하다 보면 Docker에 대한 이야기가 반드시 나오게 됩니다. 컨테이너 이미지를 패키징하여 제공하는 용기를 Docker라고 합니다. 이 자동화된 프로세스는 이미지를 생산하며(템플릿으로 효과적으로) 생산된 이미지는 온-프레미스, 클라우드, 개인 컴퓨터 등 어디서나 컨테이너로 실행할 수 있습니다.

<center>![](media/docker.png)</center>

다른 컨테이너와 마찬가지로, [Docker](https://www.docker.com)를 사용하여 Windows Server 컨테이너를 관리할 수 있습니다.

## 개발자를 위한 컨테이너 ##

From a developer’s desktop to a testing machine to a set of production machines, a Docker image can be created that will deploy identically across any environment in seconds. This story has created a massive and growing ecosystem of applications packaged in Docker containers, with DockerHub, the public containerized-application registry that Docker maintains, currently publishing more than 180,000 applications in the public community repository.

When you containerize an app, only the app and the components needed to run the app are combined into an "image". Containers are then created from this image as you need them. You can also use an image as a baseline to create another image, making image creation even faster. Multiple containers can share the same image, which means containers start very quickly and use fewer resources. For example, you can use containers to spin up light-weight and portable app components – or ‘micro-services’ – for distributed apps and quickly scale each service separately.

Because the container has everything it needs to run your application, they are very portable and can run on any machine that is running Windows Server 2016. You can create and test containers locally, then deploy that same container image to your company's private cloud, public cloud or service provider. The natural agility of Containers supports modern app development patterns in large scale, virtualized and cloud environments.

With containers, developers can build an app in any language. These apps are completely portable and can run anywhere - laptop, desktop, server, private cloud, public cloud or service provider - without any code changes.  

Containers helps developers build and ship higher-quality applications, faster.

## Containers for IT Professionals ##

IT Professionals can use containers to provide standardized environments for their development, QA, and production teams. They no longer have to worry about complex installation and configuration steps. By using containers, systems administrators abstract away differences in OS installations and underlying infrastructure.

Containers help admins create an infrastructure that is simpler to update and maintain.

## Video Overview

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## Windows Server 컨테이너 사용해 보기

컨테이너의 놀라운 성능을 활용할 준비가 되었나요? 아래 링크를 클릭하여 첫 번째 컨테이너를 직접 배포해 보세요. <br/>
Windows Server 사용자의 경우 여기로 이동 - [Windows Server 빠른 시작 소개](../quick-start/quick-start-windows-server.md) <br/>
Windows 10 사용자의 경우 여기로 이동 - [Windows 10 빠른 시작 소개](../quick-start/quick-start-windows-10.md)

