---
title: JavaScript 런타임 동작방식, 비동기 처리
description: ""
date: 2024-04-27 20:23:00 +09:00
categories: [SW Development / CS, JavaScript]
tags: [JavaScript, 비동기 처리]
pin: true
math: true
mermaid: true
---

JavaScript 는 싱글 스레드 방식을 채택한 언어로 한 번에 한 가지 작업만을 처리할 수 있다. 그런데 JavaScript 에서 비동기 처리가 가능하다는 말을 한 번쯤은 들어본 적이 있을 것이다.
싱글 스레드인데 어떻게 여러 작업을 동시에 처리하는 비동기 처리가 가능할까? 그 이유는 웹 브라우저나 Node.js 같이 JavaScript 가 실행되는 런타임(Runtime)은 멀티 스레드 방식이기 때문이다.
실제로 웹 브라우저에서는 처리해야 할 작업들이 많다. 네트워크 요청, 이벤트 처리, 파일 다운로드 등 수많은 작업들을 싱글 스레드 방식으로 처리한다면 한 작업이 끝날 때까지 대기하기 때문에
브라우저의 로딩 시간이 엄청 길어질 것이다.

이러한 상황을 방지하기 위해 브라우저를 포함한 JavaScript 런타임은 멀티 스레드 방식을 채택하여 작업을 처리한다.
JavaScript 런타임은 작업을 처리하기 위한 여러가지 요소로 구성되어 있다.

## **JavaScript 엔진, 호출 스택**

JavaScript 엔진이란 런타임 내에서 JavaScript 코드를 해석하는 역할을 한다. 초기의 JavaScript 는 웹개발을 위해 사용된 언어인만큼 브라우저에서
실행되는 다양한 종류의 엔진들이 있다. 주로 사용되는 엔진의 종류들은 다음과 같다.

- V8 : Google Chrome, Node.js
- Rhino : Mozilla
- SpiderMonkey : Mozilla/Firefox
- Chakra : Microsoft/IE 9+ Edge browser

이 중 Chrome 브라우저와 Node.js 에서 사용되는 V8 엔진이 가장 유명하고 대중적이다. 이 글에서는 V8 엔진을 기준으로 동작 원리를 설명할 예정이다.

V8 엔진은 메모리 힙(Heap) 과 호출 스택으로 구성되어 있다. 메모리 힙은 동적 메모리 할당을 위한 공간이다. 호출 스택은 프로그램 실행 시 현재 어느 위치에 있는지, 즉 실행 맥락을 기록하는 스택(stack) 자료구조이다.

예를 들어 다음 코드를 실행한다고 가정하면

```js
function greeting() {
  // Some codes to execute
  sayHi();
}
function sayHi() {
  // Some codes to execute
  return "Hi!";
}

greeting();
```

호출 스택에는 greeting 과 sayHi 가 추가된다. 자세한 동작 과정은 다음과 같다.

1. greeting 이 호출 스택에 먼저 추가되고 greeting 의 내부 코드를 실행한다.
2. sayHi 함수에 도달하면서 sayHi 를 호출 스택에 추가하고 sayHi 함수 내 모든 코드를 실행한다.
3. 실행이 완료되면 다시 greeting 함수 내의 나머지 코드를 마저 실행하고 sayHi 를 호출 스택에서 제거한다.
4. greeting 함수의 실행이 완료되고 호출 스택에서 greeting 을 제거한다.

V8 엔진에는 메모리 힙과 호출 스택이 하나밖에 없기 때문에 한 번에 한 가지 작업만 처리할 수 있다.

호출 스택에는 JavaScript 코드를 실행함과 동시에 전역 실행 맥락이 푸시된다. 따라서 특정 함수를 실행하지 않더라도 호출 스택이 비어있는 것은 아니다. JavaScript 코드 실행이 완전히 끝나면 전역 실행 맥락이 제거되고 그 때 호출 스택이 "완전히" 비게 된다.

