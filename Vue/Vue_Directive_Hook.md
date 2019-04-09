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
