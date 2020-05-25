# 25.6 Promise Objects
Promise 는 지연된 (그리고 아마 비동기로 처리될) 연산의 결과에 대한 플레이스홀더로 사용될 수 있는 객체이다.

어떤 Promise 객체든 다음 세가지의 서로 베타적인 상태중 하나를 갖는다. *fulfilled*, *rejected*, *pending*

- 프로미스 `p` 는 `p.then(f, r)` 이 즉시 함수 `f` 를 호출하는 작업을 큐에 밀어 넣을 경우 *fulfilled* 상태가 된다.
- 프로미스 `p` 는 `p.then(f, r)` 이 즉시 함수 `r` 를 호출하는 작업을 큐에 밀어 넣을 경우 *rejected* 상태가 된다.
- 프로미스가 *fulfilled* 도, *rejected* 도 아니라면 *pending* 이다.

프로미스가 *pending* 상태가 아니라면, 즉 *fulfilled* 거나 *rejected* 중 하나라면, *settled* 되었다고 얘기할 수 있다.

프로미스가 *settled* 거나, 다른 프로미스의 상태에 맞추기위해  "locked in" 되어 있다면 *resolved* 되었다고 한다.

이미 resolved 된 프로미스를 `resolve` 하거나 `reject` 하는건 아무런 효과도 없다.
만약 resolved 되지 않은 프로미스를 *unresolved* 되었다고 하는데, 이 *unresolved* 된 프로미스는 항상 pending 상태에 놓이게 된다.
resolved 된 프로미스는 pending 일수도, fulfilled 일수도, rejected 일 수도 있다.

## 25.6.1 Promise Abstract Operations

### 25.6.1.1 PromiseCapability Records
PromiseCapability 는 프로미스 객체를 resolve 하거나 reject 하는 함수와 함께 캡슐화 하는데 사용되는 [Record](https://tc39.es/ecma262/#sec-list-and-record-specification-type) 값이다.
PromiseCapability Records 는  NewPromiseCapability 추상 연산에 의해 생성된다.
PromiseCapability Records 는 다음 표에 있는 필드들을 가지고 있다.

| Field       | Name              |	Value	Meaning
| [[Promise]] |	An object	        | An object that is usable as a promise.
| [[Resolve]] |	A function object |	The function that is used to resolve the given promise object.
| [[Reject]]	| A function object	| The function that is used to reject the given promise object.

#### 25.6.1.1.1 IfAbruptRejectPromise ( value, capability )
IfAbruptRejectPromise is a shorthand for a sequence of algorithm steps that use a PromiseCapability Record. An algorithm step of the form:

IfAbruptRejectPromise는 단축이다. PromiseCapability Record를 사용하는 일련의 알고리즘 단계를 나타내기 위한 단축된 표현이다. 알고리즘 단계는 다음과 같은 형태를 갖는다.

1. IfAbruptRejectPromise(value, capability).

이는 다음과 동일한 의미를 갖는다.

1. If value is an [abrupt completion](https://tc39.es/ecma262/#sec-completion-record-specification-type), then
  a. Perform ? [Call](https://tc39.es/ecma262/#sec-call)(*capability*.[[Reject]], **undefined**, « *value*.[[Value]] »).
  b. Return *capability*.[[Promise]].
2. Else if value is a Completion Record, set value to value.[[Value]].

### 25.6.1.2 PromiseReaction Records
PromiseReaction은 프로미스가 resoved 되었거나 rejected 되었을 때 어떻게 반응해야 하는지에 대한 정보를 저장하기 위해 사용되는 [Record](https://tc39.es/ecma262/#sec-list-and-record-specification-type) 값이다.

PromiseReaction records는 [PerformPromiseThen](https://tc39.es/ecma262/#sec-performpromisethen) 추상 연상에 의해 생성되고, [NewPromiseReactionJob](https://tc39.es/ecma262/#sec-newpromisereactionjob) 로 부터 반환되는 [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure)에 의해 사용된다.

https://tc39.es/ecma262/#sec-promise-objects
