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

https://tc39.es/ecma262/#job
