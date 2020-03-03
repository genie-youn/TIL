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

##### Example
예를들어 항공시스템에서 목적지 국가를 선택하면 기본 도시가 자동으로 선택되는 입력 폼을 가정해보자.

```javascript
var flightDispatcher = new Dispatcher();
// Keeps track of which country is selected
var CountryStore = {country: null};
// Keeps track of which city is selected
var CityStore = {city: null};
// Keeps track of the base flight price of the selected city
var FlightPriceStore = {price: null};
```

사용자가 도시를 선택했을때 페이로드를 디스패치하게 된다.
```javascript
flightDispatcher.dispatch({
  actionType: 'city-update',
  selectedCity: 'paris'
});
```

이러한 페이로드는 `CityStore` 에서 다음과 같이 처리된다.
```javascript
flightDispatcher.register(function(payload) {
  if (payload.actionType === 'city-update') {
    CityStore.city = payload.selectedCity;
  }
});
```

사용자가 국가를 선택했을때 다음과 같이 페이로드를 디스패치하게 된다.

```javascript
flightDispatcher.dispatch({
  actionType: 'country-update',
  selectedCountry: 'australia'
});
```

이 페이로드는 두 스토어 모두에서 처리된다.

```javascript
CountryStore.dispatchToken = flightDispatcher.register(function(payload) {
  if (payload.actionType === 'country-update') {
    CountryStore.country = payload.selectedCountry;
  }
});
```

`CountryStore`에 콜백을 등록할때 반환받은 토큰을 통해 `waitFor` 메서드를 사용하여 `CityStore`가 필요한 데이터를 사용하기 위해 `CountryStore`가 먼저 업데이트 되는것을 보장할 수 있다.

```javascript
CityStore.dispatchToken = flightDispatcher.register(function(payload) {
  if (payload.actionType === 'country-update') {
    // `CountryStore.country` may not be updated.
    flightDispatcher.waitFor([CountryStore.dispatchToken]);
    // `CountryStore.country` is now guaranteed to be updated.
    // Select the default city for the new country
    CityStore.city = getDefaultCityForCountry(CountryStore.country);
  }
});
```

`waitFor()`는 다음과 같이 체이닝해서 사용할 수도 있다.
```javascript
FlightPriceStore.dispatchToken =
  flightDispatcher.register(function(payload) {
    switch (payload.actionType) {
      case 'country-update':
      case 'city-update':
        flightDispatcher.waitFor([CityStore.dispatchToken]);
        FlightPriceStore.price =
          getFlightPriceStore(CountryStore.country, CityStore.city);
        break;
  }
});
```

위와 같이 작성할 경우 `country-update` 페이로드는 `CountryStore`, `CityStore`, `FlightPriceStore` 순서로 콜백이 처리되는 것을 보장한다.

### Flux Utils
Flux Utils는 Flux를 처음 시작하는데 도움을 주는 유틸 클래스들을 모아둔 것이다. 이 유틸들은 간단한 Flux 애플리케이션을 위한 것으로 모든 케이스에 대한 기능들을 전부 갖추고 있는 프레임워크가 아니다. 만약 기능이 부족하다고 느껴진다면 다른 훌륭한 Flux 프레임워크를 찾아보는걸 권장한다.

#### Usage
Flux Utils는 세 가지 클래스를 포함하고 있다.
- 1. Store
- 2. ReduceStore
- 3. Container

Flux Utils 의 기본적인 사용은 다음과 같다.
```javascript
import { ReduceStore } from 'flux/utils';
class CounterStore extends ReduceStore<number> {
  getInitialState(): number {
    return 0;
  }
  reduce(state: number, action: Object): number {
    switch (action.type) {
      case 'increment':
        return state + 1;
      case 'square':
        return state * state;
      default:
        return state;
    }
  }
}
```

#### Best Practice
아래 클래스를 사용함에 있어 몇가지 Best Practice를 소개한다.

##### Stores
- 데이터 캐싱
- 데이터에 접근 가능한 `public getters` 제공
- `dispatcher` 로부터의 특정 액션에 대해 응답
- 데이터가 변경될때 변경에 대한 이벤트를 방출
- 오로지 `dispatch` 를 통해서만 변경을 방출

##### Actions
`setter`가 아닌 사용자의 액션을 기술 (e.g. `set-page-id`가 아닌 `select-page`)

##### Containers
- `view`를 제어하는 리액트 컴포넌트
- 주요한 역할은 스토어로부터 정보를 모으고, 컴포넌트의 상태로 저장하는 일
- `props`를 갖지 않으며 UI와 관련된 로직 또한 없다.

##### Views
- `Container`로 부터 제어를 당하는 리액트 컴포넌트
- UI와 렌더링에 관련된 모든 로직을 포함하고 있음
- 모든 정보와 콜백을 `props`를 통해 전달받음

### API

#### `Store`

##### `constructor(dispatcher: Dispatcher)`

##### `addListener(callback: Function): {remove: Function}`
스토어에 변경이 생겼을 때 발생한 콜백을 실행할 리스너를 스토어에 등록한다. `remove()` 함수에 반환받은 토큰을 함께 호출하면 리스너를 삭제할 수 있다.

##### `getDispatcher(): Dispatcher`
스토어에 등록되어 있는 디스패쳐를 반환한다.

##### `getDispatchToken(): DispatchToken`
디스패쳐가 스토어를 구분하는데 사용되는 디스패쳐 토큰을 반환한다. 이 토큰을 사용하여 이 스토어에 대한 `waitFor()`를 호출할 수 있다.

