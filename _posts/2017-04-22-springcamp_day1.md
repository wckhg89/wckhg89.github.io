---
layout: post
title:  "스프링캠프2017 첫째날"
date:   2017-03-27 17:00:01 -0500
categories: SPRING
fb_title: springcamp_day1
---

스프링 캠프 2017 리뷰!

# Session 1. 프로세스, 스레드, 리액티브 (부종민 님)

- Thread 일반

  - 프로세스 - 독립된 실행 단위

    - 과거
    - 현대
      - CPU - OS의 스케줄러에 의해 타임스라이스 만큼 실행
      - Memory - OS로 할당받은 메모리 공간 사용
        - CODE, DATA, HEAP, STACK

    - 힙영역 : 스레드끼리 공유 가능한 공간

  - 컨테스트 스위칭

    - 스케줄러간에 스레드가 옮겨지는 과정

    - 스레드큐 -> 스케줄러 -> CPU

    - 컨텍스트 스위칭 비용

  - 스레드 풀

    - 스레드를 미리 만들어 놓고 재사용 하는것 -> 비용감소 (스레드 생성 cpu, memory)

    - 스레드풀이 꽉 차있으면 더이상 요청이 불가능

    - 스레드 매니저를 두어서 적절히 코드를 스레드풀에 있는 스레드에 할당하고 할당받지 못한 코드들은 큐에 대기


  - 스레드 인 JVM

    - JVM 스레드 비용

      - 스레드 생성 비용 (OS + JVM)
      - 컨텍스트 스위칭
      - Garbage collection

        - 성능 최적화 측면에서는 더 많이 신경 써야함

- 자바에서 비동기 API

  - 비동기가 필요한 상황

    - CPU expensive

    - IO Blocking

      - 다음 채널 서비스 : 외부 리퀘스트가 많아 API 호출이 많음

        - 통신을 순차적으로 하면 3번의 호출이 있으면 3초가 걸림

        - 비동기로 바꾸면 서로 다른 스레드들이 개별적으로 호출

    - 자바 비동기 API

      - Thread / Runnable

      - 어려운점
        - 하나의 코드에 여러 스레드가 돌고 있어 해석이 쉽지 않음

        - 스레드들끼리 데이터 공유를 멤버변수 (Heap 영역)을 통해 해야함

          - 동기화 이슈 : wait(), notify()

            - Future interface

              - FutureTask

                - futrue.get() Block!!!

- CompletableFutre 도입

  - 의존성 있는 테스트 Future에서 해결하려면 콜백헬을 보게됨

  - CompletableFutre Class

    - 태스크 간의 순서 체이닝 방식으로 해결 가능 (thenApply)


- NIO

  - 쓰레드풀 풀! (아직 풀지 못한 상황)

    - NIO 패키지 (New IO -> Native IO, Nonblocking IO)

    - Spring MVC : DeferredResult


- Reactive (Event Programming 마무리)

  - CompletableFutre 나 Nonblocking IO 공통점

    - 이벤트 발생 -> 처리

  - 이벤트 : 프로그램에 의해 감지되고 처리되는 동작이나 사건

  - 이벤트 프로그래밍

    - 인벤트가 발생한다는 전제를 두고 이벤트를 처리하는 코드를 작성 (절차지향과 다름!)

    - 리액티브 프로그래밍도 이벤트 기반 프로그래밍중 한축


  - 이벤트 프로그래밍 관점에서 추상화 된 API

    - Spring 5 MVC, reactive-steream, Reactor

    - obserable, Rxjava, Reactor, sodium, Flow, AKKA ...



# Session2. Async & Spring

- 블록킹, 논 블록킹

  - 동기 비동기와는 관점이 다름
  - 내가 직접 제어할 수 없는 대상을 상대하는 방법
  - 대상이 제한적임
    - IO / 멀티쓰레디 동기화

- @Async (비동기 실행)

  - @Async는 비동기기 때문에 아래와 같은 리턴 타입만 지원한다.
    - void / Future<T>, ListenableFuture<T>, CompletableFuture<T>

  - @Async 어노테이션이 있는 메소드는 스레드 만들고 버림

  - SimpleAsyncTaskExecutor
    - @Async가 사용하는 기본 TaskExecutor
    - Thread poll 아님

  - Executor, ExecutorService, TaskExecutor 빈 등록하고 사용하면 재사용

    - @Async("myExecutor")

  - 50개의 @Async 메소드 호출이 동시에 일어나면 10개 만든다.

    - 스프링은 코어풀 사이즈 만큼 스레드 만들고 추가적인거는 일단 큐에 대기시킨다.


  - 비동기 - 논블록킹 API 요청과 @MVC
    - RestTemplate 동기시 요청 보내면 스레드하나가 블록킹됨
    - AsyncRestTemplate 콜백방식으로 결과를 받음 (ListenableFuture)
      - 요청때 순간적으로 스레드 만들어버리고 버림 (아깝)
      - Netty4ClientHttpRequestFactory 만들어서 사용

  - 결론
    - 비동기 작업과 API 호출이 많은 @MVC 앱이라면
      - @MVC
      - AsyncTemplate + 논블록킹 IO 라이브러리
