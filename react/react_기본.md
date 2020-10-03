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


### JSX 기본 문법

```js
import React, { Component, Fragment } from 'react';

class App extends Component {
  render() {
    const name = "wbluke";
    return (
      <Fragment>
        <input type="text" />
        <div>
          hello {name}
        </div>
        <div>
          {
            1 + 1 === 2 
              ? (<div>맞아요!</div>)
              : (<div>틀려요!</div>)
          }
        </div>
        <div>
          {
            1 + 1 === 2 && (<div>맞아요!</div>)
          }
        </div>
        <div>
          {
            (() => {
              if (value === 1) return (<div>하나</div>);
              if (value === 2) return (<div>둘</div>);
            })()
          }
        </div>
      </Fragment>
    );
  }
}

export default App;
```

- 태그는 꼭 닫혀있어야 한다.
- 두 개 이상의 엘리먼트는 무조건 하나의 엘리먼트로 감싸져 있어야 한다.
	- div 대신 렌더링 후 아무것도 남지 않는 Fragment 를 사용할 수 있다.
- 중괄호로 JS 값을 사용할 수 있다.
	- 참고 : var는 scope가 블록 단위가 아니라 함수 단위이다. 이제는 쓰지 않고, 블록 단위로 지원하는 let과 const를 사용한다.
- JSX 내부에서 조건부 렌더링을 하려면 삼항 연산자나 AND 연산자를 사용한다.
	- if문을 사용하려면 IIFE(즉시 실행 함수)를 사용해야 한다.

```js
import React, { Component } from 'react';

class App extends Component {
  render() {
    const style = {
      backgroundColor: 'black',
      padding: '16px',
      color: 'white',
      fontSize: '36px'
    };

    return (
      <div style={style}>
        {/* 주석은 이렇게 */}
        안녕하세요!
      </div>
    );
  }
}

export default App;
```

- style은 객체 형태로 넣어주고, camelCase로 속성명을 지정한다.
	- 숫자값도 단위까지 지정하여 문자열로 넣어준다.
	- 사용할 때는 `style={style}` 로 사용한다.
- 만약 별도의 css 파일을 만들어서, class name을 지정하고 싶다면 class가 아니라 `className="App"` 으로 지정한다.
- 주석은 멀티라인 문법으로 작성해야 하고, 중괄호 안에 넣어주어야 한다.
	- 태그 안에 주석을 사용할 때는 `//`로 작성할 수 있다.


### Props

```html
<Child value="value" />
```

부모가 자식에게 전달하는 속성을 `props`라고 한다.  

```js
// MyName.js
import React, { Component } from 'react';

class MyName extends Component {

  static defaultProps = { // default 값 설정
    name: '기본이름'
  }

  render() {
    return (
      <div>
         안녕하세요! 제 이름은 <b>{this.props.name}</b> 입니다.
      </div>
    );
  }
}

// MyName.defaultProps = {
//   name: '기본이름'
// };

export default MyName;
```

```js
// App.js
import React, { Component } from 'react';
import MyName from './MyName';

class App extends Component {
  render() {
    return <MyName name="리액트" />;
  }
}

export default App;
```

함수형 컴포넌트로 작성한다면 다음과 같이 작성할 수 있다.  

```js
// MyName.js
import React from 'react';

class MyName = ({ name }) => { // 구조 분해 할당
    return <div>안녕하세요! 제 이름은 <b>{this.props.name}</b> 입니다.</div>;
};

MyName.defaultProps = {
  name: '기본이름'
};

export default MyName;
```

함수형 컴포넌트에서는 더이상 Component를 import하지 않아도 된다.  

그리고 함수형 컴포넌트에서는 State와 라이프사이클 기능이 빠져있다.  
대신 함수형 컴포넌트가 초기 mount 속도가 미세하게 조금 더 빠르다.  
불필요한 기능이 없기 때문에 메모리 자원도 덜 쓴다.  
따라서 간단히 어떤 값을 받아와서 보여주기만 하는 심플한 컴포넌트라면 활용하면 좋다.  


### State

State는 내부에서 변경할 수 있다.  
변경할 때는 언제나 setState()라는 함수를 사용한다.  

```js
// Counter.js
import React, { Component } from 'react';

class Counter extends Component {

  state = {
    number: 0
  }

// custom 함수는 람다 함수로 작성해야 한다. 안그러면 함수 내부에서 this를 찾을 수 없다.
// 아니면 아래와 같은 생성자 작업으로 함수 내부에서 this를 사용할 수 있도록 해줘야 한다.
/*
  constructor(props) {
    super(props);
    this.handleIncrease = this.handleIncrease.bind(this);
    this.handleDecrease = this.handleDecrease.bind(this);
  }
*/

  handleIncrease = () => {
    this.setState({
      number: this.state.number + 1
    })
  }

  handleDecrease = () => {
    this.setState({
      number: this.state.number - 1
    })    
  }

  render() {
    return {
      <div>
        <h1>카운터</h1>
        <div>값: {this.state.number}</div>
        <button onClick={this.handleIncrease}>+</button>
        <button onClick={this.handleDecrease}>-</button>
      </div>
    }
  }
}

export default Counter;
```

```js
// App.js
import React, { Component } from 'react';
import MyName from './MyName';

class App extends Component {
  render() {
    return <Counter />;
  }
}

export default App;
```













































