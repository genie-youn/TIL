# state of js 2020 리뷰

## 들어가며
연말이면 어김없아 첮아오는 state of js. 슥슥 읽어가며 **개인적인** 소회를 남겨둔다.

### 기능
![optional_chaining](https://stateofx-images.netlify.app/captures/js2020/en-US/optional_chaining.png)
![nullish_coalescing](https://stateofx-images.netlify.app/captures/js2020/en-US/nullish_coalescing.png)

`Optional Chaining`과 `Nullish Coalescing`는 항상 쌍으로 붙어다니는것 같은데 인지도나, 사용량에서 차이가 꽤 난다.

![private_field](https://stateofx-images.netlify.app/captures/js2020/en-US/private_fields.png)

`Private Fields`의 사용량은 자바스크립트에서 `Class`의 위치를 보여주는듯한 느낌이었다.

![proxies](https://stateofx-images.netlify.app/captures/js2020/en-US/proxies.png)

`Proxies`는 vue3에서 reactivity를 `Proxies`를 통해 구현하여 관심있게 보고있었는데, 아직은 사용량이 미비해보인다.

![shadow_dom](https://stateofx-images.netlify.app/captures/js2020/en-US/shadow_dom.png)

`Shadow dom` 의 사용성이 높아 놀랐다. 다들 어디에 그렇게 쓰고 있는거지.

### 기술
![javascript_flavors_experience_marimekko](https://stateofx-images.netlify.app/captures/js2020/en-US/javascript_flavors_experience_marimekko.png)

타입스크립트의 성장세가 눈에 뛴다. 작년에만 해도 이정도는 아니었던것 같은데..

![front_end_frameworks_experience_ranking](https://stateofx-images.netlify.app/captures/js2020/en-US/front_end_frameworks_experience_ranking.png)

![front_end_frameworks_experience_marimekko](https://stateofx-images.netlify.app/captures/js2020/en-US/front_end_frameworks_experience_marimekko.png)

작년에 이어 올해도 `Svelte`가 무섭게 올라오나보다. 이제 React, Vue, Angular 가 아니라 React, Vue, Svelte 3파전으로 보인다.

![datalayer_experience_ranking](https://stateofx-images.netlify.app/captures/js2020/en-US/datalayer_experience_ranking.png)

![datalayer_experience_marimekko](https://stateofx-images.netlify.app/captures/js2020/en-US/datalayer_experience_marimekko.png)

통계에 `Vuex`가 추가된게 눈에 뛴다. `GraphQL`에 비가 `Vuex`와 `Redux`의 인지도를 월등히 앞서는것과, `Redux`의 높은 부정적 경험이 눈에 띈다.

![testing_experience_ranking](https://stateofx-images.netlify.app/captures/js2020/en-US/testing_experience_ranking.png)

![testing_experience_marimekko](https://stateofx-images.netlify.app/captures/js2020/en-US/testing_experience_marimekko.png)

`Testing Library` 를 처음 알게되어 가볍게 살펴보았다. 긍정적 평가가 많은 만큼, 현재 사용중인 `Vue`는 오피셜 라이브러리 `vue-test-utils` 와 비교해볼 필요를 느꼈다.


### 기타 도구
![libraries](https://stateofx-images.netlify.app/captures/js2020/en-US/libraries.png)

유지보수 모드에 들어간 `Moment`가 아직 높은 순위에 랭크되어 있는것이 눈에 들어온다.

![text_editors](https://stateofx-images.netlify.app/captures/js2020/en-US/text_editors.png)

`VS Code` 가 압도적이다. 하지만 나는 인텔리제이를 사랑한다.

### 의견
![missing_from_js](https://stateofx-images.netlify.app/captures/js2020/en-US/missing_from_js.png)

JS에 부족한 점이 무엇이냐는 질문에 `Static Typing`이 압도적인 응답을 차지했다. 많이 공감되기도 하고 그래서 `TypeScript`를 점점 더 찾나 싶기도 하다.

### 시상식
해가 지남에 따라 "다시 사용할 예정"의 진행이 가장 큰 **최다 적용 기술** 에는 `TypeScript` 가 **+14.7%** 의 증가량을 보이며 1위를 차지했다.

사용자의 만족도가 가장 높은 기술인 **최고 만족도** 에는 `Testing Library` 가 97%의 만족도로 1위를 차지했다.

개발자들이 일단 알게 되면 학습에 가장 큰 관심을 갖게 된 기술인 **최고 관심도** 에는 `GraphQL`이 관심도 비율 86%로 1위를 차지했다.
