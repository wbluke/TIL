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

## map, filter, reduce

- 사용할 예시 데이터

```js
const products = {
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 }
}
```


### map

#### 이터러블 프로토콜을 따른 map 의 동작 방식

```js
const map = (f, iter) => { // 함수를 받아서 사용하는 고차함수
  let res = [];
  for (const a of iter) {
    res.push(f(a));
  }
  return res;
};

log(map(p => p.name, products));
log(map(p => p.price, products));
```

#### 이터러블 프로토콜을 따른 map의 다형성

```js
log(document.querySelectorAll('*').map(el => el.nodeName)); // error
```

위 querySelectorAll은 nodeList를 반환하는 웹 브라우저의 함수인데, nodeList는 array를 상속받지 않았기 때문에 map 함수가 존재하지 않아 에러가 난다.  

하지만 아래와 같이 방금 만든 map 함수를 사용하면, nodeList가 `이터러블 프로토콜`을 따르고 있기 때문에 정상 동작하게 된다.

```js
log(map(el => el.nodeName, document.querySelectorAll('*')); // 정상 동작
```

다시 말해서, 이터러블 프로토콜을 따르는 모든 함수는 우리가 만든 map의 대상이 될 수 있다. generator의 결과도 이터러블하기 때문에 마찬가지로 정상 동작한다.  

```js
function *gen() {
  yield 2;
  yield 3;
  yield 4;
}

log(map(a => a * a, gen()); // [4, 9, 16]
```

Map 자료구조도 이터러블 프로토콜을 따르기 때문에 다음과 같은 예제도 가능하다.  

```js
let m = new Map();
m.set('a', 10);
m.set('b', 20);

log(new Map(map(([k, a]) => [k, a * 2], m))); // value를 2배 한 새로운 Map
```


### 이터러블 프로토콜을 따른 filter

```js
const filter = (f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
};

log(...filter(p => p.price < 20000, products));
log(filter(n => n % 2, [1, 2, 3, 4])); // [1, 3]
log(filter(n => n % 2, function *() { // [1, 3, 5]
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
} ()));
```


### 이터러블 프로토콜을 따른 reduce

```js
const reduce = (f, acc, iter) => {
  if (!iter) {  // 초기값 acc가 없는 경우
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }

  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

const add = (a, b) => a + b;
log(reduce(add, 0, [1, 2, 3, 4, 5])); // 15

log(reduce(add, [1, 2, 3, 4, 5])); // 15, 초기값은 optional하도록
```

- 조금 더 복잡한 경우 (products 예제)

```js
log(
  reduce(
    (total_price, product) => total_price + product.price, 
    0, 
    products)); // 105000
```


### map + filter + reduce 중첩 사용과 함수형 사고

```js
const add = (a, b) => a + b;

log(
  reduce(
    add, 
    map(p => p.price, 
      filter(p => p.price < 20000, products)))); // 30000, 2만원 미만의 상품 가격을 전부 더한 값

log(
  reduce(
    add, 
    filter(n => n >= 20000, 
      map(p => p.price, products))));
```

- 함수형 사고로 코드를 작성할 때는 단계별로 `평가`된 형태의 값을 생각하면서 코드를 작성하면 편하다.
	- 예를 들어, 위 예제에서 reduce, map, filter를 해야하는데, 먼저  `reduce(add, [1, 2, 3, 4])` 같이 어떤 평가된 숫자 배열이 있고, 해당 배열에 대한 reduce 함수를 먼저 작성한 후, 그 다음 단계로 숫자 배열을 뽑아내기 위한 map 함수를 작성하는 식이다.

---

## 코드를 값으로 다루어 표현력 높이기

### go, pipe

- go : 함수의 리스트를 차례로 실행시켜서 즉시 평가된 값을 내는 함수
- pipe : 함수들을 합성한 함수를 리턴하는 함수

```js
const go = (...args) => reduce((a, f) => f(a), args);
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);

go(
  0,
  a => a + 1,
  a => a + 10,
  a => a + 100,
  log); // 111

const f = pipe(
  a => a + 1,
  a => a + 10,
  a => a + 100);

log(f(0)); // 111

const f2 = pipe(
  (a, b) => a + b,
  a => a + 10,
  a => a + 100);

log(f2(0, 1)); // 111
```

- go를 사용하여 읽기 좋은 코드 만들기

```js
// 기존 코드
log(
  reduce(
    add, 
    map(p => p.price, 
      filter(p => p.price < 20000, products))));

go(
  products,
  products => filter(p => p.price < 20000, products),
  products => map(p => p.price, products),
  prices => reduce(add, prices),
  log);
```