##### `hasChanged(): boolean`
현재 디스패치에 의해 스토어가 변경됐는지를 반환한다. 디스패칭 도중에만 호출할 수 있으며 이를 사용하여 다른 스토어의 데이터 의존하는 스토어를 구현할 수 있다.

##### `__emitChange(): void`
이 저장소가 변경되었음을 알리는 이벤트를 모든 리스너에게 전파한다. 오로지 디스패칭 하는 도중에만 호출되며 스토어의 `__onDispatch` 함수의 마지막에 중복되지 않고 처리된다.

##### `onDispatch(payload: Object): void`
서브클래스는 반드시 이 메소드를 오버라이드 해야한다. 이는 스토어가 디스패쳐로부터 액션을 어떻게 받을것인지를 나타낸다. 모든 상태 변경 로직은 반드시 이 메소드가 실행되는 동안 완료되어야 한다.

#### `ReduceStore<T>`
`Store` 클래스를 상속받은 클래스

##### `getState(): T`
현재 스토어의 모든 상태를 노출하는 getter. 만약 스토어의 상태가 불변하지 않다면 반드시 상태를 직접 노출하지 않도록 오버라이드해야 한다.

##### `getInitialState(): T`
스토어의 상태를 초기화한다. 이 메서드는 스토어를 생성하는 동안 한번 호출된다.

##### `reduce(state: T, action: Object): T`
현재의 상태와 액션을 소비하여 스토어를 새로운 상태로 변경한다. 모든 서브클래스는 반드시 이 메소드를 구현해야하며 이 메소드는 반드시 순수하고 사이드이펙트가 없어야만 한다.

##### `areEqual(one: T, two: T): boolean`
두 버전의 상태가 동일한지를 판단한다. 상태가 불변하다면 이 메소드를 오버라이드하지 않아도 된다.

##### `Doesn't Need to Emit a Change`
`ReduceStore`를 상속받은 모든 스토어는 명시적으로 `reduce()` 함수 내에서 변경 이벤트를 뱉을 필요가 없다. (하지만 여전히 이러한 방식이 필요하다면 가능하다.) 디스패치 이전의 상태와 이후의 상태를 비교하여 변경이 생겼다면 자동으로 변경 이벤트를 뱉는다. 만약 이러한 방식을 제어할 필요가 있다면 (아마 상태가 불변이지 않을 것이다.) `areEqual()`을 오버라이드해야한다.

#### `Container`

##### `create(base: ReactClass, options: ?Object): ReactClass`
`create` 는 리액트 클래스를 연관된 스토어가 변경될 때 자신의 상태를 업데이트하는 컨테이너로 변경한다. 인자로 주어진 기본 클래스는 반드시 정적 메서드로 `getStores()` 와 `calculateState()`를 가지고 있어야 한다.

```javascript
import { Component } from 'react';
import { Container } from 'flux/utils';
class CounterContainer extends Component {
  static getStores() {
    return [CounterStore];
  }
  static calculateState(prevState) {
    return {
      counter: CounterStore.getState(),
    };
  }
  render() {
    return <CounterUI counter={this.state.counter} />;
  }
}
const container = Container.create(CounterContainer);
```

특정 동작을 제어하기 위하여 컨테이너를 생성할 때 추가적인 옵션을 부여할 수 있다.

- 컨테이너는 순수해야 한다. - 기본적으로 컨테이너는 순수해야 한다. 순수하는 것은 주입받은 프로퍼티와 상태가 변경되지 않는다면 (`shallowEquals()` 로 결정된다.) 다시 렌더링되지 않는다는 것을 의미한다. 이러한 동작은 `create()`에 두번째 인자로 `{pure: false}` 옵션을 전달하면 비활성화 할 수 있다.
- 컨테이는 프로퍼티에 접근할 수 없다. - 기본적으로 컨테이너는 어떠한 프로퍼티접근할 수 없다. 이는 두가지 이유가 있다. 하나는 성능적인 이유이고, 두번째는 컨테이너의 재사용을 보장하고 컴포넌트 트리 전체에 걸쳐 프로퍼티를 끼워넣을 필요가 없도록 하기 위한 것이다. 몇가지 유효한 상황이 있다. 스토어의 상태와 프로퍼티 모두를 기준으로 상태를 결정해야 하는 몇 가지 유효한 상황이 있다. 이러한 경우 `create()` 의 두번째 인자로 `{withProps: true}` 옵션을 전달하면 된다. 이렇게 하면 `calculateState()`의 두번째 인자로 컴포넌트의 프로퍼티가 노출되게 된다.

If you are unable to utilize react classes most of this functionality is also mirrored in a mixin. import {Mixin} from 'flux/utils';

만약 리액트 클래스를 활용할 수 없다면 대부분의 기능은 `mixin` 에도 구현되어 있어 다음과 같이 사용하면 된다.

```Javascript
import {Mixin} from 'flux/utils';
```

### Using Flux with React Hooks
리액트 16.8 에서 소개된 [Hooks](https://reactjs.org/docs/hooks-intro.html)에서 Flux와 Flux Utils의 대부분의 기능들은 [`useContext`](https://reactjs.org/docs/hooks-reference.html#usecontext) 와 [`uesReducer`](https://reactjs.org/docs/hooks-reference.html#usereducer)를 사용하여 구현할 수 있다.

### Existing Projects with Store/ReduceStore
이미 존재하는 프로젝트에서 Flux Util's Stores 를 계쏙 사용해야 한다면 [`flux-hooks`](https://github.com/Fieldscope/flux-hooks) 를 사용할 수 있다. 컨테이너의 `calculateState` 와 유사한 API를 제공하는 `useFluxStore` 를 사용하여 스토어에 접근하게 된다.


> https://haruair.github.io/flux/docs/overview.html
