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

- 이터러블 : 이터레이터를 리턴하는 [Symbol.iterator]() 를 가진 값
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
그리고 반환된 각 이터레이터도 [Symbol.iterator]() 로 자기 자신의 이터레이터를 반환하기 때문에 이터러블하다.  
그래서 다음 코드와 같이 반환한 이터레이터를 사용하여 또 순회를 할 수 있는 것이다.  

```js
for (const a of map.keys()) log(a);
for (const a of map.values()) log(a);
for (const a of map.entries()) log(a);
```

