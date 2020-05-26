# For...of와 For...in의 차이
자바스크립트로 코드를 작성하다 보면 어떤 리스트나 객체에 대해 순회를 해야할 경우가 생기는데요.  



## 이터러블 프로토콜 (Iterable Protocol)

(작성중)

- 이터러블 : 이터레이터를 리턴하는 `[Symbol.iterator]()` 를 가진 값
- 이터레이터 : { value, done } 객체를 리턴하는 next() 를 가진 값
- 이터러블/이터레이터 프로토콜 : 이터러블을 for...of, 전개 연산자 등과 함께 동작하도록 한 규약

[Iteration protocols - JavaScript | MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Iteration_protocols#iterable)


## For...of와 For...in의 차이

### For...of

For...of는 **이터러블 프로토콜을 따르는 값을 순회할 수 있는** 구문입니다.  

이터러블 프로토콜을 따른다는 것은 순회 가능한 이터레이터를 리턴하는 `[Symbol.iterator]()` 를 가진다는 것이고, 이를 가진 값에는 Array, Map, Set, String 등이 있습니다.  

즉, 이터러블 프로토콜을 따르고 그에 따라 `[Symbol.iterator]()`가 정의되어 있는 모든 배열, Map, Set, String 등은 For...of문으로 순회 가능합니다.  

그리고 이터러블 프로토콜을 따르고 있다는 것은 마찬가지로 [전개 연산자](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax)도 사용할 수 있다는 뜻도 됩니다.  
전개 연산자가 **이터러블한 값을 순회하면서** 풀어주는 역할을 하는 구문이기 때문입니다.  

참고로 사용자가 제너레이터로 직접 정의한 이터레이터도 이터러블하기 때문에 For...of, 전개 연산자로 순회할 수 있습니다.  

### For...in

For...in은 `[[Enumerable]]` 속성이 true인, **열거 가능한 객체**에 대해 사용 가능합니다.  

(작성중)

