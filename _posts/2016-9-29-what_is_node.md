---
layout: post
title:  "node.js 에 대해서 알아보려고 합니다."
date:   2016-09-28 17:55:01 -0500
categories: nodeJS
permalink: /archivers/test
---
# node.js 에 대한 소개

<h4>Single-thread & non-blocking IO</h4>

 > node.js는 <font color='red'>싱글 스레드 기반</font>으로 동작하는 <font color='red'>non-blocking IO</font>를 지원하는 서버입니다.

<h4>Runnig on v8 engine & event-driven</h4>

 > <font color='red'>V8 엔진</font>으로 개발되어 있으며 프로그래밍 언어로는 Javascript를 사용하며 <font color='red'>Event 기반(event-driven)</font>의 프로그래밍 모델을 사용합니다.

<h2> node.js의 내부구조와 내부동작 원리</h2>

<h4>node.js 내부구조</h4>

<img src='https://wckhg89.github.io/images/20160929_node_arch.jpg'/>

- 빨강 상자

> node.js는 구글 V8 자바스크립으 엔진을 기본으로 합니다. 이를 기반으로 Single-thread 기반의 Event Loop (libev)가 돌면서 요청을 처리합니다.
그림을 보면 Thread Pool (libeio)를 볼 수 있는데 (싱글 스레드인데 스레드 풀!?) 이는 시스템적으로 non-blocking IO를 지원하지 않는 IO 호출이 있는 경우, 이를 비동기 처리 하기 위해 내부의 Thread Pool을 별도 이용하여 차리하도록 되어 있습니다.


- 파랑상자

> 빨강상자 위의 파랑상자 영역은 네트워크 프로토콜 (http ...)을 처리하는 socket, http 바인딩 모듈등이 있습니다.


- 오렌지 상자

> 마지막 오렌지 상자는 노드 JS의 기본 라이브러리를 말합니다. 노드에서 기본적으로 제공하는 라이브러리(모듈)에는  HTTP, TCP, FS, OS, EVENT 모듈 등이 있습니다.
노드 기본 라이브러리는 require를 할때에 별도의 경로 지정없이 사용할 수 있습니다.


```
// node binding module
var http = require('http');
...
http.get('/', function () {...});
```

```
var fs = require('fs');
fs.readFile('./bigFile.txt', 'utf8', function(err, data) {
  console.log(data);
});
```

<h4>node.js 내부동작 원리</h4>

지금까지 node.js의 내부 구조가 어떻게 되었는지 살펴 보았으니 이제 어떤식으로 동작이 되는지 살펴보려 합니다.

앞서 언급했듯이 node.js는 'single-thread' 기반으로 'event-driven non-blocking IO'를 지원한다고 했는데요. (뭔소리야...?)

지금 이 2가지 (single-thread, event-driven non-blocking IO)에 대해 살펴보겠습니다.

<h6>Multi-thread(process) 모델</h6>

 Tomcat과 같은 WAS나 Apache와 같은 일반적인 웹서버는 Mulit Process 또는 Multi Thread 형태를 가지고 있습니다.

<img src='https://wckhg89.github.io/images/20160929_node_single_thread.jpg'/>

> 톰켓과 같은 경우는 위의 그림에서 보는것 처럼 Client로 부터 요청이 오면, 미리 만들어놓은 Thread(Thread Pool에 있고, tomcat 설정에서 몇개까지 만들어 놓을지 지정할 수 있습니다...)를 꺼내서 사용하고 요청이 끝나면 다시 Thread Pool 넣어 주는 구조입니다. 그렇기 때문에 동시에 처리할 수 있는 Client 수에 한계가 있습니다.(미리 지정한 thread의 수만큼 동시에 처리 할 수 있겠지요...)

<img src='https://wckhg89.github.io/images/20160929_node_blocking_io.jpg'/>

> 또한 IO 작업을 할때는 (DB...) CPU를 사용하지 않고 Wait 상태로 빠져 CPU를 낭비하게 됩니다.

<h6>Single-thread 모델</h6>
앞서 언급했던 Multi-thread 모델의 한계를 해결 하기 위한 것이 Single-thread 모델 입니다.

<img src='https://wckhg89.github.io/images/20160929_node_non_blocking_io.jpg'/>

Single-thread 모델은 다음과 같이 정의 할 수 있는데요.

> <font color='red'>하나의 Thread</font>만을 사용해서 여러 Client로부터 오는 Request(요청)를 처리한다.
단, IO 작업과 같이 block되는 작업과 같은 경우는 비동기 IO 방식으로 IO 요청을 던져 놓고, 다시 돌아와서 다른 작업을 하다가 IO 작업이 끝나면 이벤트를 받아서 처리하는 구조이다. (그래서 node.js에서는 유독 callback function이 많이 이용되죠...)

그런데 말입니다...(김상중 gem) 앞서 노드구조를 살펴봤듯이 node.js도 thread-pool(libeio)을 가지고 있는데요.

이는 node.js도 single-thread만 사용하는 것이 아니라 내부적으로는 multi thread pool을 사용하기는 한다고 합니다.

<img src='https://wckhg89.github.io/images/20160929_node_thread_pool.jpg'/>


<h6>Event-driven (Async) Non-blocking IO</h6>

node.js는 이벤트가 발생하면, 그 이벤트를 받아서 작업을 처리합니다. 그렇기 때문에 event-driven 이라고 말합니다. <br>또한 이러한 이벤트 처리가 blocking 되는 작업이 있는 작업이라면(예를들어 DB 작업이라던가...) 이를 비동기적으로 처리해서 blocking 되지 않는 것처럼 하기 때문에 Non-blocking 이라고 합니다.

자, 그러면 blocking IO(동기식:sync)와 Non-blocking(비동기식:async)의 차이를 한번 살펴보겠습니다.

- Blocking IO (동기식 : sync)

<img src='https://wckhg89.github.io/images/20160929_node_sync_io.jpg'/>
> 먼저 동기식 IO는 위의 그림과 같이 동작합니다. 그림 보면 다 이해 되쥬? (백종원 gem)

- Non-blocking IO (비동기식 : async)

<img src='https://wckhg89.github.io/images/20160929_node_async_io.jpg'/>
> 비동기식 IO는 위 그림처럼 동작합니다.

---

# 마치며..

지금까지 node.js가 어떤식으로 되어 있는 구조적으로 살펴보았습니다. 사실 이보다 훨씬 복잡한 구조로 되어있을텐데, node.js를 패스트캠퍼스에서 수강하고 있는데 어떤 구조로 동작하는지 정확히 이해하지 못해 와닿는게 조금 부족한 것 같아 간략하게라도 어떤식으로 동작하는지 정리해보았습니다.

node.js의 구조에 대해서는 앞으로도 계속 공부해서 좀 더 상세히 보강할 수 있도록 하려 합니다.

# 끝!!! (오와리, 엔드)
