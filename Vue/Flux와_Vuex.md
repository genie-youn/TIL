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
React는 조합가능한 View를 제공한다. 이렇게 조합되어 중첩된 View 계층의 최상위에는 Controller-View라는 특별한 뷰가 존재하게 된다. 이 Controller-View는 의존하고 있는 상태에 대한 스토어의 변경 이벤트가 전파되면 이를 받아 하위에 있는 View에게 상태를 전달하게 된다. 이로써 하위에 있는 View는 순수한 함수의 성격을 갖도록 유지할 수 있게 된다.

`Store`의 전체 상태를 단일객체로 하위에 있는 View로 전달하게 되는데 이는 관리해야할 프로퍼티를 줄이는 효과를 갖는다.

때때로 이 Controller-View를 계층 중간에 끼워넣어야 할 상황도 생긴다. 이렇게 할 경우 특정 도메인 계층을 캡슐화 할 수 있다는 장점을 갖지만 데이터 흐름이 복잡해질 가능성이 있으며 이는 디버깅을 어렵게 만드는 요인으로 작용하기도 한다.

#### Actions

`Dispatcher`는 `Action` 이라는 데이터 페이로드를 받아 `Store`를 디스패치 할 수 있는 메소드들을 외부로 노출한다. `Action`은 이 메소드들을 통해 생성되어 `Dispatcher`로 전달되게 된다.

예를 들어, TO-DO 어플리케이션에서 특정 해야할 일의 텍스트를 변경한다고 해보자. 이 경우 `TodoActions` 모듈의 `updateText(todoId, newText)` 메소드를 호출하여 `Action` 을 생성하게 될테고 `Store`는 이 `Action`에 따라 적절하게 `Store`의 상태를 변경하게 된다.

이런 `Action`을 생성하는 메소드는 `Action`에 특정한 타입을 부여할 수 있는데, `Store` 는 이 타입을 해석하여 적절히 처리하게 된다.
위 예제에선 `TODO_UPDATE_TEXT` 와 같이 타입을 부여할 수 있을 것이다.

아마 이런 `Action`을 생성하는 대부분의 메서드는 유저와의 상호작용에 대한 응답으로 `View`가 호출하게 될 텐데, 꼭 그렇지만은 않다. 예를 들어 필요한 데이터를 첫 진입시 초기화 할 때 서버의 API 응답에 대한 결과로 `Action`을 생성할 수도 있다.

#### What About that Dispatcher?
앞서 언급한 것과 같이, `Dispatcher`는 스토어들 간의 의존성 또한 관리할 수 있다. 이는 `Dispatcher` 클래스의 `waitFor()` 메서드를 통해 가능하게 된다. 이 메소드는 간단한 어플리케이션에서는 사용할 필요가 없지만, 어플리케이션이 복잡해진다면 꼭 사용하게 될 메소드 중에 하나이다.

TodoStore 의 상태를 변경하기 전에 의존하고 있는 다른 상태를 먼저 변경하려면 다음과 같이 구현할 수 있다.

```javascript
case 'TODO_CREATE':
  Dispatcher.waitFor([
    PrependedTextStore.dispatchToken,
    YetAnotherStore.dispatchToken
  ]);
  TodoStore.create(PrependedTextStore.getText() + ' ' + action.text);
  break;
```

`waitFor()` 는 디스패쳐 토큰이라고 불리는 디스패쳐 레지스트리 인덱스의 배열을 인자로 받는다. 이 토큰들을 통해 `waitFor()`를 호출한 스토어는 다른 스토어의 상태에 의존하여 자신의 상태를 어떻게 변경해야 할지 알 수 있다.

#### Dispatcher
디스패쳐는 등록된 콜백들에게 페이로드를 전파하기 위해 사용된다. 이는 양방향 구조의 일반적인 pub-sub과는 차이가 있다.
- 콜백은 특정 이벤트를 구독하고 있지 않다. 모든 페이로드는 등록된 모든 콜백으로 전달된다.
- 콜백은 그 일부나 혹은 전부가 다른 콜백이 실행될 때 까지 연기되어질 수 있다.

##### API
제공하는 API는 다음과 같다.

- `register(function callback):string`: 모든 페이로드에 대해 실행되어질 콜백을 등록, `waitFor()`에 사용가능한 토큰(식별자)를 반환
- `unregister(string id):void`: 주어진 토큰에 해당하는 콜백을 삭제
- `waitFor(array<string> ids): void`: 현재의 콜백이 실행되기전 인자로 받은 모든 콜백이 실행될 때까지 기다림, 페이로드를 디스패치항 응답으로의 콜백으로만 사용가능
> 이게 무슨말인지 모르겠다.

- `dispatch(object payload):void`: 등록된 모든 콜백을 실행
- `isDispatching(): boolean`: 현재 디스패칭 중인지



> https://haruair.github.io/flux/docs/overview.html
