---
title: JavaScript Scope
description: ""
date: 2024-05-17 13:43:25 +09:00
categories: [SW Development / CS, JavaScript]
tags: [JavaScript, Scope]
pin: true
math: true
mermaid: true
---

## Scope 란?

Scope(스코프)는 JavaScript 뿐 아니라 다른 언어에서도 언급되는 개념이다. 본래의 단어 뜻 그대로 범위라는 의미를 가지고 있는데 어떤 범위를
의미할까? 코드를 작성하면 여러 가지 변수가 존재하고 그 변수들을 참조해야 할 일이 있다. 그런데 변수가 선언되었다고 해서 무작정 참조할 수는
없고 변수가 선언된 위치(또는 context)에 따라 참조 가능 여부가 달라진다. 이 때 변수가 선언된 위치가 스코프에 해당된다. 정확히 정의하면,
스코프는 변수에 접근하여 참조할 수 있는 영역의 범위를 의미한다.

이해를 위해 간단한 예시를 살펴보면,

```js
function exampleFunction() {
  const x = "declared inside function"; // x 는 함수 내에서만 참조 가능.
  console.log("Inside function");
  console.log(x);
}

console.log(x); // 에러 발생. x 를 함수 밖에서 참조할 수 없음.
```

상수 x 를 함수 내에서 정의한 후, 함수 안쪽과 바깥쪽에서 참조를 시도했다. 뒤에서 설명하겠지만, x 는 함수 내에서만 유효한 함수 스코프에 포함되어
있기 때문에 함수 바깥쪽에서는 참조를 할 수 없다. 함수 바깥쪽에서 적용되는 스코프는 전역 스코프이다.

## Scope 의 종류

### 1.전역(Global) 스코프

전역 스코프는 코드가 실행되는 전체 영역에서 접근 가능한 스코프이다. 전역 스코프에서 변수가 선언되면 그 변수는 함수 내부, 조건문, 루프 등 어디에서든 참조할 수 있다.

다음 예시를 보면,

```js
var globalX = "x";

console.log("print x in global :", x);

function execute() {
  console.log("print x in function :", x);
}

if (true) {
  console.log("print x in conditional statement :", x);
}
```

변수 globalX 는 위치에 관계없이 접근할 수 있다. 전역 스코프에 선언된 변수를 전역 변수라고 한다.

얼핏 보면 전역 스코프에 모든 변수를 선언하면 문제가 없을 것 같지만 그렇지 않다. 변수는 실행 과정 중 값이
변할 수 있기 마련이다. 전역 변수는 어디에서나 접근 및 수정이 가능하므로 그만큼 변화에 민감하고 취약하다.
사용자가 의도치 않게 변수를 변경할 수 있고 이는 에러로 이어질 수 있다.

전역 변수는 보통 실행 전반에 걸쳐 사용될 필요가 있을 때 선언하는 것이 일반적이다. 만약 특정 상황, 특정 구간에서만 사용되는 변수의 경우, 전역 변수를 선언하는 만큼 메모리 자원이 소비되고 관리해야 할 변수의 수가 많아지므로 오히려 비효율적이다.

### 2.로컬 스코프

로컬 스코프는 특정 영역에서만 유효한 스코프로, 외부 환경의 영향을 받지 않는다. 로컬 스코프에 선언된 변수는 선언된 영역 내에서만 접근할 수 있다.

다음 예시를 보면,

```js
function execute() {
  var localX = "localX
  console.log("print localX in function :", localX)
}

console.log("print localX in global :", x )  // 에러 발생
```

함수 execute 내에서 선언된 localX 는 함수 내에서만 유효한 로컬 스코프를 가지며 함수 밖에서는 접근할 수 없다. 로컬 스코프에서 선언된 변수를 로컬 변수라고 한다. 로컬 변수는 변수를 특정 영역에서만 사용하는 것으로 제한하고 외부의 접근으로부터 보호할 때 유용하다.

### 3.블록(Block) 스코프

블록 스코프는 로컬 스코프와 비슷하다. 차이점이 있다면 로컬 스코프가 함수에 의해 정의되는 반면, 블록 스코프는 중괄호({}) 로 선언된 블록에 의해 정의되며, 특정 블록 내에서만 유효하다.

다음 예시를 보면,

```js
function execute() {
  var localX = "localX";
  if (true) {
    var blockX = "blockX";
    console.log("print blockX in block :", blockX);
  }
  console.log("print blockX in function local :", blockX); // 에러 발생
}

console.log("print blockX in global :", blockX); // 에러 발생
```

조건문 내에서 선언된 blockX 는 중괄호로 선언된 블록 내에서만 유효한 블록 스코프를 가지며 블록 밖에서는 접근할 수 없다. 블록 스코프는 if, for, while 문과 같은 코드 블록에서 주로 형성된다.

지금까지의 예제들을 보면 스코프 내에 정의된 변수들을 판별하고 접근하는 것이 간단해 보이지만 엄연한 과정이 존재한다. JavaScript 에서 여러 스코프가 중첩되어 있는 환경에서 특정 변수를 찾아내는 과정을 스코프 체인이라고 한다. 스코프 체인은 내용이 많고 연관된 개념들도 많아서 별도로 학습을 한 후에 다뤄볼 예정이다.
