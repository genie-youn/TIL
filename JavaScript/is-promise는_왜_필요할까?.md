# is-promise는 왜 필요할까?

## 들어가며
얼마전, 한 `is-promise`라는 라이브러리를 의존하는 수많은 라이브러리, 프로젝트들이 빌드가 깨지는 일이 있었다. `promise`인지 체크하려면 `typeof a === "promise"`를 쓰면 되는거 아닌가? 왜 이토록 많은 프로젝트들이 이 라이브러리를 사용하고 있을까? 하는 생각이 들었고 `is-promise`를 좀 더 자세히 살펴보기로 했다.

## is-promise
결론부터 얘기하면 `is-promise`는 주어진 객체가 `promise` 타입인지가 아니라 `promise`로써 다루어 질 수 있는지를 체크한다.

https://github.com/then/is-promise/issues/6
https://promisesaplus.com/#the-promise-resolution-procedure
https://www.stefanjudis.com/today-i-learned/promise-resolution-with-objects-including-a-then-property/
https://github.com/then/is-promise/issues/38#issuecomment-619929999
이걸 읽어봐야할 듯

- The reason this library treats your example as a Promise is that the Promises spec requires any object or function with a .then method to be treated as a Promise.
- The reason the spec requires that is because it allows interoperability between the many Promise implementations that existed before native promises. Without this, promises would be virtually unusable for years while everyone migrated.
- This library exists, even though it's only 3 lines because it means you don't have to think about exactly these problems. Lots of people know little enough about Promises that they wouldn't be able to implement this correctly (see all the pull requests & comments offering to "simplify" the logic). That's fine, they shouldn't need to worry about that, they can just import is-promise.
