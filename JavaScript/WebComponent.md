# WebComponent

웹 컴포넌트는 특정 기능을 캡슐화하여 재사용 가능한 커스텀 엘리먼트를 생성하고 웹 앱에서 활용할 수 있게 해주는 다양한 기술들의 표준

코드를 재사용하는건 좋지만 전통적인 마크업 구조에선 쉽지 않았다.
크게 세가지 주요 기술이 있는데,

1. 커스텀 엘리먼트
  : 사용자가 정의한 엘리먼트와 해당 엘리먼트의 동작을 정의할 수 있게 해주는 Javascript API 들의 집합

2. 섀도우 돔
  : 메인 도큐먼트의 DOM 으로부터 독립적으로 렌더링 되는 캡슐화 된 DOM 트리와 이를 제어하기 위한 Javascript API 들의 집합

3. HTML 템플릿
  : `<template>` 와 `<slot>` 엘리먼트 렌더링된 페이지에 나타나지 않는 마크업 템플릿을 작업하게 해준다.

다음과 같이 구현한다.

1. ES6의 `Class` 를 사용해 커스텀 엘리먼트의 기능을 명시한 클래스를 생성한다.
2. `CustomElementRegistry.define()` 함수를 이용해 커스텀 엘리먼트를 등록하고, 정의할 엘리먼트 이름, 기능을 정의한 클래스, 상속할 엘리먼트를 전달한다.
3. 필요한 경우 `Element.attachShadow()` 를 사용해 `ShadowDom` 을 커스텀 엘리먼트에 추가한다. 일반적인 DOM 메소드를 사용해 자식 엘리먼트, 이벤트 리스너 등을 shadow DOM 에 추가한다.
4. 필요한 경우 `<template>` 과 `<slot>` 을 사용해 HTML 템플릿을 정의한다. 다시 일반적인 DOM 메소드를 사용해 템플릿을 클론하고 shadow DOM 에 추가한다.
5. 일반적인 HTML element 처럼 사용한다.

##### 커스텀 엘리먼트 생성하기

커스텀 엘리먼트의 컨트롤러는 [CustomElementRegistry](https://developer.mozilla.org/ko/docs/Web/API/CustomElementRegistry) 객체를 사용한다.
`customElements.define(name, constructor, options);` 함수를 통해 새로운 커스텀 엘리먼트를 등록할 수 있다.
> `customElements.define(name, constructor, options);`
>
> name : 커스텀 엘리먼트 태그의 이름. 반드시 하이픈(-)을 포함해야 한다. (한 단어가 될 수 없음) <br>constructor : 커스텀 엘리먼트의 기능을 캡슐화한 (ES6의) 클래스 <br> // 이거 ES6 클래스만 되나? 일반 생성자 함수는 안되고?
> options(optional) : 옵션을 주입할 객체, 현재는 `extends` 하나의 속성만 지원. 어떤 엘리먼트를 상속할 지 옵션으로 줄 수 있다.

```Javascript
class WordCount extends HTMLParagraphElement {
  constructor() {
    // 항상 생성자에서 super는 처음으로 호출됩니다
    super();
    // 엘리먼트의 기능들은 여기에 작성합니다.
    ...
  }
}
```



커스텀 엘리먼트는 표준 HTML 엘리먼트를 상속하는 Customized built-in elements 와 상속하지 않고 오로지 사용자 자율적으로 정의된 Autonomous custom elements 로 나누어진다.



### 예제

##### Autonomous custom elements

아이콘에 포커스를 가져가면 정보를 툴팁으로 나타내는 커스템 엘리먼트를 만들어보자
우선 `PopUpInfo` 라는 커스텀 엘리먼트의 기능이 정의된 `HTMLElement` 를 상속하는 클래스를 만든다. Autonomous custom elements 는 보통 `HTMLElement` 를 상속하게 된다. 

```javascript
class PopUpInfo extends HTMLElement {
  constructor() {
    // Always call super first in constructor
    super();

    // write element functionality in here
    ...
  }
}
```

프로토 타입 체인을 올바르게 만들기 위해 생성자의 첫 부분엔 항상 `super()` 를 호출해야 한다.
생성자 내부에서 커스텀 엘리먼트가 인스턴트화 될 때 인스턴스로써 가져야할 모든 기능들을 정의한다. 이 경우 섀도우 루트를 커스텀 엘리먼트에 붙이고, 몇개의 DOM을 조작하는 함수를 통해 커스텀 엘리먼트 내부에 섀도우 DOM의 구조를 세운 후 마지막으로 섀도우 루트에 CSS로 스타일을 정의한다.



HTMLElement ??

https://developer.mozilla.org/ko/docs/Web/Web_Components#%EB%AA%85%EC%84%B8
https://www.html5rocks.com/ko/tutorials/webcomponents/shadowdom/
https://www.html5rocks.com/ko/tutorials/webcomponents/shadowdom-201/
https://html.spec.whatwg.org/multipage/scripting.html#the-template-element
태그 : 웹컴포넌트