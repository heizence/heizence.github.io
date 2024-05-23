---
title: JavaScript Scope Chain
description: ""
date: 2024-05-23 19:50:25 +09:00
categories: [SW Development / CS, JavaScript]
tags: [JavaScript, Scope]
pin: true
math: true
mermaid: true
---

## Scope Chain(스코프 체인) 이란?

스코프 체인에 대해 학습하면서 여러 글을 참고하였는데 정의가 1가지로 결정되지 않고 크게 2가지로 나뉘는 것 같다.

1. 변수를 참조할 때 해당 변수가 어느 스코프에 있는지 탐색하여 변수값을 찾는 과정을 의미한다.
2. 여러 개의 스코프가 연결되어 있고 각 스코프간에 변수를 참조할 수 있는 일종의 계층 구조를 의미한다.

어느 것이 더 확실한 정의인지는 아직 모르겠지만 2가지 정의 모두 틀린 정의는 아니라고 생각한다. 다만 관점에 따라 개념을 바라보는 정의가
나눠진 것이 아닐까 하는 생각이 든다.

스코프 체인이 어떻게 동작하는지 자세히 이해하려면 추가적인 개념이 필요한데 실행 컨텍스트(context)와
lexical scope 가 무엇인지 알 필요가 있다. 이 2가지를 먼저 다뤄보려고 한다.

## 실행 컨텍스트(context)

실행 컨텍스트는 소스 코드가 실행되는 환경을 의미한다. 실행 컨텍스트는 전역 영역에서 생성되는 전역 컨택스트, 함수가 실행될 때 생성되는 함수 실행 컨텍스트, eval 함수에 의해 생성되는 eval 실행 컨텍스트로 나뉜다.
eval 함수는 사용되는 일이 거의 없기 때문에, 컨텍스트는 크게 전역, 함수 컨텍스트 2가지라고 생각할 수 있다. 코드가 실행되고 함수가 실행됨에 따라 각 실행 컨텍스트가 생성되고 JavaScript 엔진의 콜 스택에 푸시(push)된다. 이후 콜 스택에 있는 컨택스트들을 하나씩 차례로 실행한다.

## Lexical Scope

실행중인 특정 코드 또는 함수가 있다고 가정하면 실행 컨텍스트를 포함하는 함수가 있고, 그 함수는 로컬 스코프를 형성할 것이다. 그리고 그 함수를 둘러싼 외부 코드가 있으며, 그 외부 코드들은 별도의 스코프를 형성할 것이다. lexical 스코프란 현재 실행 컨텍스트를 포함한 함수가 가지는 스코프에 대한 상태와 외부 코드가 가지는 외부 스코프의 상태를 의미한다. 쉽게 표현하면, 현재 실행 중인 코드 내부 및 외부 스코프와 변수에 대한 정보를 포함한다고 보면 된다. lexical environment 라고도 한다. lexical Scope 는 실행 컨택스트가 생성될 때 같이 생성된다.

lexical Scope 는 2가지 요소로 구분된다.

- environment record : 현재 정의되어 있는 lexical scope 내의 변수와 함수에 대한 정보를 포함하는 객체.
- 외부 환경에 대한 reference : 현재의 lexical scope 를 둘러싼 외부의 lexical scope 에 대한 참조를 포함하는 객체.

## Scope Chain 의 동작과정

다음 예시를 통해 변수를 찾는 과정을 살펴볼 수 있다.

```js
const globalVar = "globalVar";

function outer() {
  let localVar1 = "localVar1";
  inner1();
}

function inner1() {
  let localVar2 = "localVar2";
  inner2();
}

function inner2() {
  let localVar3 = "localVar3";
  console.log("print globalVar :", globalVar);
  console.log("print localVar1 :", localVar1);
  console.log("print localVar2 :", localVar2);
  console.log("print localVar3 :", localVar3);
}

outer();
```

코드가 실행되면서 실행 컨텍스트(EC)가 생성된다. 전역 EC, outer EC, inner1 EC, inner2 EC 이렇게 총 4가지의 실행 컨텍스트가 생성되어 콜 스택에 푸시된다. 푸시된 EC 는 하나씩 pop 되어 실행된다.

가장 마지막에 푸시된 inner2 EC 가 가장 먼저 실행된다. inner2() 함수 내에 있는 4가지 변수값을 찾아서 출력하는 과정에서 각 변수들의 값을 찾게 된다. inner2 내에 선언된 localVar3 를 제외한 나머지 변수들은 로컬 스코프(inner2 scope)에서 찾을 수 없으므로 외부 스코프에서 참조해야 한다.

다음으로 inner1 EC 가 실행된다. inner2 함수는 inner1 내부에 선언된 localVar2 변수를 참조할 수 있게 된다. 이어서 outer EC 가 실행되고 localVar1 변수의 참조가 가능해진다. 마지막으로 전역 EC 까지 거슬러 올라가고, 전역 변수인 globalVar 를 참조할 수 있게 된다.

스코프 체인을 통해 JavaScript 는 함수의 내부, 외부에 정의된 변수들을 참조해서 사용할 수 있다. 그리고 이러한 JavaScript 의 특징을 보고 클로저(closure) 라고 한다.

스코프에 대해 학습하고 글을 쓰면서 생각보다 내용이 많아서 글이 길어졌다. 내용도 많았지만 이해하기가 쉽지 않아서 여러 번 반복해서 학습을 하느라 시간도 많이 썼던 것 같다. 학습을 마치고 나니 JavaScript 코드가 어떻게 동작하는지 좀 더 잘 이해한 것 같아서 보람은 있었다. 이 주제는 앞으로 몇 번은 더 꼼꼼히 학습해서 완전히 습득하도록 해야겠다.
