# Vue 3.0에서 주목해야 할 피쳐들

Evan You가 medium 에 Vue가 나아갈 새로운 버전에 대한 아티클을 제시한지 9개월정도가 지난 지금 다시 한번 읽어보며 어떤 변화가 있을지 정리하려 한다. 
당초 목표는 2019년 4분기에는 IE11을 마지막으로 모든 피쳐가 릴리즈될 계획이였던 것 같은데, 6월 현재 어떻게 될지는 조금 더 지켜봐야 할 것 같다.

### High-Level API Changes
render function api 와 scoped-slots 문법을 제외한 모든것은 그대로 유지되거나 호환성 빌드를 통해 2.x 과 호환이 가능하다.

#### template syntax
scoped-slot 문법에 약간의 변화가 생긴다. 그외에 템플릿 문법은 거의 동일하게 유지될 예정이다. (2.6에 이미 적용됨)

#### Class API 
클래스 API가 제안되었으나, 여러 엣지 케이스들을 커버하지 못하는등 풀기 어려운 문제들을 마주해 현재는 포기된 상태이다. RFC는 다음을 참고한다. [[Abandoned] Class API proposal](https://github.com/vuejs/rfcs/pull/17)

#### Functional API
클래스 기반 컴포넌트 대신 함수 기반 API들이 제안되었다. 다음을 참고할 것 [Function-based API proposal](https://github.com/vuejs/rfcs/pull/42)

#### Mixin
믹스인 또한 그대로 제공될것
> 믹스인이 왜? 논의가 오고 갔었나? - 못찾겠음

#### Global mounting/configuration API change
API는 플러그인을 설치할때 전역적인 뷰 런타임에 대한 변이를 피하기 위해 점검을 받도록 변경될 가능성이 있다.
대신 플러그인을 적용하고 컴포넌트 트리에 적용될 범위를 지정한다.
이로인해 특정한 플러그인에 의존성을 갖는 컴포넌트를 테스트하기 쉬워지며 같은 페이지에 여러 뷰 애플리케이션을 서로 다른 플로그인과 함께 마운트 하면서도 동일한 뷰 런타임을 사용하는것이 가능해진다.
이로인해 특정한 플러그인에 의존성을 갖는 컴포넌트를 테스트하기 쉬워지며 같은 페이지에 여러 뷰 애플리케이션을 서로 다른 플로그인과 함께 마운트 하면서도 동일한 뷰 런타임을 사용하는것이 가능해진다
> RFC https://github.com/vuejs/rfcs/pull/29

```javascript
import Vue from 'vue'
import App from './App.vue'

Vue.config.ignoredElements = [/^app-/]
Vue.use(/* ... */)
Vue.mixin(/* ... */)
Vue.component(/* ... */)
Vue.directive(/* ... */)

new Vue({
  render: h => h(App)
}).$mount('#app')
```

위와 같이 작성했던 코드들이 다음과 같이 변경된다.

```javascript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

app.config.ignoredElements = [/^app-/]
app.use(/* ... */)
app.mixin(/* ... */)
app.component(/* ... */)
app.directive(/* ... */)

app.mount('#app')
```

전역 Vue에서 일어났던 일들이 `createApp` 메소드를 통해 생성된 애플리케이션 인스턴스로 관심이 옮겨가며 오로지 앱 인스턴스
스코프에만 영향을 끼친다. 또한 `nextTick`과 같은 전역 API들은 전역 Vue의 상태를 변화시키지 않을 것이다.

이런 변화가 생긴 이유는 몇가지 문제를 해결하기 위함이다.
- 전역 설정은 테스트하는동안 다른 테스트 케이스를 의도치 않게 오염시키기 쉽다. 사용자는 항상 기존의 전역 설정을 잘 저장해두었다가, 테스트가 종료되면 되돌려야 하는 버거로움을 겪었다.
  + `vue-test-utils`는 이 문제를 `createLocalVue`라는 API를 통해 해결했다.
- 한 페이지에 두개 이상의 뷰 인스턴스가 생성될때 어려움을 겪게 했다.

#### Functional Component
- 함수형 컴포넌트는 마침내 단어 그대로의 함수가 된다. - 하지만 이제 비동기 컴포넌트는 헬퍼 함수를 통해 명시적으로 생성해야 한다.
> 연관된 RFC https://github.com/vuejs/rfcs/pull/27

출처 : [Plans for the Next Iteration of Vue.js](https://medium.com/the-vue-point/plans-for-the-next-iteration-of-vue-js-777ffea6fabf)
