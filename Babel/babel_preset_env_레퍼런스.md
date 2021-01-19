# @babel/preset-env

@babel/preset-env는 최신의 자바스크립트를 목표로 하는 브라우저 환경에 맞춰 세세한 문법 변환 (과 브라우저 폴리필을) 신경쓸 필요 없이 사용할 수 있도록 해주는 똑똑한 프리셋이다.

이는 개발인생 삶의 질과 높여주고 자바스크립트 번들 사이즈를 줄여준다.

## 설치
npm을 통하여 설치할 수 있다.

```shell
npm install --save-dev @babel/preset-env
```

혹은 yarn을 사용할 수 있다.

```shell
yarn add @babel/preset-env --dev
```

## 동작 방식

[browserslist](https://github.com/browserslist/browserslist), [compat-table](https://github.com/kangax/compat-table), [electron-to-chromium](https://github.com/Kilian/electron-to-chromium) 와 같은 몇몇 멋진 오픈소스 프로젝트가 없었더라면 `@babel/preset-env` 는 불가능했을 것이다.

우리는 이러한 데이터 소스를 활용하여 우리가 지원하려는 대상 환경들이 자바스크립트 문법이나 브라우저 기능들을 어떤 버전부터 지원하는지 맵핑한 [데이터](https://github.com/babel/babel/blob/master/packages/babel-compat-data/data/plugins.json)를 생성하여 관리하고, babel transform plugin과 core-js의 polyfill들에 맵핑한다.

> `@babel/preset-env`는 더 이상 `stage-x` 플러그인을 지원하지 않는 점에 유념할것.

`@babel/preset-env`는 지정한 대상 환경을 가져와서 위 맵핑 데이터와 비교하여 플러그인 목록을 컴파일하고 바벨에 전달한다.

## Browserslist 통합

브라우저, 또는 일렉트론 기반의 프로젝트에선 대상 환경을 지정하는데 `.browserslistrc` 를 사용하는것을 추천한다. `autoprefixer`, `stylelint`, `eslint-plugin-compat` 등과 같은 도구를 사용한다면 이미 이 파일이 있을수도 있다.

기본적으로 `@babel/preset-env` 은 [targets](https://babeljs.io/docs/en/babel-preset-env#targets) 나 [ignoreBrowserslistConfig](https://babeljs.io/docs/en/babel-preset-env#ignorebrowserslistconfig) 옵션을 따로 설정하지 않는 한, [browserslist config sources](https://github.com/browserslist/browserslist#queries) 를 사용하게 된다.

> browserslist 의 defaults query 를 사용하는 경우 (명시적으로든, browserslist config를 따로 생성하지 않았든) preset-env가 어떻게 동작하는지 [No targets](https://babeljs.io/docs/en/babel-preset-env#no-targets)을 참고할것.

예를들어 시장 점유율이 0.25% 가 넘는 (IE 10이나 블랙베리등 보안 업데이트 이슈가 있는 브라우저를 제외한) 브라우저만을 대상으로 동작할 수 있도록 폴리필을 추가하고 코드를 변환하기를 원한다면 다음과 같이 작성할 수 있다.

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "entry"
      }
    ]
  ]
}
```

#### browserslist
```json
> 0.25%
not dead
```

또는

#### package.json
```json
"browserslist": "> 0.25%, not dead"
```

> `v7.4.5` 부터 browserslist query는 [mobileToDesktop: true](https://github.com/browserslist/browserslist#js-api) 옵션과 함께 동작하게 된다. 쿼리가 실행되는 스냅샷을 생성해보고 싶다면 다음 명령어를 사용할 수 있다. `npx browserslist --mobile-to-desktop ">0.25%, not dead"`.

## 옵션

프리셋의 옵션에 대한 추가적인 정보가 필요하다면 [preset options](https://babeljs.io/docs/en/presets#preset-options) 문서를 참고할 것.

### `targets`
`string | Array<string> | { [string]: string }`, 기본값 `{}`

프로젝트가 지원할 대상 환경을 명시한다.

[browserslist-compatible](https://github.com/ai/browserslist) query 또한 사용할 수 있다. (with [caveats](https://babeljs.io/docs/en/babel-preset-env#ineffective-browserslist-queries)):

```json
{
  "targets": "> 0.25%, not dead"
}
```

또는 지원할 최소 버전을 각각 명시할 수 있다. `chrome`, `opera`, `edge`, `firefox`, `safari`, `ie`, `ios`, `android`, `node`, `electron` 를 설정할 수 있다.

```json
{
  "targets": {
    "chrome": "58",
    "ie": "11"
  }
}
```

#### No targets

`preset-env` 의 목표중 하나는 기존에 `preset-latest` 를 사용중이던 사용자들이 쉽게 `preset-env` 로 전환할 수 있도록 돕는것이다. 이를 위해 no targets 으로 설정하면 `preset-latest` 와 비슷하게 동작한다. (ES2015-ES2020 에 해당하는 모든 코드를 ES5 에서 동작할 수 있도록 변환한다.)

> 지원한 환경/버전을 특정하여 명시하여 얻는 이점이 많기 때문에 `preset-env` 를 이러한 방식으로 사용하는건 권장하지 않는다.

```json
{
  "presets": ["@babel/preset-env"]
}
```

이러한 이유로 `preset-env`의 no targets 시 동작은 [browserslist](https://github.com/browserslist/browserslist#queries) 와는 다르다. (바벨 혹은 browserslist config(s)에 no targets 로 설정되었을 때 `defaults` query를 사용하지 않는다.) 만약 `defaults` query를 사용하고 싶다면 명시적으로 target 에 전달해주어야 한다.

```json
{
  "presets": [["@babel/preset-env", { "targets": "defaults" }]]
}
```

이러한 동작이 이상적이지 않다는걸 우리도 인지하고 있으며 Babel v8 에서 다시 검토할 예정이다.

#### `targets.esmodules`
`boolean`

ES Modules (https://www.ecma-international.org/ecma-262/6.0/#sec-modules) 을 지원하는 브라우저를 대상으로 할 수 있다. 이 옵션을 설정하면 `browsers` 필드는 무시된다. `<script type="module"></script>` 와 함께 사용하여 사용자로 하여금 꼭 필요한 최소한의 스크립트만 내려받을 수 있도록 한다. (https://jakearchibald.com/2017/es-modules-in-browsers/#nomodule-for-backwards-compatibility).

> esmodules 을 활성화 할 경우 browsers 는 무시된다는것을 명심.

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "esmodules": true
        }
      }
    ]
  ]
}
```

