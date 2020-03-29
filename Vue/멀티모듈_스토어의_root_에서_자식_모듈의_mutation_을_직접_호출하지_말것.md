# 멀티모듈 스토어의 root에서 자식 모듈의 mutation을 직접 호출하지 말 것

## 들어가며
얼마 전 업무를 하다가 반응형이 제대로 걸리지 않는 버그가 발생했고, 원인을 찾다보니 멀티모듈 스토어로 구성된 vuex 스토어의 root 에서 자식 모듈의 `mutation` 을 직접 호출할시 반응형이 걸리지 않는 다는 사실을 알게 되었다.

왜 그런 문제가 발생하는지, 레퍼런스는 어떻게 가이드 하고 있는지 정리하려 한다.

> 전체 코드는 다음 [레파지토리](https://github.com/genie-youn/do-not-call-child-module-mutation) 에서 확인 가능하다.

## 멀티모듈 스토어
우선 `vuex` 의 멀티 모듈 스토어에 대해 설명하려 한다.

`vuex`는 스토어를 모듈로 관리할 수 있는 기능을 제공한다. 다음과 같이 각각의 도메인에 대응되는 스토어를 모듈로 작게 쪼개서 구성할 수 있고, 네임스페이스를 부여하여 사용할 수 있다.

```javascript
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> `moduleA`'s state
store.state.b // -> `moduleB`'s state
```

## 문제

다음과 같이 단순히 스토어의 `state` 를 렌더링하는 컴포넌트가 있다고 하자.

```vue
<template>
  <div>
    <h1>{{ message }}</h1>
  </div>
</template>

<script>
import {mapGetters} from "vuex";

export default {
  name: 'HelloWorld',
  computed: {
    ...mapGetters("moduleA", ["message"])
  },
};
</script>
```

컴포넌트가 의존하고 있는 스토어는 다음과 같이 생겼다.

```javascript
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
  },
  mutations: {
  },
  actions: {
  },
  modules: {
    moduleA,
  },
});


const moduleA = {
  state: {
    message: "",
  },
  mutations: {
    updateMesasge(state, payload) {
      state.message = payload;
    },
  },
  actions: {},
  getters: {
    message: state => state.message,
  },
}
```
