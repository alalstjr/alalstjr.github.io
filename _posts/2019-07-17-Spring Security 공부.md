---
layout:     post
title:      Spring Security 공부
author:     쭌프로
tags:       JAVA Spring-Boot SpringSecurity
subtitle:   Spring Security 공부노트
category:   JAVA
---

<!-- Start Writing Below in Markdown -->

![Description]({{ site.url }}/img/java_bg.png)

# Spring Security(스프링 시큐리티) 란 무엇인가?

스프링 시큐리티 레퍼런스에서는 자바 EE 기반의 엔터프라이즈 소프트웨어 애플리케이션을 위한 포괄적인 보안 서비스들을 제공하고 
<b>오픈 플랫폼</b>이면서 자신만의 인증 매커니즘을 간단하게 만들 수 있습니다.

스프링 시큐리티를 이해하기 위해서는 스프링 시큐리티가 <b>애플리케이션 보안을 구성하는 두가지 영역</b>에 대해서 알아야 합니다. 

바로 <b>인증(Authentication)과 권한(Authorization)</b> 이라는 것입니다.

- 인증 : 애플리케이션의 작업을 수행할 수 있는 주체(사용자)라고 주장할 수 있는 것
- 권한 : 권한은 인증된 주체가 애플리케이션의 동작을 수행할 수 있도록 허락되있는지를 결정하는 것

권한 승인이 필요한 부분으로 접근하려면 인증 과정을 통해 주체가 증명 되어야만 한다는 것입니다.

# 자세한 것은 아래 GIT 저장소에서 확인하기

[쭌프로 git 저장소](https://github.com/alalstjr/spring-security-jwt){:target="_blank"}

# 참고자료

[Okky 커뮤니티 사이트 - 초보자가 이해하는 Spring Security (좋은 글)](https://okky.kr/article/382738){:target="_blank"}