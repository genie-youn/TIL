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
> RFC [Global mounting/configuration API change](https://github.com/vuejs/rfcs/pull/29)

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

기존에 다음과 같이 사용하던 Functional Component를 

```javascript
const FunctionalComp = {
  functional: true,
  render(h) {
    return h('div', `Hello! ${props.name}`)
  }
}
```

다음과 같이 변경한다.

```javascript
import { h } from 'vue'

const FunctionalComp = props => {
  return h('div', `Hello! ${props.name}`)
}
```

AsyncFunctionalComponent는 어쩔수 없이..

```javascript
import { createAsyncComponent } from 'vue'

const AsyncComp = createAsyncComponent(() => import('./Foo.vue'))
```
 
> 연관된 RFC [Functional and async components API change](https://github.com/vuejs/rfcs/pull/27)

#### Virtual DOM format
가장 큰 변화를 겪는 부분은 렌더 함수에서 사용되는 가상 돔의 형식이다. 여러 주요 라이브러리들의 저자들에게 피드백을 구하는 중이며, 하위호환성을 지킬 수 있도록 신경쓸께

### Source Code Architecture
내부적으로 모듈관의 결합도를 낮추고, 코드베이스를 기여하기 더 쉽게 하도록 하기 위해 타입스크립트로 다시 작성한다.
타입스크립트의 타입 정보를 통해 IDE의 더 강력한 지원에 힘입어 새로운 컨트리뷰터가 의미있는 기여를 할 수 있도록 도울것이라 확신한다.

`observer` 모듈과 `scheduler` 모듈을 패키지를 분리해 다른 구현체로 대체하기 쉽게 만들 예정이다 . 
예를들어, IE11만을 위한 `observer` 구현체를 작성한다거나..

### Observation Mechanism
더 완전하고, 정확하고 효율적이면서 디버깅하기 용이한 반응형 추적과 `observable`을 만들기 위한 API들을 제공한다.

3.0에서는 Proxy에 기반을 둔 반응형 추적을 제공하는 옵저버 구현체로 변경된다.
`Object.defineProperty`로 구현되는 현재의 반응형 시스템에는 몇가지 문제가 존재한다.

- 프로퍼티의 추가와 삭제의 탐지가 어렵다.
- 배열의 인덱스를 통한 상태 변화와 `.length`의 변화를 탐지하기가 어렵다.
- Map, Set, WeakMap, WeakSet을 지원하기가 어렵다.

또한 새로운 옵저버는 다음과 같은 기능을 제공할 것이다.

- 옵저버 생성에 관련된 API를 노출한다. 이것은 가볍고, 중간 규모의 프로젝트에서 서로 다른 컴포넌트간의 상태 관리에 매우 간결한 대한 해결책이 될 것이다.
- 게으른 관측<sup>Lazy observation</sup>이 기본설정으로 변경된다. 2.x 에서는 반응형 데이터는 그 크기가 얼마나 크던지 상관없이 애플리케이션이 시작됨과 동시에 관측되기 시작했다. 이것은 데이터 셋이 클 경우 앱이 시작되는 동안 인지할 만한 수준의 오버헤드를 유발했다. 3.x 에서는 오로지 첫 페이지에서 렌더링되어 보여지는 부분에 필요한 데이터만 관측되고, 당연히 관측자체도 굉장히 빨라질 것이다.
- 반응형 관측을 위해 `Vue.set`을 통해 강제로 새로운 프로퍼티를 추가할 필요 없이, 자동으로 새롭게 추가된 프로퍼티 또한 추적될것이다.
- 불변 옵저버 타입을 제공한다 
> 자식 컴포넌트로 드릴링 되는 프로퍼티나 Vuex이 상태 트리에 유용할것 같다.
- 디버깅이 좀 더 쉬워진다. 언제, 왜 컴포넌트가 다시 랜더링 되었는지를 추적하는데 사용할 수 있는 `renderTracked` 훅과 `renderTriggered` 훅이 추가된다.
> 오 이거 좋네요..

### Other Runtime Improvements
작아지고, 빨라지고, `tree-shakable`을 지원하고 (:wow:) fragments & portals?, 사용자정의 렌더 API를 제공한다.

- 새로운 코드베이스는 `tree-shakable`에 친화적으로 작성될 것이다. 빌트인 컴포넌트(`<transition>`, `<keep-alive>`) 나 디렉티브 런타임 헬퍼 (`v-model`) 등과 같은 기능들은 필요시에만 추가된다. 결과적으로 반드시 포함해야 하는 번들의 사이즈는 gzip으로 압축했을 때 10kb 미만으로 작아질 것이다.
- [Inferno](https://infernojs.org/) 로부터 영감을 받아 버츄얼 돔에 관련된 성능을 크게 개선하였다. 성능 측정 결과 약 100% 정도 빨라진다. 옵저버의 개선으로 앱을 실행시켰을때, 컴포넌트 인스턴스를 초기화하고 데이터에 반응형을 설정하는데 필요한 시간을 절반으로 줄였다.
- Fragments & Portals: 작아진 크기에도 불구하고 3.0에서는 Fragments(컴포넌트가 여러 루트 노드를 반환함)와 Portals(컴포넌트 내부 대신 DOM의 다른 영역의 서브트리를 렌더링함)을 빌트인으로 지원한다.
- 슬롯 메커니즘이 개선된다. 모든 컴파일러에 의해 생성된 슬롯은 이제 함수로 취급되고 자식 컴포넌트가 렌더 함수가 호출되는 동안 실행된다. 이렇게 하면 슬롯의 종속성이 부모컴포넌트 대신 자식 컴포넌트로 잡히게 된다. 이것은 두가지 의미를 갖는다. 1. 슬롯의 내용이 변경되었을 때 오로지 자식 컴포넌트만 새로 렌더링 되고. 2. 부모 컴포넌트가 다시 렌더링 됐을 때, 슬롯에 변경사항이 없다면 자식 컴포넌트는 다시 렌더링되지 않는다. 이 변경 사항은 컴포넌트 트리 수준에서 훨씬 더 정확한 변경 탐지를 제공하므로 쓸모없이 다시 렌더링되는 횟수를 줄일것이다.
- 사용자 렌더러를 생성할 수 있는 1급 API가 제공된다. 이로 인해 더이상 vue의 코드를 포크해서 사용자가 재정의할 필요가 없어진다. 이 변경사항은 네이티브를 위한 렌더러가 필요한 [Weex](https://weex.apache.org/)나 [NativeScript Vue](https://nativescript-vue.org/)와 같은 프로젝트가 Vue의 변경사항을 따라가는데 더 적은 노력을 들게 만들것이다. 또한 다양한 다른 목적으로 사용자 정의 렌더러를 만드는것을 쉽게 만들것이다.

### Compiler Improvements
tree-shaking을 보다 더 잘 지원하고, 좀 더 AOT 최적화되고, 파서는 에러정보와 소스맵을 더 잘 지원하게 된다.
> AOT?

- tree-shaking이 가능한 번들러를 사용할 때 선택적 기능을 사용하는 템플릿은 ES 모듈 구문을 통해 해당 기능을 가져오는 코드를 생성하게 된다. 이로인해 사용하지 않는 선택적 기능은 최종 번들파일에서 제거되게 된다.
- 새로운 버츄얼 돔 구현으로 인해 정적 트리 호이스팅, 정적 프로퍼티 호이스팅, 런타임을 위한 컴파일러 힌트, 자식 노드 정규화 스킵, 빠른 가상 노드 패스 생성등 좀 더 효과적인 컴파일 타임 최적화가 이루어진다.
- 템플릿을 컴파일하다 에러가 발생했을 때 위치정보를 제공하기 위해서 파서를 다시 구현할 예정이다. 새로 작성되는 파서는 템플릿 소스맵을 지원해야 하며 IDE나 `eslint-plugin-vue` 서드파티 도구들에게 기반을 제공해야한다.

### IE11 Support
IE11은 여전히 지원될것이다. 하지만 2.x 에서의 반응형 한계들이 여전히 유지될 것이다.

새로운 코드베이스는 ES2015를 지원하는 ever green 브라우저들에 초점이 맞추어져 있다. 그러나 우리는 뷰 사용자들이 아직 IE11에 대한 지원이 필요하다는 사실 또한 잘 알고있다. 대부분의 ES2015 기능들은 IE11을 위해 트랜스파일되거나 폴리필 될 수 있지만, `Proxy`는 불가능하다. 우리의 계획은 같은 API를 제공하는 대체 옵저버를 `proxy` 대신 `Object.defineProperty`를 사용하여 구현하여 제공할 예정이다. 이 옵저버 구현을 사용한 Vue 3.x의 별도의 빌드가 배포된다. 그러나 이 빌드는 Vue 2.x와 동일한 변경 감지 한계를 갖게 되므로 3.x의 모던한 빌드와 완전히 호환되지는 않는다. 이것이 라이브러리 제공자들이 두가지 다른 빌드에 대한 호환성을 알아야 하기 때문에 불편함을 갖는다는 사실을 인지하고 있지만, 최대한 상세히 가이드함으로써 해결할 예정다.

-- 이하 어떻게 이것을 진행해 나갈것인지에 대한 이야기
인상적인 부분은 RFC를 운영하겠다. 이네요.

출처 : [Plans for the Next Iteration of Vue.js](https://medium.com/the-vue-point/plans-for-the-next-iteration-of-vue-js-777ffea6fabf)
