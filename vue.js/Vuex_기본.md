# Vuex 기본

## 참고

[Vuex Tutorial - YouTube](https://www.youtube.com/playlist?list=PL4cUxeGkcC9i371QO_Rtkl26MwtiJ30P2)를 보고 정리한 글입니다.

## Vuex 란?

한마디로 Vue.js에서 모든 상태를 한 곳에서 관리해 주는 친구입니다.  

중앙에 Store라는 상태 저장소가 있고, 이 Store는 모든 애플리케이션의 컴포넌트에서 접근이 가능합니다.  


### Vuex를 사용한다면

[사진]  

Vue.js만 사용한다면, 보통 여러 컴포넌트 간의 관계는 위와 같은 트리 구조일 것입니다.  

만약 4번 컴포넌트에서 이벤트가 발생해서 다른 컴포넌트들의 상태에 변화를 주어야 한다면, 이벤트는 부모를 타고 올라가 Root 컴포넌트까지 전달된 다음, 다시 props를 통해 다른 하위 컴포넌트들로 전달되어야 하겠죠.  

[사진]  

하지만 Vuex를 사용한다면 중앙 저장소인 Store가 있고, 특정 컴포넌트에서 상태를 변경한다면 이 상태를 사용하고 있는 모든 컴포넌트에서는 해당 상태를 중앙 Store에서 가져다가 쓰기만 하면 됩니다.  

이벤트나 상태 전달 시 컴포넌트들 간의 관계를 전부 고려해주어야 하는 리소스가 훨씬 줄어들게 되죠.  

## 중앙 저장소 세팅하기

### Vuex 설치

```
npm install vuex --save
```

npm으로 설치합니다.  

`--save` 옵션은 package.json 파일의 dependencies 항목에 플러그인 정보를 저장하겠다는 의미입니다.  
프로덕션 빌드 시 해당 플러그인을 포함하게 됩니다.  

### Store 생성하기

```js
// store.js

import Vue from 'Vue';
import Vuex from 'vuex';

Vue.use(Vuex);

const store = new Vuex.Store({
  state: {
    fruits: [
      {name: 'apple', price: 3},
      {name: 'banana', price: 4},
      {name: 'strawberry', price: 5}
    ]
  }
});
```

store.js 를 만들고 Store를 생성해 봅시다.  

Vue와 Vuex를 import하고, 전역 Vue에 Vuex를 등록합니다.  
그리고 store라는 이름으로 위와 같이 만든 후, state 필드에 상태를 관리하고 싶은 오브젝트를 세팅합니다.  

store가 상태를 관리하는 오브젝트는 필요한 모든 컴포넌트에서 사용할 수 있습니다.  


### Store 등록하기

만든 store를 사용하려면 먼저 main.js에 등록을 해야 합니다.  

기존에 있던 프로젝트의 main.js에 store.js를 import하고, 아래와 같이 등록합니다.  

```js
import Vue from 'Vue';
import App from './App.vue';
import { store } from './store/store';

new Vue({
  store: store, // store 등록
  el: '#app',
  render: h => h(App)
})
```


## Store 사용하기

### computed 속성으로 등록한 상태 가져오기

상태 관리로 등록한 오브젝트를 컴포넌트에서 사용해 봅시다.  

정적인 계산된 값을 불러오는 computed 속성을 사용한다면, 다음과 같이 사용할 수 있습니다.  

```html
// ExampleComponent.vue

<template>
  <div id="example-component">
    <li v-for="fruit in fruits">
      <span> {{ fruit.name }}</span>
      <span> ${{ fruit.price }}</span>
    </li>
  </div>
</template>

<script>
export default {
  computed: {
    products() {
      return this.$store.state.fruits;
    }
  }
}
</script>
```

`this.$store.state` 를 통해 Store에서 관리되는 상태 오브젝트의 값을 가지고 올 수 있습니다.  


### 등록된 상태의 값을 변경하여 가져오기

만약 등록된 오브젝트의 상태를 변경해서 뿌려주어야 하는데, 이 변경하는 방식 또한 모든 컴포넌트에서 공통으로 사용되는 함수라면 Store에서 관리하는 게 좋을 것입니다.  

예를 들어 예제에서 등록했던 fruits의 가격을 특정 컴포넌트들에서는 반값으로 할인하고 표시해주고 싶다면, 해당 가공 로직은 컴포넌트마다 동일할테니 중앙에서 공통으로 관리하면 좋겠죠?  

이 때는 Store의  `getters` 필드에 사용자 정의 함수를 만들고 사용할 수 있습니다.  

```js
// store.js

const store = new Vuex.Store({
  state: {
    fruits: [
      {name: 'apple', price: 3},
      {name: 'banana', price: 4},
      {name: 'strawberry', price: 5}
    ]
  },
  getters: {
    saleFruits: state => {
      return state.fruits.map(fruit => {
        return {
          name: '**' + fruit.name + '**',
          price: fruit.price / 2
        }
      });
    }
  }
});
```

```js
// ExampleComponent.vue

<script>
export default {
  computed: {
    products() {
      return this.$store.state.fruits;
    },
    saleProducts() {
      return this.$store.getters.saleFruits;
    }
  }
}
</script>
```

