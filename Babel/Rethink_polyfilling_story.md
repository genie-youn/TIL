# 우리 폴리필에 대해서 다시 생각해 봅시다.

### 들어가며
바벨은 왠지모르게 어렵게 느껴진다. 레퍼런스도 읽기 좀 어려운것 같고, 똑같은 결과를 얻기 위한 설정들도 여러가지들이 있다. 얼마 전, 하위 브라우저들이 `fxjs`를 지원하지 않아 추가로 심어주어야 할 폴리필들을 찾고 이걸 어떻게 심어줘야 `best practice` 일까 찾아보다가 흥미로는 이슈를 하나 찾아서 천천히 읽어보려 한다.

> 원글의 스레드는 이슈는 [다음](https://github.com/babel/babel/issues/10008)을 참고한다.

### 문제
지난 2년 반동안, `@babel/preset-env`는 단순히 지원하지 않는 기능을 트랜스파일링 하지 않을 뿐더러, 불필요한 `core-js` 의 폴리필을 포함하지 않음으로써, 번들크기를 줄이는데 잠재력을 보여주였다.

현재 바벨을 `core-js` 의 폴리필을 주입하는데 세가지 다른 방법을 가지고 있다.

- `@babel/preset-env` 의 `useBuiltIns:"entry"` 옵션을 사용하는것. 이것은 목표로 설정한 브라우저들이 기본적으로 지원하지 않는 모든 ECMAScript의 기능들에 대한 폴리필을 주입한다.
- `useBuiltIns: "usage"` 옵션을 사용하여 소스코드에서 사용하고 있는 목표한 브라우저가 지원하지 않는 ECMAScript 의 기능들만 폴리필을 주입한다.
- `@babel/plugin-transform-runtime` 을 사용하여 `core-js` 가 지원하는 모든 ECMAScript 폴리필을 주입한다. 이는 "pure" 하며 전역 스코프를 오염시키지 않는다. 이는 주로 라이브러리 개발자들이 많이 사용한다.

자바스크립트 생태계에서 우리의 위치는 우리가 이러한 최적화를 훨씬 더 촉진할 수 있도록 한다.
Our position in the JavaScript ecosystem allows us to push these optimization even further.
> 뭔소리지

`@babel/plugin-transform-runtime` 는 일부 사용자에게 `useBuiltIns` 을 사용하는것 보다 더 큰 이점이 있지만, 목표 환경을 고려하지 않는다. 2019년에 `Array.prototype.forEach` 의 폴리필이 필요한 사람을 아마 얼마 없을 것이다.

추가로 왜 우리는 자동으로 필요한 폴리필을 주입하는것을 `core-js` 를 통해서만 가능하도록 제한해야 하는가?

DOM polyfills, Intl polyfills, 등등의 무수히 많은 다른 웹 플랫폼 API를 위한 폴리필들이 존재한다.

게다가 모든 사람들이 `core-js` 를 사용하기를 원하진 않는다. 다른 트레이드 오프 (소스의 사이즈 vs 스펙의 충실함) 에 놓인 많은 합당한 폴리필들이 존재하며, 때로는 몇몇 사용자에겐 이 폴리필들이 더 나은 선택이 될 수 있다.

만약 폴리필을 주입하는 로직이 가능하거나 필요한 폴리필에 대한 실제 데이터와 관련이 없어 독립적으로 사용하고 개발가능하면 어떨까?

### Concept
**Polyfill provider** 는 새로운 바벨의 패키지이다. 이는 어느 자바스크립트 표현식이 폴리필을 필요로 하는지, 폴리필을 어디서 로드하고 어떻게 적용해야 하는지를 지정하는데 사용된다.

동시에 여러 Polyfill provider 가 사용될 수 있다. 예를들어 사용자는 ECMAScript 폴리필을 제공하는 Provider 와 DOM과 관련된 폴리필을 제공하는 Provider를 동시에 로드할 수 있다.

**Polyfill providers** 는 폴리필을 주입하기 위한 세개의 메소드를 제공한다.

- `entry-global`: 현재 `@babel/preset-env` 의 `useBuiltIns: "entry"` 옵션과 동일하다.
- `usage-global`: 현재 `@babel/preset-env` 의 `useBuiltIns: "usage"` 옵션과 동일하다.
- `usage-pure`: 현재 `@babel/plugin-transform-runtime` 가 폴리필을 주입하는 방식과 동일하다.
