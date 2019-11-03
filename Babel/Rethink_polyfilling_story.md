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

Every interested project should have their own polyfill provider: for example, babel-polyfill-provider-corejs3 or @ungap/babel-polyfill-provider.

**Polyfill injector** 는 바벨의 플러그인으로 프로그램의 추상 구문 트리를 분석한다. 이 플러그인은 글로벌 참조나 프로퍼티 접근등의 폴리필이 필요한 모든 표현식을 찾고, 이것을 처리할 수 있는 polyfill provider가 존재하는지 확인한다.

생태계는 바벨의 공식적인 패키지중 일부인 오로지 하나의 polyfill injector만 있으면 될것이다. 아마 ` @babel/preset-env` 에 포함되거나, `@babel/plugin-inject-polyfills` 라는 새로운 이름의 패키지가 생길 수도 있다.

### New user experience
사용자가 일부 제안되어진 ECMAScript를 사용하고 있고, `Intl` 을 통해 로컬라이징을 하며 `fetch`를 사용하고 있다고 가정해보자.

너무 많은 바이트의 폴리필을 로딩하는건 피하고 싶고, 범용적으로 사용되는 브라우저만 지원하고 사용하지 않는 폴리필은 로드되지 않기를 바랄때 다음과 같이 설정을 한다면 어떻겠는가?

```JavaScript
{
  "presets": [
    ["@babel/preset-env", { "targets": [">1%"] }]
  ],
  "plugins": [
    ["@babel/inject-polyfills", {
      "method": "usage-global",
      "providers": [
        "intl",
        "fetch",
        ["corejs3", { "proposals": true }]
      ]
    }],
  ]
}
```

### New developer experience
polyfill provider들이 안전하게 사용할 수 있는 새로운 cross-plugin communication protocol을 제공하기 위해서 우리는 communication이 어떻게 발생하는지 추상화 할 수 있는 몇가지 유틸리티를 제공해야 한다.

우리는 아마도 `@babel/helper-create-class-features-plugin` 에서 했던 것 처럼 `File`의 클래스 `Map` 에 공유된 엔트리를 사용할 것이다. 그러나 버그의 출저와 복잡성을 제한하기 위해 이를 개발자에게 노출해서는 안된다.

또한 우리는 `@babel/helper-polyfill-provider` 패키지를 새로 만들 거나 `@babel/plugin-inject-polyfills` 패키지에서 헬퍼들을 노출할 수 있다.

```JavaScript
import createPolyfillProvider, { injectImport } from "@babel/helper-polyfill-provider";

export default createPolyfillProvider((api, options) => {
  return {
    name: "object-entries-polyfill-provider",

    entryGlobal(name, targets, path) {
      if (name !== "object-entries-polyfill") return false;
      if (!supportsEntries(targets)) {
        injectImport("object-entries-polyfill/global", path);
      }
      path.remove();
      return true;
    },

    usageGlobal(name, targets, path) {
      if (name !== "Object.entries") return false;
      if (!supportsEntries(targets)) {
        injectImport("object-entries-polyfill/global", path);
      }
      return true;
    },

    usagePure(name, targets, path) {
      if (name !== "Object.entries") return false;
      if (!supportsEntries(targets)) {
        const id = path.scope.generateUidIdentifier("Object.entries");
        injectImport("object-entries-polyfill/local", path, id);
        path.parentPath.replaceWith(id);
      }
      return true;
    }
  }
});
```

`createPolyfillProvider` 함수는 폴리필 플러그인 팩토리를 가져와 적절한 바벨 플러그인을 만들기 위해 랩핑한다.
팩토리 함수는 다른 플러그인과 동일하게 바벨의 API 인스턴스와 플러그인 옵션을 인자로 받는다.

반환되어지는 객체는 다른 일반 플러그인이 반환하는 값과는 다르게 폴리필 구현방법에 따른 선택적인 메소드들을 포함하고 있을 것이다.

우리는 해당 객체에 다른 키는 허용하지 않으므로 쉽게 새로운 폴리필 주입방법을 도입할 수 있을 것이다. (예를 들면 `inline` 같은)
