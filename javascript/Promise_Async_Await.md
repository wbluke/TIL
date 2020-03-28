# Promise / Async / Await
#TIL/javascript

---

## 비동기 처리
- Ajax나 setTimeout() 등의 비동기 API를 일반적인 동기 방식으로 사용하면 원하지 않는 결과나 실행 순서를 불러올 수 있다.
- 그래서 js에서는 콜백 함수로 비동기 API의 작업이 마무리 되었을 때의 할 일을 클라이언트가 지정할 수 있다.
- 하지만 과도한 콜백의 연속 사용은 콜백 지옥을 낳기도 한다.

```js
$.get('url', function(response1) {
  func1(response1, function(response2) {
    func2(response2, function(response3) {
      func3(response3, function(result) {
        console.log(result);
      });
    });
  });
});
```


## Promise

### 정의

- 약속. 미래의 어느 시점에 결과물을 반환하겠다는 약속.

### Promise의 세 가지 상태

* Pending(대기) : 비동기 처리 로직이 아직 완료되지 않은 상태
* Fulfilled(이행, 완료) : 비동기 처리가 완료되어 프로미스가 결과 값을 반환해준 상태
* Rejected(실패) : 비동기 처리가 실패하거나 오류가 발생한 상태

```js
function getData() {
  return new Promise(function(resolve, reject) {
    $.get('url 주소/products/1', function(response) {
      if (response) {
        resolve(response); // 정상적인 이행 상태
      }
      reject(new Error("Request is failed")); // 실패 상태
    });
  });
}

getData().then(function(data) {
  console.log(data); // 정상적인 이행 상태
}).catch(function(err) {
  console.error(err); // 실패 상태
});
```

## Async & Await

Async & Await 방식은 비동기 처리 시 콜백함수를 넘겨준다는 개념을 조금 더 풀어낸 방식이다.  
콜백 함수를 넘기는 방법 대신, 일반적인 동기 프로그래밍을 하듯이 콜백함수의 내용을 `await`이 걸려 있는 비동기 함수(Promise를 반환하는 함수)에 이어서 작성한다.  

```js
async function method() {
  const response = await 비동기메소드('url'); // 원래는 두번 째 인자로 콜백 함수를 넘겼다.
  console.log(response); // 원래는 콜백 함수로 처리하던 내용
}
```

- Async & Await를 사용해 동기적인 방식처럼 아주 편하게 비동기 처리를 진행할 수 있다.
- Async & Await 방식의 예외 처리는 try-catch 문으로 처리하면 된다.

## 참고

장기효님의 Promise/Async/Await 시리즈
- https://joshua1988.github.io/web-development/javascript/javascript-asynchronous-operation/
- https://joshua1988.github.io/web-development/javascript/promise-for-beginners/
- https://joshua1988.github.io/web-development/javascript/js-async-await/
