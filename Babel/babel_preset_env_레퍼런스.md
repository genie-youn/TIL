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
