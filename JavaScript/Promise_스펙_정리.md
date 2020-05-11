# Promise 스펙 정리

- Promise 는 지연된 (그리고 아마 비동기로 처리될) 연산의 결과에 대한 플레이스홀더로 사용될 수 있는 객체이다.
- 어떤 Promise 객체든 다음 세가지의 서로 베타적인 상태중 하나를 갖는다.
  + *fulfilled*
  + *rejected*
  + *pending*

https://tc39.es/ecma262/#sec-promise-objects
