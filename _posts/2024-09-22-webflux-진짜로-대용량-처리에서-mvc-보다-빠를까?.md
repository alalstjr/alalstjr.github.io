---
title: Webflux 진짜로 대용량 처리에서 MVC 보다 빠를까?
description: 직접 Webflux 와 MVC 성능 비교 테스트를 하고 결과를 알려드리죠!
author: JJunpro
date: 2024-09-22 00:00:00 +0800
categories: [ SpringBoot ]
tags: [ Spring Boot,Java,Webflux,Non-blocking,Kafka ]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/2024-09-22-7.png
  alt: 모든 프로젝트에 Webflux 를 도입해볼까!? (그건 쫌 아닌듯;;)
---

## 프롤로그

동시 접속 사용자 대용량 처리에 관심이 많아서 찾아보다가 Webflux 까지 오게되었습니다.
일반 MVC 와 Webflux 성능 차이를 정리한 블로그글, 유튜브 영상까지 모두 찾아보았고 둘의 엄청난 차이에 Webflux 매력에 빠져서 정말 많은것을 찾아보았습니다. 하지만 직접 테스트 해보고 경험해 봐야 알 수 있는것 직접 테스트 해보기로 합니다.

추가로 회사 프로젝트로 하루 평균 30 ~ 50 만건의 카카오 챗봇 로그를 적재하는 API 개발을 담당하게 되었습니다. (프로젝트 이름은 WaveLens)
제가 가지고있는 개념으로 개발을 하자니 Webflux 로 API 를 개발하고 Kafka 로 메세지를 받고 MongoDB(지금은 엘라스틱 서치) 로 저장하는 방향으로 개발하였습니다.

