# ES6+

## 함수형 자바스크립트 기본

### 평가와 일급

- 평가 : 코드가 계산되어 값을 만드는 것
- 일급
	- 값으로 다룰 수 있다.
	- 변수에 담을 수 있다.
	- 함수의 인자로 사용될 수 있다.
	- 함수의 결과로 사용될 수 있다.

### 일급 함수

- 함수를 값으로 다룰 수 있다.
- 조합성과 추상화의 도구

```js
const f1 = () => () => 1;
log(f1()); // () => 1;

const f2 = f1();
log(f2); // () => 1;
log(f2()); // 1;
```


### 고차 함수

함수를 값으로 다루는 함수

#### 함수를 인자로 받아서 실행하는 함수

```js
const apply1 = f => f(1);
const add2 = a => a + 2;
log(apply1(add2)); // 3
log(apply1(a => a - 1)); // 0
```

```js
const times = (f, n) => {
  let i = -1;
  while (++i < n) f(i);
}

times(log, 3); // 0 1 2
times(a => log(a + 10), 3); // 10 11 12
```

#### 함수를 만들어 리턴하는 함수 (클로저를 만들어 리턴하는 함수)

- 클로저 : 함수가 만들어질 때의 환경을 기억하는 함수

```js
const addMaker = a => b => a + b;
const add10 = addMaker(10);
log(add10(5));
log(add10(10));
```


---

## ES6에서의 순회와 이터러블

### ES6에서의 리스트 순회

```js
const list = [1, 2, 3];
const str = 'abc';

for (const a of list) {
  log(a);
}
for (const a of str) {
  log(a);
}
```


### Array, Set, Map을 통해 알아보는 이터러블/이터레이터 프로토콜

Array, Set, Map 모두 `for...of` 문으로 순회가 가능하다.  

다만 기존 ES5의 for문에서 사용하던 방법인, 인덱스를 1씩 증가시키면서 순회하는 방법은 ES6의 for...of문에서 내부적으로 쓰이지 않는 다는 것을 알 수 있다.  
왜냐하면 Set과 Map은 인덱스로 각 요소에 접근하면서 순회할 수 있는 것이 아니기 때문이다.  

```js
const arr = [1, 2, 3];
log(arr[0]); // 1

const set = new Set([1, 2, 3]);
log(set[0]); // undefined

const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
log(map[0]); // undefined
```


#### 이터러블/이터레이터 프로토콜

- 이터러블 : 이터레이터를 리턴하는 `[Symbol.iterator]()` 를 가진 값
- 이터레이터 : { value, done } 객체를 리턴하는 next() 를 가진 값
- 이터러블/이터레이터 프로토콜 : 이터러블을 for...of, 전개 연산자 등과 함께 동작하도록 한 규약

```js
const arr = [1, 2, 3];

let iterator = arr[Symbol.iterator](); // array는 이터러블이다.
iterator.next() // {value: 1, done: false}
iterator.next() // {value: 2, done: false}
iterator.next() // {value: 3, done: false}
iterator.next() // {value: undefined, done: true}
```

map.keys() 는 key들만 모아놓은 이터레이터를 반환하는 함수이다.  
map.values(), map.entries()도 마찬가지로 각각 이터레이터를 반환하는 함수이다.  
그리고 반환된 각 이터레이터도 `[Symbol.iterator]()` 로 자기 자신의 이터레이터를 반환하기 때문에 이터러블하다.  
그래서 다음 코드와 같이 반환한 이터레이터를 사용하여 또 순회를 할 수 있는 것이다.  

```js
for (const a of map.keys()) log(a);
for (const a of map.values()) log(a);
for (const a of map.entries()) log(a);
```


### 사용자 정의 이터러블

`well-formed 이터레이터` 는 next() 로 일정 부분 진행하다가 순회를 진행할 수 있어야 한다.  
그러기 위해서는 이터레이터 자신도 `[Symbol.iterator]()` 를 실행시켰을 때 자기 자신인 이터레이터를 반환할 수 있어야 한다.  

사용자 정의 이터러블을 만들 때도 이 점을 인지하고 작성하면 다음과 같이 만들 수 있다.  

```js
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
      [Symbol.iterator]() { return this; } // 자기 자신도 Symbol.iterator로 가지고 있는다.
    }
  }
};

let iterator = iterable[Symbol.iterator]();
log(iterator.next()); // 3
for (const a of iterator) log(a); // 2 1
```


### 전개 연산자

전개 연산자도 마찬가지로 이터러블/이터레이터 프로토콜을 사용한다.

```js
const a = [1, 2];
log([...a, ...arr, ...set, ...map.keys()]); 
```


---

## 제너레이터와 이터레이터

- 제너레이터 : 이터레이터이자 이터러블을 생성하는 함수
	- **제너레이터라는 문장을 통해, 어떤 값이든 순회할 수 있는 이터러블로 만들 수 있다**.  

```js
function *gen() {
  yield 1;
  yield 2;
  yield 3;
  return 100; // done이 true일 때 value 값
}

let iter = gen();
log(iter[Symbol.iterator]() == iter); // true
log(iter.next()); // { value: 1, done: false }
log(iter.next()); // { value: 2, done: false }
log(iter.next()); // { value: 3, done: false }
log(iter.next()); // { value: 100, done: true }
```

- `yield` : value, done 두 속성을 가진 IteratorResult 객체를 반환한다. 
	- yield 표현식에서 중지되면, 제너레이터의 next() 메서드가 호출될 때까지 제너레이터의 코드 실행이 중지된다.
	- [yield - JavaScript | MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/yield)


### odds

여러가지 제너레이터를 활용해서 홀수만 출력하는 제너레이터를 만들어 보자.  

```js
function *infinity(i = 0) { // 무한루프를 썼지만 yield가 있기 때문에 next() 호출 범위까지만 실행되어 안전하다.
  while (true) yield i++;
}

function *limit(l, iter) { // iter를 받아서 실행하다가 l과 같은 값이 나오면 멈춘다.
  for (const a of iter) {
    yield a;
    if (a == l) return;
  }
}

function *odds(l) { // 위 두 제너레이터를 활용하여 만든 홀수 제너레이터
  for (const a of limit(l, infinity(1))) {
    if (a % 2) yield a;
  }
}

for (const a of odds(40)) log(a);
```


### for...of, 전개 연산자, 구조 분해, 나머지 연산자

```js
log(...odds(10)); // 1 3 5 7 9
log([...odds(10), ...odds(20)]); // [1, 3, 5, 7, 9, 1, 3, 5, 7, 9, 11, 13, 15, 17, 19]

const [head, ...tail] = odds(5);
log(head); // 1
log(tail); // [3, 5]

const [a, b, ...rest] = odds(10);
log(a); // 1
log(b); // 3
log(rest); // [5, 7, 9]
```

---


