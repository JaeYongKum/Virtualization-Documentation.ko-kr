---
layout: LandingPage
title: Windows 문서의 컨테이너
description: Windows에서 컨테이너를 실행 하는 방법에 대 한 설명서
keywords: Docker, 컨테이너
author: cwilhit
ms.date: 09/11/2019
ms.topic: landing-page
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 74c9d604-0915-4d89-bc69-0263b76bc66b
ms.openlocfilehash: 9a5b08f87983e285418ae333e3a948af9911d73d
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209743"
---
<div id="main" class="v2">
    <ul class="cardsY panelContent featuredContent">
        <li>
            <a href="https://docs.microsoft.com/en-us/azure/aks/windows-container-cli" data-linktype="external">
                <div class="cardSize">
                    <div class="cardPadding">
                        <div class="card">
                            <div class="cardImageOuter">
                                <div class="cardImage">
                                    <img src="media/logo_kubernetes.svg" alt="" data-linktype="relative-path">
                                </div>
                            </div>
                            <div class="cardText">
                                <h3>AKS에서 Windows 컨테이너를 사용해 보세요!</h3>
                            </div>
                        </div>
                    </div>
                </div>
            </a>
        </li>
        <li>
            <a href="https://hub.docker.com/_/microsoft-windows-base-os-images" data-linktype="external">
                <div class="cardSize">
                    <div class="cardPadding">
                        <div class="card">
                            <div class="cardImageOuter">
                                <div class="cardImage">
                                    <img src="media/logo_docker.svg" alt="" data-linktype="relative-path">
                                </div>
                            </div>
                            <div class="cardText">
                                <h3>Docker 허브에서 컨테이너 이미지를 확인 하세요.</h3>
                            </div>
                        </div>
                    </div>
                </div>
            </a>
        </li>
        <li>
            <a href="https://techcommunity.microsoft.com/t5/Containers/bg-p/Containers" data-linktype="external">
                <div class="cardSize">
                    <div class="cardPadding">
                        <div class="card">
                            <div class="cardImageOuter">
                                <div class="cardImage">
                                    <img src="media/i_blog.svg" alt="" data-linktype="relative-path">
                                </div>
                            </div>
                            <div class="cardText">
                                <h3>컨테이너 블로그를 읽습니다.</h3>
                            </div>
                        </div>
                    </div>
                </div>
            </a>
        </li>
    </ul>
    <h1>Windows 문서의 컨테이너</h1>
    <br/>
    <div class="abstract">Windows 컨테이너는 사용자가 종속성을 사용 하 여 응용 프로그램을 패키지 하 고 운영 체제 수준의 가상화를 활용 하 여 단일 시스템에서 빠르고 완전 하 게 격리 된 환경을 제공할 수 있도록 합니다. 빠른 시작 가이드, 배포 가이드, 샘플에서 Windows 컨테이너를 사용 하는 방법에 대해 알아봅니다.</div>
    <ul class="cardsW panelContent featuredContent">
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/virtualization-containers-about.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">개요</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/about/index" data-linktype="absolute-path">Windows 컨테이너 정보</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/deploy-containers/system-requirements" data-linktype="absolute-path">시스템 요구 사항</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/about/faq" data-linktype="absolute-path">FAQ</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/virtualization-containers-quick-start.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">시작</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/quick-start/set-up-environment" data-linktype="external">환경 설정</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/quick-start/run-your-first-container" data-linktype="external">첫 번째 컨테이너 실행</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/quick-start/building-sample-app" data-linktype="external">샘플 앱 Containerize</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/container-tutorials.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">자습서</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile" data-linktype="external">Windows 컨테이너 빌드</a></li>
                                <li><a href="/azure/aks/windows-container-cli" data-linktype="external">AKS에서 실행</a></li>
                                <li><a href="/azure/service-fabric/service-fabric-quickstart-containers" data-linktype="external">Service Fabric에서 실행</a></li>
                                <li><a href="/azure/app-service/app-service-web-get-started-windows-container" data-linktype="external">Azure 앱 서비스에서 실행</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/virtualization-containers-management-tools.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">개념</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon" data-linktype="external">Docker</a></li>
                                <li><a href="/virtualization/windowscontainers/about/overview-container-orchestrators" data-linktype="external">컨테이너 오케스트레이션</a></li>
                                <li><a href="/virtualization/windowscontainers/manage-containers/container-base-images" data-linktype="external">Windows 컨테이너 essentials</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/container-reference.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">참조</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility" data-linktype="external">버전 호환성</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/deploy-containers/base-image-lifecycle" data-linktype="external">이미지 서비스 수명 주기</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/images-eula" data-linktype="external">컨테이너 OS 이미지 EULA</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
        <li>
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage bgdAccent1">
                                <img src="media/virtualization-containers-community.svg" alt="" data-linktype="relative-path">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3 style="margin: 8px 0 2px 0;">리소스</h3>
                            <ul>
                                <li><a href="/en-us/virtualization/windowscontainers/samples" data-linktype="external">샘플</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/troubleshooting" data-linktype="external">문제 해결</a></li>
                                <li><a href="/en-us/virtualization/windowscontainers/communitylinks" data-linktype="external">커뮤니티 비디오 및 블로그</a></li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </li>
    </ul>
</div>