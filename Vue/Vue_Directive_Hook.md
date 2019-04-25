# Vue 디렉티브와 Hook

이번엔 커스텀하게 만든 디렉티브가 예상치 못한 버그를 만들어내서 디버깅을 찍다보니 `unbind` 훅을 제대로 구현해 주지 않아 디렉티브 내부의 `innerHTML` 이 제대로 초기화되지 않아서 발생한 이슈였다. 이 참에 Vue 와 디렉티브에 관하여 정리하려 한다.

### Directive?

디렉티브는 `v-` 접두어를 가진 특별한 attribute 이다. 값으로 단일 자바스크립트 표현식을 받으며 하는 일은 이 표현식의 결과값이 변경되었을 때 '반응형'으로 DOM 에 side-effect 를 적용하는 것이다.

가장 처음 접하는 `v-if`가 디렉티브중 하나인데

```javascript
<p v-if="seen">Now you see me</p>
```
위 코드는 `seen` 의 값의 변경에 따라 반응하여 `seen` 이 truthy 한 값으로 변경되면 DOM에 `<p>` 엘리먼트를 추가하고 falsy 한 값으로 변경되면 DOM에 `<p>` 엘리먼트를 삭제하게 된다.

### Custom Directive?

사용자 정의 디렉티브 (custom directive) 란 말 그대로 개발자가 정의한 디렉티브이다. 즉, 전달받은 표현식의 값에 따라 반응형으로 DOM 에 side-effect 를 주는 컴포넌트라고 할 수 있겠다.

### 문제

Vue는 주어진 문자열을 `innerHTML` 에 넣어 HTML 렌더링 하는 디렉티브로 `v-html` 디렉티브를 제공한다. 다만 기본적인 XSS 를 방어할 수 있는 디렉티브는 따로 제공하지 않고, 그럴 생각도 없다고 한다.

> https://github.com/vuejs/vue/issues/7860

문제가 되는 경우는 NCR 을 렌더링 해야 하는 경우인데, 렌더링은 해야겠고 XSS는 막아야겠으니 `<` 와 `>` 만 escape 하는 커스텀 디렉티브를 만들어야겠다고 결심한다.

> NCR (Numeric Character Reference) 에 대해서는 다음을 참고하자. https://en.wikipedia.org/wiki/Numeric_character_reference

아마 다음과 같이 구현할 수 있을 것이다.

```javascript
function escapeTag(el, binding) {
  const fakeElement = document.createElement('div');
  fakeElement.innerHTML = binding.value;
  el.innerText = fakeElement.innerHTML;
}

Vue.directive('escape-tag', escapeTag);
```

```HTML
<div>
  <span class="test" v-escape-tag='"&#9822"'></span>
</div>
```

`v-escape-tag`에 `&#9822` 를 넣으면 의도한대로 `♞` 가 출력되는걸 볼 수 있다.

이 디렉티브가 virtual dom 과 만나면 굉장히 기묘한 일이 발생하게 된다.

```HTML
<div>
  <span v-if="!a" class="test" v-escape-tag="'&#9822'"></span>
  <span v-if="a" class="test">기본 텍스트</span>
</div>
```

다음과 같이 `a`의 값에 따라서 두 `<span>` 중 하나를 노출한다고 가정해보자.

`a` 가 처음에 false 일때는 '♞' 만 보이다가 true로 바뀌는 순간 '♞기본 텍스트' 로 노출되게 된다.
분명 다른 element 로 변경되었는데 기존에 있던 '♞'가 지워지지 않게 된것.

그 이유는 첫번째 `span` 에서 두번째 `span` 으로 변경될 때 뷰의 virtual dom 이 해당 element 를 재활용하기 때문인데, 디렉티브가 `unbind` 될 때의 훅이 설정되어 있지 않아서 ♞가 지워지지 않고 남아있게 되는것이다.

해결책은 간단한데, `Vue.directive(...)`의 인자로 함수가 아니라 각 훅마다 동작할 콜백이 정의된 객체를 넘기는 것이다.
