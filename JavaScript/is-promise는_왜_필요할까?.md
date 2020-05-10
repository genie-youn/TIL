# is-promise는 왜 필요할까?

## 들어가며
얼마전, 한 `is-promise`라는 라이브러리를 의존하는 수많은 라이브러리, 프로젝트들이 빌드가 깨지는 일이 있었다.

`Promise`인지 체크하려면 `typeof obj === "Promise"`를 쓰면 되는거 아닌가? 왜 이토록 많은 프로젝트들이 이 라이브러리를 사용하고 있을까? 하는 생각이 들었고 `is-promise`를 좀 더 자세히 살펴보기로 했다.

## is-promise
결론부터 얘기하면 `is-promise`는 주어진 객체가 `Promise` 타입인지가 아니라 `Promise`로써 다루어 질 수 있는지를 체크한다.

`Promise`를 다루는 많은 스펙들이 ES6에 표준으로 포함된 `native Promise` 이전에 존재했던 여러 `Promise` 구현체들과의 호환성을 유지해야 했고,

이를 지원하기 위해 주어진 인자들이 `native Promise` 인지를 체크하는 `typeof obj === "promise"` 가 아니라 `Promise` 로 다루어 질 수 있는지 체크하는 `is-promise(obj)`가 필요했던 것이다.

`Promise`를 다루는 대표적인 스펙인 `Promise.resolution`을 보자.

`Promise.resolve()`를 호출하면 다음과 같은 처리가 이루어진다.

1. Let F be the active function object.
2. Assert: F has a [[Promise]] internal slot whose value is an Object.
3. Let promise be F.[[Promise]].
4. Let alreadyResolved be F.[[AlreadyResolved]].
5. If alreadyResolved.[[Value]] is true, return undefined.
6. Set alreadyResolved.[[Value]] to true.
7. If SameValue(resolution, promise) is true, then
  a. Let selfResolutionError be a newly created TypeError object.
  b. Return RejectPromise(promise, selfResolutionError).
8. If Type(resolution) is not Object, then
  a. Return FulfillPromise(promise, resolution).
9. Let then be Get(resolution, "then").
10. If then is an abrupt completion, then
  a. Return RejectPromise(promise, then.[[Value]]).
11. Let thenAction be then.[[Value]].
12. If IsCallable(thenAction) is false, then
  a. Return FulfillPromise(promise, resolution).
13. Let job be NewPromiseResolveThenableJob(promise, resolution, thenAction).
14. Perform HostEnqueuePromiseJob(job.[[Job]], job.[[Realm]]).
15. Return undefined.


> 위 내용과 관련된 내용은 [이 아티클](https://www.stefanjudis.com/today-i-learned/promise-resolution-with-objects-including-a-then-property/)에 잘 설명되어 있다.

> 'Promise' 로써 다루어 질 수 있는지에 대한 표준 스펙은 다음 [링크](https://tc39.es/ecma262/#sec-promise-resolve-functions)를 참고한다.
