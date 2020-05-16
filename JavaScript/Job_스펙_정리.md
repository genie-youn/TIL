# Job 스펙 정리

Job은 현재 처리중인 다른 ECMAScript 연산이 없을 때, ECMAScript 연산을 시작하는 매개 변수가 없는 Abstract Closure 이다.

Job들은 ECMAScript 호스트환경에 의해 실행되도록 스케줄링 되어 있다.

This specification describes the host hook [HostEnqueuePromiseJob](https://tc39.es/ecma262/#sec-hostenqueuepromisejob) to schedule one kind of job; host environments may define additional [abstract operations](https://tc39.es/ecma262/#sec-algorithm-conventions-abstract-operations) which schedule jobs.

Such operations accept a Job Abstract Closure as the parameter and schedule it to be performed at some future time.

Their implementations must conform to the following requirements:

https://tc39.es/ecma262/#job