> 논문집을 참고해서 테스트 하였습니다.
{: .prompt-tip }
[자바 기반의 스프링 WebMVC 와 WebFlux 성능 분석 논문집](https://manuscriptlink-society-file.s3-ap-northeast-1.amazonaws.com/kips/conference/kips2020spring/KIPS_C2020A0050.pdf)

## 웹플럭스를 왜 사용할까?

가장 기본적인 이유는 많은 요청을 서버 프로그램이 효율적으로 동작을 해서 Thread, CPU, Memory 의 자원을 낭비하지 않고 더 많은 요청을 처리할 수 있는 고성능 웹 어플리케이션을 개발하는 것이 목적입니다.
Spring MVC는 1:1로 요청을 처리하기 때문에 트래픽이 몰리면 많은 쓰레드가 생겨납니다.   
`쓰레드가 전환될때 context switching 비용이 발생`하게 되는데 쓰레드가 많을 수록이 비용 커지게 되기 때문에 적절한 쓰레드의 수를 유지해야하는 문제가 있습니다.
이에 반해서 WebFlux 는 `Event-Driven` 과 `Asynchronous Non-blocking I/O` 을 통해 리소스를 효율적으로 사용할 수 있도록 만들어 줍니다.

## Webflux 와 MVC 성능 테스트

당시 테스트한 내용을 공유하자면 다음과 같습니다.  
챗봇로그 데이터 적재를 위해 Non-Blocking 방식과 Blocking 방식 간의 데이터 전송 속도와 효율성을 비교하여 성능 측정 진행

- 테스트환경
  - non-blocking : Webflux(MongoDB)
    - [Non-Blocking WebFlux Git Repository 바로가기](https://github.com/alalstjr/non-blocking-webflux)
  - blocking : MVC(MySQL)
    - [Blocking MVC Git Repository 바로가기](https://github.com/alalstjr/blocking-mvc)

가상 컴퓨팅 공간내부의 설치된 DB 와 Docker 로 가동된 Spring Application 에서 진행 (옛날 테스트는 개인 컴퓨터 docker 에서 실행)



### Insert 테스트

#### non-blocking
![non-blocking 테스트 결과 사진](/assets/img/2024-09-22-1.png)

#### blocking
![blocking 테스트 결과 사진](/assets/img/2024-09-22-2.png)

#### 결과 요약
총 표본 수 : 100000(Thread 1000 - 사용자 수)

- 평균 응답 시간 (Response Times - 평균) 
  - blocking 결과 테이블: 1817.11 ms
  - `non-blocking결과 테이블: 26.82 ms`
- Throughput (트랜잭션(초당)) 
  - blocking 결과 테이블: 541.68
  - `non-blocking 결과 테이블: 1141.91`
- Network 수신 및 전송 (Network (KB/sec)) 
  - blocking 결과 테이블: 수신 - 117.96 KB/sec, 전송 - 80.93 KB/sec
  - `non-blocking 결과 테이블: 수신 - 289.94 KB/sec, 전송 - 301.09 KB/sec`

## Webflux + Kafka

`1,000,000만` 건의 데이터로 성능 측정 진행했으며 `Kafka Topic 파티션 갯수는 12개` 입니다.

#### Topic Lag 처리량 확인

service-trace.record-create.json topic 의 Lag 결과

![service-trace.record-create.json topic 의 Lag 결과](/assets/img/2024-09-22-5.png)

Lag 값이 `3700 ~ 4000 사이를 일정하게 유지하면서 컨슈머가 메세지를 처리`하고 있습니다.

#### 처리속도 & 오류 확인

![오류없이 처리하는 서버!](/assets/img/2024-09-22-6.png)

오류는 0% 를 유지하며 `8분동안 1,000,000건의 데이터`를 빠른 속도로 문제없이 처리하였습니다.

## 결론

non-blocking의 평균 응답 시간이 낮고 Throughput이 높으며, Network 수신 및 전송량도 높은 것을 확인할 수 있습니다.  
Webflux는 데이터 전송 속도 측면에서 수신이 매우 높으므로, 대용량 데이터 전송에 용이할 것으로 보입니다.  

옛날에는 개인 컴퓨터로 테스트를 했다면 이번에는 가상 커퓨팅 공간에 테스트를 진행하였습니다.  
확실히 많은 데이터를 처리할 때 속도는 webflux 앞서 있다는 것을 확인할 수 있었습니다.  

하지만 테스트 양이 적으면 오히려 MVC 성능이 더 잘 나오는것을 확인하였습니다. 
그러므로 무작정 webflux 가 최고다! 이게 아닌 **프로젝트 규모를 보면서 도입해야 하는것을 개발자는 인지**해야합니다.  

### Spring Webflux가 유리한 프로젝트의 유형 10가지

1. **고효율 스트리밍 서비스**
  - 웹플럭스는 리액티브 스트림 스펙을 준수하므로, 백프레셔(Backpressure)를 통해 스트림의 효율성을 극대화할 수 있습니다.
2. **대규모 실시간 데이터 처리**
   - 리액티브 모델은 데이터 처리량이 많고 실시간 응답이 필요한 경우에 매우 효과적입니다.
3. **마이크로서비스 아키텍처**
   - 각각의 서비스가 독립적으로 확장 가능해야 하는 환경에서는 리액티브 시스템이 유용합니다.
4. **이벤트 주도 프로젝트**
   - 비동기식 및 non-blocking 방식의 처리로 인해 이벤트 주도 시스템에서는 WebFlux가 잘 맞습니다.
5. **서버 푸시 기반 애플리케이션**
   - Server-sent events나 WebSocket 같은 기술을 통해 클라이언트에 서버로부터의 실시간 업데이트가 필요한 애플리케이션에 유리합니다.
6. **로드 밸런싱이 필요한 서비스**
   - 높은 트래픽에 대응하기 위해 로드 밸런싱이 필요한 서비스에서는 더 효과적인 리소스 사용이 가능합니다.
7. **클라우드 기반 서비스**
   - 클라우드 환경에서는 컴퓨팅 리소스를 효율적으로 활용해야 하므로 리액티브 프로그래밍이 유리합니다.
8. **API 게이트웨이**
   - 여러 백엔드 서비스로부터 데이터를 집계하고 반환하는 API 게이트웨이에서는 비동기 및 non-blocking 처리를 통해 더 효율적인 응답 시간을 달성할 수 있습니다.
9. **비동기 메시징 어플리케이션**
   - 실시간 채팅이나 알림 등의 비동기 메시징 시스템에서는 WebFlux의 비동기 처리 능력이 장점으로 작용합니다.
10. **IoT 시스템**
    - 수많은 IoT 디바이스로부터의 데이터를 실시간으로 처리해야 하는 IoT 시스템에서 리액티

[출처: 황민호 (Kakao General Developer)](https://careerly.co.kr/comments/85626)

### Spring MVC 가 적합한 프로젝트의 유형 10가지

1. **단순 CRUD 작업이 주인 프로젝트**
  - 데이터베이스와의 간단한 상호작용이 주인 경우, 리액티브 프로그래밍의 복잡성이 오히려 부담스러울 수 있습니다.
2. **전통적인 블로킹 I/O가 주로 사용되는 프로젝트**
  - 블로킹 I/O 작업이 많은 경우, 리액티브 프로그래밍의 장점을 충분히 활용하지 못할 수 있습니다.
3. **동기식 처리가 필요한 프로젝트**
  - 동기식 처리가 필요한 로직이 많은 경우, 비동기적인 리액티브 프로그래밍은 코드를 복잡하게 만들 수 있습니다.
4. **리소스가 제한적인 프로젝트**
  - 리액티브 프로그래밍은 메모리와 CPU를 효율적으로 사용하지만, 이로 인해 가비지 컬렉션에 더 많은 부하가 가해질 수 있습니다. 리소스가 제한적인 경우, 이는 고려해야 할 부분입니다.
5. **단일 스레드 환경 프로젝트**
  - 리액티브 프로그래밍은 멀티스레드 환경에서 가장 큰 이점을 제공합니다. 단일 스레드 환경에서는 그 이점이 크게 나타나지 않을 수 있습니다.
6. **레거시 시스템과의 통합이 필요한 프로젝트**
  - 이미 구축된 블로킹 형식의 레거시 시스템과 통합이 필요한 경우, 리액티브 시스템은 통합을 복잡하게 만들 수 있습니다.
7. **리액티브 프로그래밍에 경험이 부족한 팀**
  - 리액티브 프로그래밍은 복잡하고 학습 곡선이 길어, 팀이 이에 익숙하지 않은 경우 프로젝트의 진행을 방해할 수 있습니다.
8. **코드 디버깅이 많이 필요한 프로젝트**
  - 리액티브 프로그래밍의 비동기 및 병렬 처리 특성은 코드의 디버깅을 어렵게 만듭니다.
9. **트랜잭션 처리가 필요한 프로젝트**
  - 일반적인 RDBMS 기반의 트랜잭션 처리는 리액티브 모델과 잘 맞지 않을 수 있습니다.
10. **반응성이 크게 중요하지 않은 프로젝트**
  - 대량의 요청을 빠르게 처리하거나, 높은 확장성이 필요하지 않은 프로젝트에서는 리액티브 프로그래밍의 복잡성이 그 이점을 상회할 수 있습니다.

[출처: 황민호 (Kakao General Developer)](https://careerly.co.kr/comments/85626)

## 에필로그

[Webflux 진짜로 대용량 처리에서 MVC 보다 빠를까? OKKY 토론 글](https://okky.kr/questions/1412539)  
이 글은 약 1년 전에 제가 WebFlux에 대해 조사하고 테스트한 내용을 공유한 것입니다.   
댓글에는 WebFlux와 MVC 성능에 대한 개발자들의 토론이 많으니 한 번 읽어보시길 추천합니다.  
또한, 1년 후 실무에 적용한 사례를 추가해 게시글을 수정했으니, 원문과 함께 읽어보시면 많은 도움이 될 것입니다.

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Falalstjr.github.io%2Fposts%2Fwebflux-%25EC%25A7%2584%25EC%25A7%259C%25EB%25A1%259C-%25EB%258C%2580%25EC%259A%25A9%25EB%259F%2589-%25EC%25B2%2598%25EB%25A6%25AC%25EC%2597%2590%25EC%2584%259C-mvc-%25EB%25B3%25B4%25EB%258B%25A4-%25EB%25B9%25A0%25EB%25A5%25BC%25EA%25B9%258C%2F&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)
