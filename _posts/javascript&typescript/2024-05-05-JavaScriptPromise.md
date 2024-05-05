---
title: JavaScript Promise
description: ""
date: 2024-05-05 21:00:00 +09:00
categories: [Language, JavaScript]
tags: [JavaScript, 비동기 처리, Promise]
pin: true
math: true
mermaid: true
---

JavaScript 에서 비동기 작업 도중 순차적으로 처리해야 하는 작업을 처리하기 위해 콜백 함수를 사용할 수 있지만, 처리해야 할 작업이 많아질
경우 콜백 함수의 증가로 콜백 지옥이 발생할 수 있다. 이러한 문제를 방지면서도 비동기 작업을 하기 위한 몇 가지 기법들이 있다.

## **Promise**

Promise 는 JavaScript 에서 비동기 작업이 진행되었을 때 발생할 결과를 나타내고 결과값을 전달해 주는 객체이다. 얼핏 들었을 때는 무슨
의미인지 이해가 어려울 수 있다.

외부에서 데이터를 불러와서 순차적으로 처리하는 상황을 가정해 보자. 일반적으로 데이터 불러오기 요청은 항상 성공하지 않고 실패하는 경우도 존재한다.

```js
function fetchData(callback) {
  // 비동기적으로 데이터를 불러오는 시뮬레이션
  const isSuccess = Math.random() < 0.5; // 50% 확률로 성공 또는 실패

  setTimeout(function () {
    if (isSuccess) {
      const data = "데이터를 불러왔습니다.";
      callback(null, data); // 성공 시 null과 데이터 전달
    } else {
      const errorMessage = "데이터를 불러오는 중에 오류가 발생했습니다.";
      callback(errorMessage, null); // 실패 시 에러 메시지와 null 전달
    }
  }, 1000);
}

// 데이터를 불러와서 처리하는 함수
function processData(error, data) {
  if (error) {
    console.error("데이터 불러오기 실패:", error);
  } else {
    console.log("받은 데이터:", data);
    console.log("데이터를 가공하여 출력합니다.");
  }
}

// fetchData 함수를 호출하여 데이터를 불러온 후, processData 함수를 콜백으로 전달
fetchData(processData);
```

위 예시에서 데이터를 불러오는 작업이 가질 수 있는 결과는 성공 또는 실패 2가지이다. 성공하면 data 를, 실패하는 errorMessage 객체를 콜백 함수에
전달해 주고, 콜백 함수에서는 상황에 맞게 적절하게 처리한다.

Promise 객체도 마찬가지로 비동기 작업의 결과와 결과값을 포함하고 이를 전달해 주는 역할을 한다. Promise 를 이용하면 콜백 함수를
사용할 때 보다 더 깔끔하게 코드를 작성할 수 있다. 방금 전 데이터를 불러오는 예시를 Promise 를 이용하여 작성하면 다음과 같다.

```js
function fetchData() {
  return new Promise((resolve, reject) => {
    // 비동기적으로 데이터를 불러오는 시뮬레이션
    const isSuccess = Math.random() < 0.5; // 50% 확률로 성공 또는 실패

    setTimeout(function () {
      if (isSuccess) {
        const data = "데이터를 불러왔습니다.";
        resolve(data); // 성공 시 데이터 전달
      } else {
        const errorMessage = "데이터를 불러오는 중에 오류가 발생했습니다.";
        reject(errorMessage); // 실패 시 에러 메시지 전달
      }
    }, 1000);
  });
}

// 데이터를 불러와서 처리하는 함수
function processData(data) {
  console.log("받은 데이터:", data);
  console.log("데이터를 가공하여 출력합니다.");
}

// 실패 처리 함수
function handleFailure(error) {
  console.error("데이터 불러오기 실패:", error);
}

// fetchData 함수를 호출하여 데이터를 불러온 후, then 메서드를 통해 성공한 경우와 catch 메서드를 통해 실패한 경우를 처리
fetchData().then(processData).catch(handleFailure);
```

생소한 문법 때문에 더 복잡해 보일 수 있지만 알고 나면 그렇게 어렵지 않다. 우선은 Promise 객체의 3가지 상태에 대해 먼저 알 필요가 있다.
일반적으로 Promise 객체는 다음 3가지 상태를 가진다.

1. 대기(Pending) : 비동기 작업이 아직 완료되지 않은 상태. 초기 상태로 Promise 객체를 생성할 때는 대기 상태이다.
2. 이행(Fulfilled) : 비동기 작업이 성공적으로 완료된 상태. 이 때 Promise 객체는 작업 결과값을 가지고 있으며 이 값은 전달이 가능하다.
3. 거부(Rejected) : 비동기 작업이 실패한 상태. 작업이 실패하면 거부 상태가 되며 실패 원인을 나타내는 이유를 가지고 있다.