#### `targets.node`
`string | "current" | true`

만약 현재 노드의 버전에 맞춰 컴파일하고 싶다면 `"node": true` 혹은 `"node": "current"` 로 설정해 주면 된다. `"node": process.version.node` 와 동일하게 동작한다.

#### `targets.safari`
`string | "tp"`

사파리의 [technology preview](https://babeljs.io/docs/en/babel-preset-env) 버전에 맞춰 컴파일하고 싶다면 `"safari": "tp"` 로 설정해 주면 된다.

#### `targets.browsers`
`string | Array<string>`
[browserslist](https://github.com/browserslist/browserslist) 를 사용하여 지원할 브라우저 환경을 지정한다. (ex: last 2 versions, > 5%, safari tp)

browsers 옵션으로 설정한 값은 `targets` 의 설정 결과에 의해 덮어써진다는 점에 명심할것.

> 이 옵션은 추후 삭제될 예정이므로 `targets`에 직접 쿼리를 작성하는걸 권장한다.

#### `bugfixes`
`boolean` 기본값: `false`
`v7.9.0` 에 추가됨

> Babel 8 부터는 `true`가 기본값

이거 뭔 소리지?

By default, `@babel/preset-env` (and Babel plugins in general) grouped ECMAScript syntax features into collections of closely related smaller features.

기본적으로 `@babel/preset-env` (와 일반적인 Babel 플러그인들은) ECMAScript의 문법들을 밀접하게 연관된 문법들의 작은 모음으로 그룹화했다.

These groups can be large and include a lot of edge cases, for example "function arguments" includes destructured, default and rest parameters.

이러한 그룹은 클 수 있으며 많은 예외 경우를 포함할 수 있다. 예를들어 "function arguments"는 destructured, default and rest parameters 를 포함한다.

From this grouping information, Babel enables or disables each group based on the browser support target you specify to `@babel/preset-env`’s `targets` option.

이 그룹화된 정보들로부터, Babel 은 각각의 그룹을 `@babel/preset-env`의 `targets` 옵션에 명시한 지원할 브라우저를 기준으로 활성화할지, 비활성화할지 판단한다.

When this option is enabled, `@babel/preset-env` tries to compile the broken syntax to the closest non-broken modern syntax supported by your target browsers.

이 옵션이 활성화되면 `@babel/preset-env` 는 망가진 문법을 대상으로 설정한 브라우저가 지원하는 가장 가까운 안망가진 문법으로 컴파일을 시도한다???

Depending on your targets and on how many modern syntax you are using, this can lead to a significant size reduction in the compiled app.

지원하려는 브라우저와 얼마나 많은 모던한 문법을 컴파일된 앱의 사이즈를 확연히 줄일 수 있다.

This option merges the features of [@babel/preset-modules](https://github.com/babel/preset-modules) without having to use another preset.

이 옵션은 다른 프리셋 설정을 사용하지 않고도 [@babel/preset-modules](https://github.com/babel/preset-modules) 의 기능들을 병합한다..?

#### `spec`
`boolean`, defaults to `false`

Enable more spec compliant, but potentially slower, transformations for any plugins in this preset that support them.

> ???

#### `loose`
`boolean`, defaults to `false`

preset의 ["loose" transformations](https://2ality.com/2015/12/babel6-loose-mode.html) 를 지원하는 모든 플러그인의 해당 기능을 활성화시킨다.

#### `modules`
`"amd" | "umd" | "systemjs" | "commonjs" | "cjs" | "auto" | false`, defaults to `"auto"`.

ES 모듈 문법을 다른 모듈 타입으로 변환하도록 활성화한다. `cjs`는 `commonjs`의 별칭이다.

Setting this to `false` will preserve ES modules. Use this only if you intend to ship native ES Modules to browsers. If you are using a bundler with Babel, the default `modules: "auto"` is always preferred.

`false`로 설정할 경우 ES 모듈을 유지하므로 네이티브 ES 모듈을 브라우저로 전달하려는 경우에만 `false`로 설정한다. 만약 바벨과 함께 번들러를 사용한다면 기본값인 `modules: "auto"`를 권장한다.

##### `modules: "auto"`

기본적으로 `@babel/preset-env` 는 ES 모듈과 모듈 기능들 (예를 들어 `import()`) 을 변환할지 말지를 결정하는데 [caller](https://babeljs.io/docs/en/options#caller) 데이터를 사용한다.

일반적으로 `caller` 데이터는 번들러 플러그인 (`babel-loader`, `@rollup/plugin-babel` 같은) 에서 지정된다. 그러므로 `caller` 데이터를 직접 전달하는건 권장하지 않는다.

#### `debug`
`boolean`, defaults to `false`

`preset-env`로 활성화된 목표로 설정한 환경에 필요한 폴리필과 변환 플러그인을 `console.log` 로 출력한다.

#### `include`
`Array<string|RegExp>`, defaults to `[]`

항상 포함될 플러그인의 목록을 정의한다.
- [Babel plugins](https://github.com/babel/babel/blob/master/packages/babel-compat-data/scripts/data/plugin-features.js) - (`@babel/plugin-transform-spread`) 와 프리픽스가 없는 형식 (`plugin-transform-spread`) 모두 지원한다.
- Built-ins ([core-js@2](https://github.com/babel/babel/blob/master/packages/babel-preset-env/src/polyfills/corejs2/built-in-definitions.js) 와 [core-js@3](https://github.com/babel/babel/blob/master/packages/babel-preset-env/src/polyfills/corejs3/built-in-definitions.js), 예를 들어 `es.map`, `es.set`, `es.object.assing`)

플러그인 이름은 전체를 기술해도 되지만 부분적으로 정의해도 된다. (혹은 `RegExp`를 사용할 수 있다.)

입력받을 수 있는 형식은 다음과 같다.
- 전체 이름 (`string`): `"es.math.sign"`
- 부분적인 이름 (`string`): `"es.math.*"` (모든 `es.math` 프리픽스가 붙은 플러그인)
- `RegExp` 객체: `/^transform-.*$` 또는 `new RegExp("^transform-modules-.*")`

이 옵션은 네이티브 구현체에 버그가 있거나 지원되는 기능과 지원되지 않는 기능이 합쳐져 있어 정상적으로 동작하지 않을 때 유용하다.

예를들어 Node 4는 네이티브 클래스는 지원하지만 전개 연산자는 지원하지 않는다.
만약 `super`의 파라미터를 전개연산자로 호출하면 (`super(...args)`) `@babel/plugin-transform-classes` 변환은 `include`에 포함되어야 한다. 이 외에는 `super` 와 함께 쓰인 전개 연산자를 변환할 방법은 없다.

> 왜지...

> NOTE: `include` 와 `exclude` 옵션은 [plugins included with this preset](https://babeljs.io/docs/en/babel-preset-env#include) 에만 동작한다. 예를들어 `@babel/plugin-proposal-do-expressions` 를 `include` 하거나 `@babel/plugin-proposal-function-bind` 를 `exclude` 하게 되면 에러를 던진다. 이런 플러그인을 사용하려면 프리셋에 `include` *하지말고* "[plugins](https://babeljs.io/docs/en/options#plugins)" 옵션에 직접 추가하여야 한다.

#### `exclude`
`Array<string|RegExp>`, defaults to `[]`

항상 제외될 플러그인의 목록을 정의한다.
제외 가능한 플러그인은 `include` 옵션과 동일하다.

이 옵션은 "변환 블랙리스트" 를 만드는데 유용하다. 예를들어 제너레이터를 사용하지 않아 `regeneratorRuntime`이 추가되는걸 원하지 않는다거나 (`useBuiltIns`를 사용하는 경우) [바벨의 `async-to-gen`](https://babeljs.io/docs/en/babel-plugin-proposal-async-generator-functions) 대신 [`fast-async`](https://github.com/MatAtBread/fast-async) 와 같은 플러그인을 사용하기 위해 `@babel/plugin-transform-regenerator` 를 `exclude`에 추가해 사용하지 않을 수 있다.

#### `useBuiltIns`
`"usage"` | `"entry"` | `false`, defaults to `false`.

이 옵션은 `@babel/preset-env`가 폴리필을 어떻게 다룰지를 설정한다.

`usage`나 `entry` 옵션이 사용되면, `@babel/preset-env` 는 `core-js` 모듈에 대한 참조를 직접적으로 추가한다. 이는 `core-js` 가 접근 가능해야한다는걸 의미한다.

`@babel/polyfill`이 7.4.0에 deprecated 되면서 [`corejs`](https://babeljs.io/docs/en/babel-preset-env#corejs) 의 버전을 설정하여 직접 추가하는것을 권장하고 있다.

```shell
npm install core-js@3 --save

# or

npm install core-js@2 --save
```

##### `useBuiltIns: 'entry'`

> NOTE: 앱 전체에 걸쳐 딱 한번만 `import "core-js";`와 `import "regenerator-runtime/runtim";` 을 사용해야만 한다. 두번 `import`하게 되면 예외를 반환한다. 만약 `@babel/polyfill`을 사용중이라면 이것이 이미 `core-js`와 `regenerator-runtime`을 포함하고 있기 때문에 마찬가지로 예외를 반환하게 된다. 이러한 패키지를 여러번 `import` 하는 것은 글로벌 스코프의 충돌을 비롯한 여러 추적하기 어려운 문제들을 만들어낸다. 그렇기에 이러한 `import` 를 포함하는 단일 엔트리 파일을 생성하여 사용할 것을 권장한다.

이 옵션은 `import "core-js/stable;"` 과 `import "regenerator-runtime/runtime;"` 구문을 (혹은 `require("core-js")`와 `require("regenerator-runtime/runtime")`) 을 목표한 환경에 따라 필요한 개별 모듈만을 `import` 하는 구문으로 대체하는 새로운 플러그인을 활성화시킨다.

##### In
```javascript
import "core-js";
```

##### Out (different based on environment)
```javascript
import "core-js/modules/es.string.pad-start";
import "core-js/modules/es.string.pad-end";
```

`"core-js"`를 `import`하면 가능한 모든 ECMAScript 기능의 폴리필을 전부 로드한다. 만약 그중 일부만 필요하다는것을 안다면, `core-js@3`와 `@bable/preset-env`를 통해 최적화 할 수 있다. 예를들어, array 의 메소드들과 새로운 `Math`의 제안된 기능들에 대한 폴리필만 필요하다면 다음과 같이 작성할 수 있다.

##### In
```javascript
import "core-js/es/array";
import "core-js/proposal/math-extensions";
```

##### Out (different based on environment)
```javascript
import "core-js/modules/es.array.unscopables.flat";
import "core-js/modules/es.array.unscopables.flat-map";
import "core-js/modules/esnext.math.clamp";
import "core-js/modules/esnext.math.deg-per-rad";
import "core-js/modules/esnext.math.degrees";
import "core-js/modules/esnext.math.fscale";
import "core-js/modules/esnext.math.rad-per-deg";
import "core-js/modules/esnext.math.radians";
import "core-js/modules/esnext.math.scale";
```
[`core-js`](https://github.com/zloirock/core-js) 의 도큐먼트에서 다른 엔트리 포인트에 대한 정보를 확인할 수 있다.

> NOTE: `core-js@2` 를 사용할 때 (혹은 [corejs: 2](https://babeljs.io/docs/en/babel-preset-env#corejs) 옵션을 사용할 때) `@babel/preset-env`는 `@babel/polyfill`의 `import`와 `required` 또한 변환을 한다. 이러한 동작은 다른 `core-js`버전에서 더 이상 `@babel/polyfill`을 사용할 수 없기 때문에 deprecated 될 예정이다.
