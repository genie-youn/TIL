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

문제는 코드중에 `NODE_ENV`로 환경별로 다르게 동장해야 할 로직을 분기처리 했는데, 이게 제대로 동작하지 않는 일이 발생했다. 당연히 이 `--mode` 옵션으로 주어진 파라미터가 `NODE_ENV`에도 반영될 것이라 생각해고 이는 한참 잘못된 생각이었다.

### `--mode` 옵션
우선 vue-cli의 `--mode`옵션에 대해서 잠깐 살펴보자면 다음과 같다.
**Mode** 는 vue-cli 프로젝트에서 중요한 개념 중 하나다. 기본적으로 다음 세가지 모드가 존재한다.

- `development` 는 `vue-cli-service serve` 로 실행시 자동으로 설정된다.
- `test` 는`vue-cli-service test:unit`로 실생시 자동으로 설정된다.
- `production` 는 `vue-cli-service build` 또는 `vue-cli-service test:e2e` 로 실행시 자동으로 설정된다.

> 레퍼런스는 [여기](https://cli.vuejs.org/guide/mode-and-env.html#modes-and-environment-variables)를 참고한다.

물론 이외의 다른 옵션도 줄 수 있지만, 중요한 점은 이 세가지 외에 다른 값을 주었을 때는 이 값이 `NODE_ENV`에 설정되지는 않는다는 것이다. 만약 `--mode`의 파라미터로 `stage`를 주었을 때 `NODE_ENV`가 `stage` 설정되게 하기 위해서는 `vue-cli`가 이 `mode` [값에 해당하는 `.env`파일](https://cli.vuejs.org/guide/mode-and-env.html#environment-variables) 에 `NODE_ENV`를 `stage`로 선언해주어야 한다.

### 빌드타임에 `process` 에서 가져오기
다만 위와같이 `mode`의 값에 따른 설정파일을 자동으로 불러오기 위해서는 `.env` 파일들이 프로젝트 루트에 존재해야만 한다. 이 구조가 마음에 들지 않는다면 위 `mode`의 파라미터 값이 `process`에 주입되는것을 사용하여 쉽게 변경할 수 있다.

vue.config.js
```javascript
const environment = new Dotenv({
  path: `./environment/.env.${process.VUE_CLI_SERVICE.mode}`,
});

module.exports = {
  configureWebpack: {
    plugins: [environment],
  },
}
```