대기(Pending)의 경우 new Promise() 매서드 호출을 통해 Promise 를 생성하면, 그 객체는 대기 상태가 된다. Promise 객체를 생성할 때
콜백 함수를 파라미터로 넘겨줄 수 있고, 콜백 함수는 파라미터로 resolve 와 reject 를 갖는다.

```js
new Promise((resolve, reject) => {
  ...
})
```

Promise 를 생성한 후 resolve 를 실행하면 Promise 객체는 이행(Fulfilled) 상태가 된다. 이후 작업을 위해 resolve 를 이용하여 결과값을 전달해
줄 수 있다.

위의 예시에서 작업 성공 부분을 다시 살펴보면,

```js
...
return new Promise((resolve, reject) => {
  // 비동기적으로 데이터를 불러오는 시뮬레이션
  const isSuccess = Math.random() < 0.5; // 50% 확률로 성공 또는 실패
    setTimeout(function() {
      if (isSuccess) {
        const data = "데이터를 불러왔습니다.";
        resolve(data); // resolve 를 실행하여 이행 상태로 전환, 결과값 전달
      }
      ...
  }, 1000);
});

...

// fetchData 실행의 결과로 Promise 객체를 전달받음.
fetchData().then((res) => {
  // resolve 를 실행하여 전달받은 결과값 출력
  console.log('resolved value : ', res);
})
```

만약 작업이 실패할 경우 reject 를 실행하면 실패(Rejected) 상태가 된다. 이후 에러 핸들링을 위해 에러 메시지를 전달할 수 있다.

위의 예시에서 작업 실패 부분을 다시 살펴보면,

```js
...
return new Promise((resolve, reject) => {
  // 비동기적으로 데이터를 불러오는 시뮬레이션
  const isSuccess = Math.random() < 0.5; // 50% 확률로 성공 또는 실패
    setTimeout(function() {
      if (isSuccess) {
        ...
      }
      else {
        const errorMessage = "데이터를 불러오는 중에 오류가 발생했습니다.";
        reject(errorMessage); // reject 를 이용하여 실패 상태로 전환, 에러 메시지 전달
      }
      ...
  }, 1000);
});

...

// fetchData 실행의 결과로 Promise 객체를 전달받음.
fetchData()
.then((res) => {
  ...
})
.catch((error) => {
  // reject 를 실행하여 전달받은 에러 메시지 출력
  console.log(error)
});
```

Promise 객체의 동작 흐름을 그림으로 나타내면 다음과 같다.

![image](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/promises.png)

<div style="font-size:15px; text-align:center">출처 : https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise</div>

<br/>
위 그림을 보면 최초 Promise 를 생성 후, 이행 또는 실패 상태로 전환되고 settled 된 이후에 다시 Promise 객체를 반환한다. 생성된 Promise
객체는 다시 이행, 실패 상태에 따라 이후 작업을 진행한다. 여기서 Promise 객체의 또다른 특징을 알 수 있다. Promise 객체는 then() 매서드를
호출하면 새로운 Promise 객체를 생성하여 반환한다.

이 특징을 활용하여 여러 개의 Promise 객체를 연결하여 연속적인 작업을 처리할 수 있다. 다음은 데이터를 불러와서 직접 처리한 후 최종 결과를
출력하는 예제이다.

```js
// 비동기적으로 데이터를 불러오는 함수
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const data = "데이터를 불러왔습니다.";
      resolve(data);
    }, 1000);
  });
}

// fetchData를 한 번 호출하여 데이터를 불러온 후, 이후에 then 메서드를 연속해서 사용하여 연속적인 작업을 수행
fetchData()
  .then((data) => {
    // 데이터를 받아와서 직접 처리
    const processedData = data.toUpperCase();
    return processedData;
  })
  .then((processedData) => {
    // 또 다른 처리
    const finalResult = "최종 결과: " + processedData;
    console.log(finalResult);
  })
  .catch((error) => {
    console.error("오류 발생:", error);
  });
```

Promise 를 이용하면 콜백 지옥을 겪지 않고도 비동기 작업을 처리할 수 있고 코드 관리에도 편리하다. 그러나 처리해야 할 작업이 많아지면 Promise
로도 감당하기 힘든 상황이 발생할 수 있다.

데이터를 불러온 후 여러 가지 처리를 해야 하는 다음의 상황을 가정하면,

```js
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
```

위 예시는 각각의 then 매서드 안에서 다음 작업을 수행하기 위해 새로운 Promise 객체를 반환한다. 이렇게 되면 then 매서드를 중첩하여 사용하게
되어 코드의 가독성이 떨어지고, 코드가 복잡해진다. 이런 상황을 "Promise 지옥" 이라고 표현한다. 콜백 지옥과 마찬가지로 Promise 지옥이
생기는 것도 바람직하지 않으므로 다른 방법을 통해 코드의 가독성을 높이는 작업이 필요하다. 여러 가지 방법이 있지만 async/await 를 이용한
방법이 주로 선호된다. 이 방법에 대해서는 다음 글에서 정리할 예정이다.
