---
title: JavaScript Async/Await
description: ""
date: 2024-05-10 20:43:25 +09:00
categories: [SW Development / CS, JavaScript]
tags: [JavaScript, 비동기 처리, Async/Await]
pin: true
math: true
mermaid: true
---

비동기 작업을 진행할 때 Promise 객체를 이용하면 콜백 함수를 이용할 때 보다 더 편하게 작업을 처리할 수 있다. 그러나 처리해야 할 작업이
많아지면 Promise 객체를 사용해도 코드가 복잡해지고 더 간결한 표현식이 필요해진다. async/await 문법은 복잡한 코드를 더 간결하게 작성할
수 있도록 해 준다.

## **기본 문법**

Promise 객체를 이용하여 데이터를 불러오는 경우 코드는 다음과 같다.

```js
function fetchData() {
  return new Promise((resolve, reject) => {
    resolve(data);
  });
}

// 비동기 작업 실행
function execute() {
  fetchData()
    .then(res => console.log("result :", res));
    .catch(error => console.log("error :", error));
}

execute();
```

이 코드를 async/await 를 이용하여 작성하면,

```js
function fetchData() {
  return new Promise((resolve, reject) => {
    resolve(data);
  });
}

// 비동기 작업 실행
async function execute() {
  try {
    let res = await fetchData();
    console.log("result :", res);
  } catch (error) {
    console.log("error :", error);
  }
}
```

딱 보기에도 코드가 간단해진 것을 볼 수 있다. 몇 가지 달라진 점이 있는데 함수 앞에 async 키워드가 붙은 것, then, catch 문 사용 없이
await 키워드를 fetchData 앞에 붙여줌으로써 결과값을 받았다는 점이다.

async 키워드는 비동기 작업을 진행할 함수 앞에 붙여줘야 한다. async 가 붙은 함수는 Promise 객체를 반환하고, 함수 내에서 await 를 사용할
수 있게 해 준다. 그리고 try/catch 문을 사용하여 작업 중 발생하는 에러를 캐치할 수 있다.

await 키워드가 있으면 함수 내에서 해당 작업이(이 경우엔 fetchData) 처리될 때까지 대기하고 완전히 처리되면 다음 작업으로 넘어간다. await 문법은 작업을 "동기적"으로 처리할 수 있도록 해 준다. 물론 JavaScript 런타임 전체가 작업이 중단되는 것은 아니다. Promise 를 처리하는 동안 런타임에서 다른 작업을 비동기적으로 처리하기 때문에 성능 문제가 발생하지는 않는다.

async/await 는 순차적으로 실행해야 할 작업이 많을 때 매우 편리하다. 다음 Promise 객체를 사용한 비동기 처리 예시를 보면 코드가 길고 복잡하다.

```js
function execute() {
  fetchData()
    .then((data1) => {
      // 첫 번째 데이터를 처리
      return processData1(data1);
    })
    .then((processedData1) => {
      // 첫 번째 가공된 데이터를 기반으로 다음 데이터를 가져옴
      return fetchData2(processedData1);
    })
    .then((data2) => {
      // 두 번째 데이터를 처리
      return processData2(data2);
    })
    .then((processedData2) => {
      // 두 번째 가공된 데이터를 기반으로 다음 데이터를 가져옴
      return fetchData3(processedData2);
    })
    .then((data3) => {
      // 세 번째 데이터를 처리
      return processData3(data3);
    })
    .then((finalResult) => {
      // 최종 결과를 출력
      console.log("최종 결과:", finalResult);
    })
    .catch((error) => {
      console.error("오류 발생:", error);
    });
}

execute();
```

이 코드를 async/await 문을 사용하여 다시 작성하면,

```js
async function execute() {
  try {
    const data1 = await fetchData();
    const processedData1 = await processData1(data1);

    const data2 = await fetchData2(processedData1);
    const processedData2 = await processData2(data2);

    const data3 = await fetchData3(processedData2);
    const finalResult = await processData3(data3);

    console.log("최종 결과:", finalResult);
  } catch (error) {
    console.error("오류 발생:", error);
  }
}

execute();
```

코드가 상당히(?) 깔끔해진 것을 볼 수 있다.

## **async 함수의 특징**

async 함수는 선언될 때 AsyncFunction 객체를 생성한다. async 함수가 호출될 때 마다 새로운 Promise 객체를 반환하는데, 이때 반환된 Promise 객체는 이전 작업에 대한 이행(fulfilled) 또는 실패(rejected)의 상태를 가지고 있다. resolved 된 결과값은 await 표현식에서 반환값으로서 사용된다. 만약 반환값이 Promise 객체가 아니라 할지라도 암시적으로 Promise 객체 형태로 감싸져서 반환된다.

다음 예시를 통해 쉽게 이해할 수 있다.

```js
// execute1 과 execute2 는 비슷하다.
async function execute1() {
  return 1;
}

async function execute2() {
  return Promise.resolve(1);
}
```

그런데 한 가지 주의해야 할 점이 있다. async 함수에서 Promise 객체가 아닌 값이 Promise 객체 형태로 반환된다고 해서, 실제 Promise.resolve() 의 값과 동일하지는 않다.

다음 예시를 보면,

```js
const p = new Promise((res, rej) => {
  res(1);
});

async function asyncReturn() {
  return p;
}

function basicReturn() {
  return Promise.resolve(p);
}

console.log(p === basicReturn()); // true
console.log(p === asyncReturn()); // false
```

asyncReturn 함수는 p 라는 Promise 객체를 반환한다. 그러나 asyncReturn 함수는 비동기 함수이므로 실행 결과는 Promise 객체가 아닌 Promise 객체가 가진 값으로 resolve 된다. 반면 basicReturn 함수는 Promise.resolve 를 사용하여 p 를 반환한다. 이 함수는 항상 p 라는 Promise 객체를 반환하므로, 반환 값은 p 자체가 된다. 따라서 p 와 basicReturn 은 같은 값이고, p 와 asyncReturn 은 다른 값이 된다.

## **await 의 특징**

await 는 Promise.then 처럼 동작한다. 다음의 예시를 보면,

```js
// execute1 과 execute2 는 비슷하다.
async function execute1() {
  await 1;
}

function execute2() {
  return Promise.resolve(1).then(() => undefined);
}
```

execute1 의 경우 await 가 .then 매서드를 호출하는 식으로 동작한다. 객체가 Promise 가 아니더라도 .then 매서드를 호출한다.

이전에 Promise 와 async/await 문을 공부해 본 적 있었지만 깊게 공부하니 어려운 내용이 많았다. Promise 와 await 의 동작에 대해서는 아직 좀 더 공부해 봐야할 것 같다. 좀 더 학습해 보고 수정 및 보완을 해 나가야 할 것 같다.
