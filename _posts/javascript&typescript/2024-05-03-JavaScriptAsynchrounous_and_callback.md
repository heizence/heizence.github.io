---
title: JavaScript 비동기 처리, 콜백 함수
description: ""
date: 2024-05-03 20:00:00 +09:00
categories: [Language, JavaScript]
tags: [JavaScript, 비동기 처리, 콜백]
pin: true
math: true
mermaid: true
---

JavaScript 는 싱글 스레드 언어이지만 JavaScript 런타임이 비동기로 동작하기 때문에 비동기 작업을 통해 효율적으로 작업을 처리할 수 있다.
그러나 비동기 처리를 이용하여 작업을 할 때 주의해야 할 점이 있다. 몇몇 작업의 경우 순차적으로 진행되어야 한다.

## **일반적인 비동기 처리의 문제점**

예를 들면 외부(서버, 데이터베이스 등)에서 데이터를 불러와서 작업을 해야 하는 상황을 가정해 보자.

```js
function getData() {
  let data;
  // 데이터를 불러오는 데 3초가 걸린다고 가정
  setTimeout(() => {
    data = 100;
  }, 3000);

  return data;
}

// 데이터 출력
let result = getData();
console.log("result : ", result);
```

위 코드를 실행하면 다음 결과가 나온다.

```
result : undefined
```

비동기 처리에 대해 제대로 이해했다면 결과가 왜 이렇게 나오는지 알 것이다. 코드가 실행되다가 setTimeout 함수가 Task Queue 에 들어가고
3초 후에 콜백 함수가 실행된다. 3초 동안 대기하는 동안 나머지 코드가 실행되는데 data 변수에 값이 할당되지 않은 상태이기 때문에 data 가 undefined 로 출력된다.
이런 문제를 예방하기 위해서 작업을 동기적으로 처리를 해야할 때가 있다. 가장 기본적으로 콜백 함수를 이용하는 방법이 있다.

### **콜백(callback) 함수란?**

콜백 함수는 다른 함수에 파라미터로 전달되는 함수를 의미한다. 전달된 콜백 함수는 전달받은 함수의 내부에서 실행된다. 콜백 함수가 언제, 어떻게 실행될지는 콜백을 전달받는 함수에 달려있다.

## **콜백 함수를 이용한 비동기 처리**

데이터를 불러와서 출력하는 첫 번째 예시에서 데이터가 할당되기 전에 작업이 처리되는 문제를 다음과 같은 방식으로 개선할 수 있다.

```js
function getData(callback) {
  // 3초 후에 데이터 값을 받아온 후, 콜백 함수에 값을 넘겨줌.
  setTimeout(() => {
    let data = 200;
    callback(data);
  }, 3000);
}

getData(function (data) {
  let result = data;
  console.log("result : ", result);
});
```

위 코드를 실행하면 다음 결과가 나온다.

```
result : 200
```

getData 에 특정 작업을 진행하는 콜백 함수를 전달해 주고, 3초가 지난 후에 콜백 함수를 실행함으로써 data 값이 할당된 채로 문제없이 작업을 진행할 수 있다. 콜백 함수는 비동기 함수에서 작업 결과를 전달받아서 처리하는 데 사용되어 작업 순서를 맞출 수 있다.

그런데 만약 순차적으로 처리해야 할 작업들이 많아지면 전달해야 할 콜백 함수의 수도 많아져서 코드가 복잡해지게 된다. 데이터를 불러온 후 추가로 처리해야 할 작업이 몇 가지 더 있는 다음 예시를 보자.

```js
// 데이터 불러오기
function getData(callback) {
  setTimeout(function () {
    const data = "데이터를 불러왔습니다.";
    callback(data);
  }, 3000);
}

// 데이터 처리하기
function processData(data, callback) {
  setTimeout(function () {
    const processedData = "가공된 데이터";
    callback(processedData);
  }, 1000);
}

// 추가로 처리할 작업 1
function task1(processedData, callback) {
  setTimeout(function () {
    const result = "Task 1 완료";
    callback(result);
  }, 1000);
}

// 추가로 처리할 작업 2
function task2(result, callback) {
  setTimeout(function () {
    const result = "Task 2 완료";
    callback(result);
  }, 1000);
}

// 추가로 처리할 작업 3
function task3(result, callback) {
  setTimeout(function () {
    const result = "Task 3 완료";
    callback(result);
  }, 1000);
}

// 추가로 처리할 작업 4
function task4(result, callback) {
  setTimeout(function () {
    const result = "Task 4 완료";
    callback(result);
  }, 1000);
}

// 데이터를 불러온 후에 각각의 작업을 수행하고 결과를 출력
getData(function (data) {
  processData(data, function (processedData) {
    task1(processedData, function (result1) {
      task2(result1, function (result2) {
        task3(result2, function (result3) {
          task4(result3, function (finalResult) {
            console.log("최종 결과:", finalResult);
          });
        });
      });
    });
  });
});
```

콜백 함수 내에 콜백 함수를 계속 사용함으로서 코드가 복잡해져 가독성이 떨어지고 추후 유지 보수가 어려워진다. 이런 상황을 "콜백 지옥" 이라고 표현한다. JavaScript 에서는 콜백 지옥을 피하기 위한 기법이 존재하는데 다음 글에서 자세히 다뤄보려고 한다.