### curry

- 함수를 값으로 다루면서 받아두었던 함수를 나중에 평가시키는 함수

```js
const curry = f => 
  (a, ..._) => _.length ? f(a, ..._) : (..._) => f(a, ..._);

const mult = curry((a, b) => a * b);
log(mult(3)(2)); // 6

const mult3 = mult(3);
log(mult3(10)); // 30
log(mult3(5)); // 15
```

- go+curry 를 사용하여 읽기 좋은 코드 만들기

```js
// 기존 코드
go(
  products,
  products => filter(p => p.price < 20000, products),
  products => map(p => p.price, products),
  prices => reduce(add, prices),
  log);

// 기존 map, filter, reduce를 curry로 감싼 후
go(
  products,
  products => filter(p => p.price < 20000)(products),
  products => map(p => p.price)(products),
  prices => reduce(add)(prices),
  log);

// 위 과정이 아래와 같이 바뀐다. 좀 소름...
go(
  products,
  filter(p => p.price < 20000),
  map(p => p.price),
  reduce(add),
  log);
```

- 함수 조합으로 함수 만들기

```js
const total_price = pipe(
  map(p => p.price),
  reduce(add));

const base_total_price = predi => pipe(
  filter(predi),
  total_price);

// 사용
go(
  products,
  base_total_price(p => p.price < 20000),
  log);
```

---

## 장바구니 예제

```js
const products = {
  { name: '반팔티', price: 15000, quantity: 1 },
  { name: '긴팔티', price: 20000, quantity: 2 },
  { name: '핸드폰케이스', price: 15000, quantity: 3 },
  { name: '후드티', price: 30000, quantity: 4 },
  { name: '바지', price: 25000, quantity: 5 }
};

const add = (a, b) => a + b;

const sum = curry((f, iter) => go(
  iter,
  map(f),
  reduce(add));

const total_quantity = sum(p => p.quantity);
const total_price = sum(p => p.price * p.quantity);


```

---

## 지연성

### range와 느긋한 L.range

#### range

```js
const add = (a, b) => a + b;

const range = l => {
  let i = -1;
  let res = [];
  while (++i < l) {
    res.push[i];
  }
  return res;
};

let list = range(4);
log(list); // [0, 1, 2, 3]
log(reduce(add, list)); // 6
```

#### 느긋한 L.range

```js
const L = {};
L.range = function *(l) {
  let i = -1;
  while (++i < l) {
    yield i;
  }
};

let list = L.range(4); 
log(list); // iterator 출력
log(reduce(add, list)); // 6
```

- L.range에서는 결과물이 이터레이터이기 때문에 실제로 해당 이터레이터를 사용하기 전까지는 값들을 가져오지 않는다.
	- 배열을 만들지 않고 Lazy하게 동작한다.
	- 우리가 만든 reduce 내부에서도 이터레이터를 직접 만든 후에 순회하는 작업을 거치는데, L.range는 이터레이터를 직접 넣어주기 때문에 이터레이터가 자기 자신을 이터레이터로 다시 반환을 해서 좀 더 효율적인 측면도 있다.


### take

- limit 개수만큼 자르는 함수

```js
const take = (l, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(a);
    if (res.length == l) return res;
  }
  return res;
}

log(take(5, range(100))); // [0, 1, 2, 3, 4]
log(take(5, L.range(100))); // [0, 1, 2, 3, 4]
```

- take를 사용할 때도, range는 모든 array를 다 만들고 자르지만, L.range는 배열을 만들기 전에 필요한 만큼만 take로 잘라서 만들기 때문에 훨씬 효율적이다.  


### 제너레이터/이터레이터 프로토콜로 구현하는 Lazy Evaluation (지연 평가)

#### L.map

```js
L.map = function  *(f, iter) {
  for (const a of iter) yield f(a);
};

var it = L.map(a => a + 10, [1, 2, 3]);
log(it.next()); // next로 실행을 시켜야만 동작하는 Lazy한 map 함수
```

#### L.filter

```js
L.filter = function  *(f, iter) {
  for (const a of iter) if (f(a)) yield a;
};

var it = L.filter(a => a % 2, [1, 2, 3, 4]);
log(it.next()); // next로 실행을 시켜야만 동작하는 Lazy한 filter 함수
```


### 중첩 사용 시 즉시 평가와 지연 평가의 차이 및 순서

```js
go(range(10),
  map(n => n + 10),
  filter(n => n % 2),
  take(2),
  log); // [11, 13]

go(L.range(10),
  L.map(n => n + 10),
  L.filter(n => n % 2),
  take(2),
  log); // [11, 13]
```

