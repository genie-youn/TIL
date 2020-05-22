# 25.6 Promise Objects

Promise 는 지연된 (그리고 아마 비동기로 처리될) 연산의 결과에 대한 플레이스홀더로 사용될 수 있는 객체이다.

어떤 Promise 객체든 다음 세가지의 서로 베타적인 상태중 하나를 갖는다. *fulfilled*, *rejected*, *pending*

- 프로미스 `p` 는 `p.then(f, r)` 이 즉시 함수 `f` 를 호출하는 작업을 큐에 밀어 넣을 경우 *fulfilled* 상태가 된다.
- 프로미스 `p` 는 `p.then(f, r)` 이 즉시 함수 `r` 를 호출하는 작업을 큐에 밀어 넣을 경우 *rejected* 상태가 된다.
- 프로미스가 *fulfilled* 도, *rejected* 도 아니라면 *pending* 이다.

프로미스가 *pending* 상태가 아니라면, 즉 *fulfilled* 거나 *rejected* 중 하나라면, *settled* 되었다고 얘기할 수 있다.

프로미스가 *settled* 거나, 다른 프로미스의 상태에 맞추기위해  "locked in" 되어 있다면 *resolved* 되었다고 한다.

Attempting to resolve or reject a resolved promise has no effect.

A promise is unresolved if it is not resolved.

An unresolved promise is always in the pending state.

A resolved promise may be pending, fulfilled or rejected.



https://tc39.es/ecma262/#sec-promise-objects
