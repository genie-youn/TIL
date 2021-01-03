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
