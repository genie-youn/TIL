# 8.4 Jobs and Host Operations to Enqueue Jobs

Job은 현재 처리중인 다른 ECMAScript 연산이 없을 때, ECMAScript 연산을 시작하는 매개 변수가 없는 Abstract Closure 이다.

Job들은 ECMAScript 호스트환경에 의해 실행되도록 스케줄링 되어 있다.

이 명세는 한 종류의 작업을 스케쥴링 하기 위한 host hook인 [HostEnqueuePromiseJob](https://tc39.es/ecma262/#sec-hostenqueuepromisejob) 에 대해 설명한다.

호스트 환경은 job을 스케쥴링 하는 추가적인 [abstract operations](https://tc39.es/ecma262/#sec-algorithm-conventions-abstract-operations) 을 정의할 수 있다.

이러한 operations 은 Job [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure) 를 파라미터로 받아 향후 어떤 시점에 실행되도록 스케줄링 한다.

반드시 다음 요구사항을 만족하여 구현되어야 한다.

At some future point in time, when there is no running execution context and the execution context stack is empty, the implementation must

- 미래의 어떤 시점에, [실행중인 execution context](https://tc39.es/ecma262/#running-execution-context) 가 없고 [execution context 스택](https://tc39.es/ecma262/#execution-context-stack) 이 비어있을 때 구현체는 반드시,
  + 1. [execution context](https://tc39.es/ecma262/#sec-execution-contexts) 을 [execution context 스택](https://tc39.es/ecma262/#execution-context-stack) 에 밀어 넣어야 한다.
  + 2. 구현체에서 정의한 준비 단계(preparation steps)를 수행한다.
  + 3. [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure)를 호출한다.
  + 4. 구현체에서 정의한 정리 단계(cleanup)를 수행한다.
  = 5. [execution context 스택](https://tc39.es/ecma262/#execution-context-stack)에 이전에 밀어넣었던 [execution context](https://tc39.es/ecma262/#sec-execution-contexts)을 꺼낸다.
- 특정 시점엔 오로지 하나의 [Job](https://tc39.es/ecma262/#job) 만이 평가될 수 있다.
- Once evaluation of a Job starts, it must run to completion before evaluation of any other Job starts.
- 한번 [Job](https://tc39.es/ecma262/#job)이 평가되기 시작하면 다른 [Job](https://tc39.es/ecma262/#job)에 대한 평가가 시작되기전 반드시 완료되어야 한다.
- [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure)은 반드시 에러에 대한 처리를 구현하고 완료를 반환해야 한다.

> NOTE: 호스팅환경은 스케줄링에 관련하여 모든 Job들을 일괄적으로 다룰 필요는 없다. 예를들어 웹 브라우저와 Node.js는 Promise를 다루는 Job을 다른 작업보다 높은 우선순위로 처리한다. 후에는 우선순위가 높지 않은 작업을 추가하는 기능이 추가될 수도 있다.

## 8.4.1 HostEnqueuePromiseJob (*job*, *realm*)
HostEnqueuePromiseJob 은 호스팅 환경에서 정의하는 추상 연산으로, 미래의 특정 시점에 [Job](https://tc39.es/ecma262/#job) [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure) *job* 을 수행하도록 스케줄링 한다.

이 알고리즘과 함께 사용되는 Abstract Closures 는 Promise를 다루는 일과 연관되어 있거나, Promise를 다루는 연산과 동일한 우선순위로 스케줄링 될 작업을 나타낸다.

*realm* 파라미터는 특별히 표준으로 정의된 요구사항 없이 호스트로 전달되게 되며, 이는 `null` 이나 [`Realm`](https://tc39.es/ecma262/#realm) 가 될 수 있다.

https://tc39.es/ecma262/#job
