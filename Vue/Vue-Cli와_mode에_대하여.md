# Vue-Cli와 mode에 대하여

### 들어가며
vue-cli로 프로젝트를 구성하고 실행시 다음과 같은 식으로 `--mode` 옵션에 파리미터를 주어 페이즈를 구분하고 있었다.

package.json
```json
"script": {
  "build-stg": "vue-cli build --mode stage",
  "build-prod": "vue-cli build --mode production"
}
```

문제는 코드중에 `NODE_ENV`로 환경별로 다르게 동장해야 할 로직을 분기처리 했는데, 이게 제대로 동작하지 않는 일이 발생했따. 당연히 이 `--mode` 옵션으로 주어진 파라미터가 `NODE_ENV`에도 반영될 것이라 생각해고 이는 한참 잘못된 생각이었다.

### `--mode` 옵션
우선 vue-cli의 `--mode`옵션에 대해서 잠깐 살펴보자면 다음과 같다.
**Mode** 는 vue-cli 프로젝트에서 중요한 개념 중 하나다. 기본적으로 다음 세가지 모드가 존재한다.

- `development` 는 `vue-cli-service serve` 로 실행시 자동으로 설정된다.
- `test` 는`vue-cli-service test:unit`로 실생시 자동으로 설정된다.
- `production` 는 `vue-cli-service build` 또는 `vue-cli-service test:e2e` 로 실행시 자동으로 설정된다.

> 레퍼런스는 [여기](https://cli.vuejs.org/guide/mode-and-env.html#modes-and-environment-variables)를 참고한다.
