# Flux와 Vuex

## 들어가며
Vuex는 규모있는 Vue 애플리케이션을 개발하는데 빠트릴 수 없는 주요한 라이브러리이다. 애플리케이션의 수 많은 컴포넌트들에 광범위하게 영향을 끼치는 상태를 관리하는데 도움을 주는 이 라이브러리는 마치 신이 주신 선물같은 감동을 선사한다.

하지만 역시 잘 모르고 써서 그런걸까? 실무에서 사용하면서 많은 시행착오를 겪었고 때로는 오히려 너무 복잡해져 버린 스토어를 보면서 깊은 탄식을 내뱉었던 길 잃은 순한 양이 된듯한 기분도 적잖이 느꼈다.

이제서야 Vuex가 무엇인지 조금은 알 것 같은 요즘 (사실 지금 느끼고 있는 이것들도 정말 best practice에 근접한 방법인지 잘 모르겠지만) Vuex와 기반이 되는 개념인 Flux에 관하여 정리를 하고자 한다.

## Vuex와 Flux, 그리고 Elm
Vuex의 공식 레퍼런스를 보면 기본적인 개념은 [Flux](https://facebook.github.io/flux/docs/overview/), [Redux](http://redux.js.org/), 그리고 [Elm](https://guide.elm-lang.org/architecture/) 에서 영감을 얻었다고 설명하고 있다.

> https://vuex.vuejs.org/

그렇다면 Flux와 Elm 은 무엇일까? 하나씩 살펴보도록 하자.

### Flux
Flux는 Facebook에서 만든 클라이언트 사이드 웹 어플리케이션을 위한 아키텍처라고 소개하고 있다. 단방향 데이터 흐름을 활용하여 React의 조합 가능한 뷰 컴포넌트를 보완한다고 한다.

Flux는 `Dispatcher`, `Store`, `View` 크게 세 부분으로 구성되어 있는데, 데이터가 단방향으로 흐른다는게 핵심이다. `View`가 사용자와의 상호작용을 통해 상태의 변경이 생긴다면 중앙의 `Dispatcher`를 통해 `action`을 전파한다. `Store`는 이 `action`을 받으면 자신의 상태를 변경하고 영향이 있는 모든 `View`를 변경하게 된다. 이는 React의 선언형 프로그램이 방식으로 구현되게 된다.

Flux는 규모가 큰 MVC 어플리케이션에서 발생할 수 있는 데이터간의 복잡한 의존성과 연쇄적으로 일어나는 갱신이 뒤얽힌 데이터 흐름을 만들고 예측하기 힘든 상황을 만들어내는 문제를 해결하기 위해 고안되었다.

Flux는 `Store`를 통해 제어를 역전시킨 점이 특징이다.

#### A Single Dispatcher
`Dispatcher`는 Flux 애플리케이션의 모든 데이터 흐름을 관리하는 중앙 허브이다. 등록된 각각의 `Store`에게 `Action`을 분배하는 단순한 역할을 수행한다. 각각의 `Store`는 스스로 등록하고 콜백을 제공한다. `Action Creator`가 새로운 `Action`이 있다고 `Dispatcher`에게 알려주면 각각의 `Store`는 앞서 등록해두었던 콜백을 통해 새로운 `Action`을 전달받게 된다.

#### Stores
상태와 비즈니스 로직이 담겨있다. MVC 패턴의 model과 유사하지만 많은 객체를 관리한다는 점에서 차이가 있다.
ORM의 단일 상태 레코드와는 좀 다르고, Backbone의 컬렉션과도 조금 다르다. `Store`는 애플리케이션의 도메인을 나타낸다.
`Dispatcher`에 스스로를 등록하고 콜백을 제공하는데, 이 콜백은 `Action`을 인자로 받게된다. `Action`의 타입에 따라 적절히 상태를 변경하는 로직을 수행하고 이 변경은 이벤트로 전파된다. 각 `View`들은 이 이벤트를 받아 변경된 상태로 자신을 스스로 변경한다.

#### Views and Controller-Views


> https://haruair.github.io/flux/docs/overview.html
