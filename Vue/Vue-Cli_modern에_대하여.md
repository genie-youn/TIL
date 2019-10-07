# vue-cli --modern 명령어는 뭐지

### 들어가며
vue-cli를 통해 진행하고 있는 프로젝트에서 프로덕션 빌드환경을 구성하다 vue-cli를 사용한지 무려 일년만에 `--modern` 옵션이 존재하는걸 보았고, 이에 대해 정리해두려 한다.

### Modern Mode
바벨을 사용하면 ES2015+의 새로운 기능들을 오래된 브라우저에도 사용할 수 있게 된지만, 이를 위해 코드를 트랜스파일하고 폴리필을 넣어주어야 한다. 이렇게 변환된 번들은 기존 코드보다 장황하고 파싱과 실행에 있어 더 많은 시간을 필요로 한다.

오늘날 대부분의 브라우저가 ES2015+에 대한 상당한 지원을 하고 있다는 것을 감안할 때 단지 오래된 브라우저를 지원해야 한다는 이유만으로 더 무겁고 느린 코드를 브라우저에 전송하는 것은 큰 낭비이다.

이러한 문제를 해결하기 위해 vue-cli는 `Modern Mode` 라는 기능을 제공한다.
아래 명령은 `Modern Mode`로 production을 위한 빌드를 수행한다.

```shell
$ vue-cli-service build --modern
```

vue-cli는 `Modern Mode`에서 ES Modules를 지원하는 최신 브라우저를 위한 번들 한 개, 이를 지원하지 않는 구형 브라우저를 위한 번들 한 개, 총 두개의 번들 결과물을 만들어낸다.

이 옵션의 멋진 부분은 배포시 특별히 신경써야할 부분이 없다는 것이다. 생성된 HTML 파일은 [Phillip Walton의 멋진 포스트](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/)에서 소개된 기술을 통해 자동으로 번들을 선택하게 된다.

최신 브라우저에서는 모던한 번들이 `<script type="module">`를 통해서 로드되며, `<link rel="modulepreload">`를 통해 필요한 다른 번들을 미리 로드할 수 있다.

구형 브라우저에서는 `<script nomodule>`를 통해 구형 브라우저를 위한 번들이 로드되고, 이는 ES 모듈을 지원하는 신형 브라우저에서는 무시된다.

헬로월드 앱에서 모던 번들은 16%나 번들 사이즈가 줄어든다. 프로덕션 환경에서 일반적으로 모던 번들은 빠르게 평가되고, 빠르게 파싱되어 애플리케이션의 로딩 성능 향상시킬것이다.

> `<script type="module>"`은 항상 CORS가 가능한 상황에서 로드되어야 된다. 이는 곧 서버에서 `Access-Control-Allow-Origin: *` 와 같은 유효한 CORS 헤더를 함께 내려주어야 한다는 이야기다. 만약 특정한 credentials을 통해 스크립트를 내려받길 원한다면 [crossorigin](https://cli.vuejs.org/config/#crossorigin) 옵션을 use-credentials로 변경해야 한다.

> 사파리 10에서 두개의 번들을 모두 로드하는걸 막으려면 설정을 더 해주어야 하는데, 요즘 세상에 사파리 10은 찾기 힘드므로 넘어간다.

> 때로는 웹팩에서 레거시 빌드나, 모던 빌드에 따라 다르게 설정할 필요가 있다. 이럴 땐 다음 플래그를 사용하면 된다.
- `VUE_CLI_MODERN_MODE`: 빌드가 `--modern` 으로 설정되었는지
- `VUE_CLI_MODERN_BUILD`: 모던 빌드
이 프로퍼티는 `chainWebpack()`나 `configureWebpack()`가 평가되었을 때에만 접근할 수 있음에 유의한다.
