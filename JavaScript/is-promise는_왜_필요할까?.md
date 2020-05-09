# is-promise는 왜 필요할까?

## 들어가며
얼마전, 한 `is-promise`라는 라이브러리를 의존하는 수많은 라이브러리, 프로젝트들이 빌드가 깨지는 일이 있었다.

`Promise`인지 체크하려면 `typeof obj === "Promise"`를 쓰면 되는거 아닌가? 왜 이토록 많은 프로젝트들이 이 라이브러리를 사용하고 있을까? 하는 생각이 들었고 `is-promise`를 좀 더 자세히 살펴보기로 했다.

## is-promise
결론부터 얘기하면 `is-promise`는 주어진 객체가 `promise` 타입인지가 아니라 `promise`로써 다루어 질 수 있는지를 체크한다.

`Promise`를 다루는 많은 스펙들이 ES6에 표준으로 포함된 `native Promise` 이전에 존재했던 여러 `Promise` 구현체들과의 호환성을 유지해야 했고,

이를 지원하기 위해 주어진 인자들이 `native Promise` 인지를 체크하는 `typeof obj === "promise"` 가 아니라 `Promise` 로 다루어 질 수 있는지 체크하는 `is-promise(obj)`가 필요했던 것이다.

구체적인 히스토리는 https://www.stefanjudis.com/today-i-learned/promise-resolution-with-objects-including-a-then-property/ 이 글을 읽어 보면 될듯