- Lazy 예제에서는 take가 가장 먼저 실행된다.
	- 인자로는 L.filter라는 제너레이터가 리턴한 이터레이터가 전달된다.
	- take 내부에서 filter의 이터레이터가 next()로 실행될 때 filter 함수가 Lazy하게 동작한다.
	- yield가 차례로 실행됨에 따라 하나의 값을 차례로 range -> map -> filter -> take를 거쳐 평가하게 된다.
	- 즉시 평가 예제와 비교해서 필요한 만큼만 평가하기 때문에 상당히 효율적이다.


### ES6의 기본 규약을 통해 구현하는 지연 평가의 장점

- 서로 다른 사람들이 만든 라이브러리라도 JS의 기본 규약을 따르기 때문에 안전하게 합성하고, 조합하여 사용할 수 있다.


### 결과를 만드는 함수 reduce, take

#### Array.prototype.join 보다 다형성이 높은 join 함수

- join 함수는 reduce를 이용해서 결과를 만드는 함수

```js
L.entries = function *(obj) { // 지연된 entries
  for (const k in obj) yield [k, obj[k]];
}

const join = curry((sep = ',', iter) =>
  reduce((a, b) => `${a}${sep}${b}`, iter));

const queryStr = pipe(
  L.entries, // Object.entries 대신 L.entries로 지연 평가
  L.map(([k, v]) => `${k}=${v}`), // map 대신 L.map으로 지연 평가
  join('&')
);

log(queryStr({ limit: 10, offset: 10, type: 'notice' })); // limit=10&offset=10&type=notice
```

- 이터러블 프로토콜을 기반으로 만든 join
	- `지연`을 시킬 수 있다.


### take, find

- find 함수는 take를 이용해서 결과를 만드는 함수

```js
const users = [
  { age: 32 },
  { age: 31 },
  { age: 37 },
  { age: 28 },
  { age: 25 },
  { age: 32 },
  { age: 31 },
  { age: 37 }
];

const find = curry((f, iter) => go(
  iter,
  L.filter(f), // 모든 배열을 다 만들지 않고 take가 하나만 찾을 때까지만 순회하는 지연 평가
  take(1),
  ([a]) => a));

log(find(u => u.age < 30)(users)); // {age: 28}

go(users,
  L.map(u => u.age), // find도 이터레이터를 받는 함수이기 때문에 지연된 결과를 받아서 처리해줄 수 있다.
  find(n => n < 30),
  log);
```


### L.map, L.filter로 map, filter 만들기

```js
const map = curry(pipe(
  L.map, // 지연된 결과를
  take(Infinity) // 모두 평가한 결과로 만들어서 리턴하면 map이 된다.
));

log(map(a => a + 10, L.range(4))); // [10, 11, 12, 13]
```

```js
const map = curry(pipe(
  L.filter, // 지연된 결과를
  take(Infinity) // 모두 평가한 결과로 만들어서 리턴하면 filer가 된다.
));

log(filter(a => a % 2, range(4))); // [1, 3]
```


### L.flatten, flatten

- 주어진 값들을 펼치는 이터레이터를 만드는 함수

```js
const isIterable = a => a && a[Symbol.iterator];

L.flatten = function *(iter) {
  for (const a of iter) {
    if (isIterable(a)) for (const b of a) yield b;
    else yield a;
  }
};

let it = L.flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]]);
log([...it]); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
log(take(3, L.flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]]))); // [1, 2, 3]

const flatten pipe(L.flatten, take(Infinity));
log(flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]])); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### yield *

- `yield *iterable` 은 `for (const val of iterable) yield val;` 과 같다.

```js
L.flatten = function *(iter) {
  for (const a of iter) {
    if (isIterable(a)) yield *a;
    else yield a;
  }
};
```

### L.deepFlat

- 깊은 iterable을 모두 펼치고 싶은 경우

```js
L.deepFlat = function *f(iter) {
  for (const a of iter) {
    if (isIterable(a)) yield *f(a);
    else yield a;
  }
};

