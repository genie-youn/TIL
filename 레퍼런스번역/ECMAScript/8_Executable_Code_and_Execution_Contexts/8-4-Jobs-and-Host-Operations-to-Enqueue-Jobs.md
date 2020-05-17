# 8.4 Jobs and Host Operations to Enqueue Jobs

Job은 현재 처리중인 다른 ECMAScript 연산이 없을 때, ECMAScript 연산을 시작하는 매개 변수가 없는 Abstract Closure 이다.

Job들은 ECMAScript 호스트환경에 의해 실행되도록 스케줄링 되어 있다.

이 명세는 한 종류의 작업을 스케쥴링 하기 위한 host hook인 [HostEnqueuePromiseJob](https://tc39.es/ecma262/#sec-hostenqueuepromisejob) 에 대해 설명한다.

호스트 환경은 job을 스케쥴링 하는 추가적인 [abstract operations](https://tc39.es/ecma262/#sec-algorithm-conventions-abstract-operations) 을 정의할 수 있다.

이러한 operations 은 Job [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure) 를 파라미터로 받아 향후 어떤 시점에 실행되도록 스케줄링 한다.

반드시 다음 요구사항을 만족하여 구현되어야 한다.



https://tc39.es/ecma262/#job
