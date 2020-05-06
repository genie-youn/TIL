`is-promise` 는 또 따로 정리해야할것 같은데?
단순히 `typeof a === "promise"` 가 아닌 이걸 써야 하는 이유?

`is-promise`는 이 객체가 `promise` 타입인지 아닌지가 아니라, `promise`로써 다루어 질 수 있는지를 체크한다.
https://github.com/then/is-promise/issues/6
https://promisesaplus.com/#the-promise-resolution-procedure
https://www.stefanjudis.com/today-i-learned/promise-resolution-with-objects-including-a-then-property/
https://github.com/then/is-promise/issues/38#issuecomment-619929999
이걸 읽어봐야할 듯

- The reason this library treats your example as a Promise is that the Promises spec requires any object or function with a .then method to be treated as a Promise.
- The reason the spec requires that is because it allows interoperability between the many Promise implementations that existed before native promises. Without this, promises would be virtually unusable for years while everyone migrated.
- This library exists, even though it's only 3 lines because it means you don't have to think about exactly these problems. Lots of people know little enough about Promises that they wouldn't be able to implement this correctly (see all the pull requests & comments offering to "simplify" the logic). That's fine, they shouldn't need to worry about that, they can just import is-promise.