log([...L.deepFlat([1, [2, [3, 4], [[5]]])]); // [1, 2, 3, 4, 5]
```


### L.flatMap, flatMap

```js
L.flatMap = curry(pipe(L.map, L.flatten)); // 지연 평가인 map 이후에 flatten

let it = L.flatMap(map(a => a * a), [[1, 2], [3, 4], [5, 6, 7]]);
log([...it]); // [1, 4, 9, 16, 25, 36, 49]

const flatMap = curry(pipe(L.flatMap, take(Infinity))); // 지연된 결과를 즉시 평가로 전환
```

---

## 비동기 : 동시성 프로그래밍

### callback과 Promise

- callback

```js
function add10(a, callback) {
  setTimeout(() => callback(a + 10), 100);
}

add10(5, res => {
  add10(res, res => {
    add10(res, res => {
      log(res);
    });
  });
});
```

- Promise : callback에 비해 연속적인 사용이 용이하다.

```js
function add20(a) { // Promise를 return한다.
  return new Promise(resolve => setTimeout(() => resolve(a + 20), 100));
}

add20(5)
  .then(add20)
  .then(add20)
  .then(log);
```


### 비동기를 값으로 만드는 Promise

- Promise가 callback과 가장 큰 차이를 가지는 점은, then으로 결과를 꺼내본다는 단순한 사실이 아니라, 비동기 상황을 `일급 값`으로 다룬다는 것이다.
	- Promise라는 클래스를 사용해서 인스턴스를 반환하고, 이 인스턴스는 `대기`, `성공`, `실패`를 다루는 일급 값으로 이루어져 있다.
	- 대기하고 있는 어떤 값을 만든다는 것.
	- **callback은 반환값이 없어서 log를 찍으면 undefined가 나오지만, Promise는 Promise라는 값을 반환한다.**
	- 값을 반환하기 때문에 계속 이어서 다음 동작을 이어나갈 수 있다.
	- **비동기 상황이 값으로 다루어진다. = 일급이다. = 변수에 할당될 수도 있고, 함수에 전달될 수도 있다.**

```js
const delay100 = a => new Promise(resolve => 
  setTimeout(() => resolve(a), 100));

const go1 = (a, f) => a instanceof Promise ? a.then(f) : f(a); // 아래의 동기, 비동기 상황 모두 받아줄 수 있는 함수
const add5 = a => a + 5;

log(go1(10, add5)); // 동기
log(go1(delay100(10), add5)); // 비동기
```


### 합성 관점의서의 Promise와 모나드

- 모나드 : 박스(컨테이너) 안에 계산할 값이 들어있고, 해당 박스의 함수를 통해 안전하게 함수를 합성할 수 있도록 하는 방법

```js
// f . g
// f(g(x))

const g = a => a + 1;
cosnt f = a => a * a;

log(f(g(1))); // 4
log(f(g())); // NaN

[1].map(g).map(f).forEach(r => log(r)); // 4
[].map(g).map(f).forEach(r => log(r)); // 실행 X. 값이 없어도 안전하다.

Promise.resolve(1).then(g).then(f).then(r => log(r)); // 4
Promise.resolve().then(g).then(f).then(r => log(r)); // 실행 X. 위의 Array를 사용한 모나드와 구조가 동일하다.
```

- Promise도 합성 관점에서 봤을 경우, 비동기 상황에서 적절한 시점에 함수를 평가해서 합성시키기 위한 도구라고 생각해볼 수 있다.


### Kleisli Composition 관점에서의 Promise

- Kleisli Composition : 오류가 있을 수 있는 상황에서 안전하게 함수를 합성할 수 있는 규칙
	- 들어오는 인자가 잘못되거나, 정확한 인자가 들어오더라도 외부의 상황에 의해 정확한 결과를 반환할 수 없을 때, 이를 해결하기 위한 방법

```js
// f . g
// f(g(x)) = f(g(x)) : 수학적으로는 맞지만, 프로그래밍에서는 평가 시점에 따라 좌변과 우변의 함수의 상태가 달라질 수 있다.
// f(g(x)) = g(x) : g가 에러가 난 상황이라면 좌변과 우변의 결과가 같아야 한다. -> Kleisli Composition

let users = [
  { id: 1, name: 'aa' },
  { id: 2, name: 'bb' },
  { id: 3, name: 'cc' }
];

const getUserById = id => 
  find(u => u.id === id, users);

const f = ({name}) => name;
const g = getUserById;

const fg = id => f(g(id));

const r = fg(2);
users.pop();
users.pop();
const r2 = fg(2); // error
```

```js
const getUserById = id => 
  find(u => u.id === id, users) || Promise.reject('없어요!');

const fg = id => Promise.resolve(id).then(g).then(f).catch(a => a);

g(2); // Promise rejected
users.pop();
users.pop();
fg(2).then(log); // 없어요!
```


### Promise.then의 중요한 규칙

- Promise.then은 중첩된 Promise가 있더라도 한번에 값을 꺼낼 수 있게 해준다.

```js
Promise.resolve(Promise.resolve(Promise.resolve(1))).then(log); // 1

new Promise(resolve => resolve(new Promise(resolve => resolve(1))).then(log); // 1
```