## **Web APIs**

Web APIs 는 브라우저 또는 JavaScript 런타임에서 제공하는 별도의 API 모음이다. 타이머, 네트워크 요청, 이벤트 처리 등 다양한 작업에 필요한 API 들이 포함되어 있다. Web API 는 각 API 마다 스레드가 할당되어 있으며 이것들이 모여서 멀티 스레드가 된다. 덕분에 런타임에서 여러 작업을 동시에 처리할 수 있다.

Web API 에는 대표적으로 다음 API 들이 있다.

- DOM
- XMLHttpRequest(AJAX)
- setTimeout, setInterval
- Console API

## **Task Queue**

Task Queue 는 Web API 를 처리하고 있던 스레드로부터 전달받은 콜백 함수을 저장하는 Queue 이다. 콜백 JavaScript 엔진에 포함된 구성 요소이다.

다음의 간단한 예시를 들어보면,

```js
console.log("시작");

setTimeout(function () {
  console.log("3초 후에 실행되는 첫 번째 콜백 함수");
}, 3000);

setTimeout(function () {
  console.log("2초 후에 실행되는 두 번째 콜백 함수");
}, 2000);

console.log("끝");
```

2개의 setTimeout 함수는 각각 별도의 스레드 공간에서 실행된다. 타이머가 완료되면, 두 번째 콜백 함수가 Task Queue 에 먼저 추가되고 이어서 첫 번째 콜백 함수가 추가된다.

Queue 는 먼저 추가되고 먼저 실행되는(FIFO) 구조이므로 Task Queue 에 추가된 콜백 함수들은 추가된 순서대로 실행된다. 만약 Task Queue 가 비어 있으면 콜백 함수는 바로 추가되고 즉시 실행된다. 그러나 다른 콜백 함수가 이미 추가되어 있는 상태라면 기존의 콜백 함수들이 모두 실행된 후에 추가되고 실행된다.

## **이벤트 루프**

위에서 호출 스택, Task Queue 와 더불어 JavaScript 런타임을 동작시키는 또 하나의 요소가 이벤트 루프이다. Task Queue 에 추가된 콜백 함수를 실행시키기 위해서는 호출 스택이 비어 있는지, Task Queue 에 대기 중인 콜백 함수가 대기하고 있는지를 매 순간 확인해야 한다. 이 역할을 수행하는 것이 이벤트 루프이다.

이벤트 루프는 호출 스택과 Task Queue 를이 비어있는지 지속적으로 확인한다. 콜백 함수가 Task Queue 에 들어있고, 호출 스택이 비어있을 때 이벤트 루프는 Task Queue 에 있는 콜백 함수를 빼내어 호출 스택에 추가한다. 만약 Task Queue 에 들어 있는 콜백 함수가 여러 개이고 특정 시점에 호출 스택이 차 있다면, 스택이 비어질 때까지 기다린 후에 Task Queue 에서 콜백 함수를 하나씩 빼내서 호출 스택에 추가해 준다. 이벤트 루프는 이 작업을 계속 반복한다.

아래 그림은 JavaScript 런타임 환경의 구성 요소들과 동작을 시각화한 것이다.
![image](https://i.ibb.co/nbQc6sk/Javascript-event-loop.png)

<div style="font-size:15px; text-align:center">출처 : https://www.webdevolution.com/blog/Javascript-Event-Loop-Explained</div>
<br/>
JavaScript 코드가 실행될 때 호출 스택, Web APIs, Task Queue, 이벤트 루프가 어떻게 동작하는지 자세하게 보고 싶다면 다음 링크에서 코드를 실행하고 확인해 볼 수 있다.

[JavaScript 런타임 동작 테스트](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDQwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)

이번 주제는 내용이 많고 이해하기도 좀 어려웠던 것 같다. 이 글을 여러 번 더 읽어보고 잘못된 부분을 찾는다면 꾸준히 수정해 나가야겠다.
