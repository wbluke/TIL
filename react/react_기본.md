# React 기본
#TIL/react

---

## 프론트엔드 라이브러리

- Angular
- React
	- Component 라는 개념에 집중이 되어있는 라이브러리 (프레임워크가 아니다)
	- 공식 서드파티 라이브러리 같은게 없어서 하나의 문제를 해결하는데 여러 방법이 있다.
	- 생태계가 넓다
- Vue
	- 초심자가 사용하기 좋다.
	- HTML 템플릿이 있어 퍼블리셔 등이 접근하기 좋다.

---

## 리액트의 Virtual DOM

### 리액트 이전 메뉴얼

We build React to solve one ploblem : building large applications with data that changes over time.
(우리는 지속해서 데이터가 변화하는 대규모 애플리케이션을 구축하기 위해 리액트를 만들었습니다.)

### Model

보통의 프레임워크에서 Model이 작동하는 방식은 `양방향 바인딩`이다.  
Model에 있는 값이 변하면 View에서도 값을 변화시키고, View에서도 값을 변화시키면 Model에 있는 값을 변화시키는 것이다.  

변화(Mutation)는 상당히 복잡한 작업이다.  
특정 Event 발생 -> Mutation이 일어남 -> 어떤 DOM을 가져와서 어떤 방식으로 View를 업데이트할지 로직을 정해야 함

페이스북에서는 리액트를 만들기 전에 다음과 같은 발상을 했다.  
"그냥 Mutation을 하지 말자. 그 대신에, 데이터가 바뀌면 그냥 뷰를 날려버리고 새로 만들어버리면 어떨까?"  

그래서 존재하는 것이 **Virtual DOM**이다.  
변화가 일어나면 실제로 브라우저의 DOM에 새로운 것을 넣는 것이 아니라 자바스크립트로 이루어진 가상의 DOM에 한번 렌더링을 한 후, 기존의 DOM과 비교해서 정말 변화가 필요한 곳에만 업데이트를 하는 방식이다.  

[React and the Virtual DOM - YouTube](https://www.youtube.com/watch?v=muc2ZF0QIO4)

### 리액트를 특별하게 만드는 점

Virtual DOM을 쓰는 라이브러리는 꽤 많다. (Vue 포함)

리액트를 특별하게 만드는 점은 다음과 같다.  

- 어마어마한 생태계
- 페이스북에서 만들었고 많은 대기업들에서 사용한다.
- 한 번 사용하면 좋아하게 된다..... 오호

---

## 본격적인 리액트 코드 작성하기

### Webpack과 Babel

- Webpack
	- 서로 의존하는 여러 확장자의 파일들을 bundling 하여 미리 정한 규칙을 통해 단일 static 파일들로 만드는 도구.  
- Babel
	- 자바스크립트 변환 도구
	- ES6 같은 최신 모던 JS 문법을 구 브라우저에서 구동시킬 수 있도록

### Code Sandbox

```js
import React, { Component } from 'react';

class App extends Component {
  render() {
    return (
      <div>
        <h1>안녕하세요 리액트</h1>
      </div>
    );
  }
}

export default App;
```

Component를 만드는 방법은 class를 통해서 만드는 방법과 함수를 통해 만드는 방법, 두 가지가 있다.  

































