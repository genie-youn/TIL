# Abstract Closure

## ECMAScript Specification Type
Specification Type 은 알고리즘 내에서 ECMAScript 언어의 구문과 타입에 대한 의미를 설명하기 위한 메타값에 해당한다.

이 Specification Type 에는 [Reference](https://tc39.es/ecma262/#sec-reference-specification-type), [List](https://tc39.es/ecma262/#sec-list-and-record-specification-type), [Completion](https://tc39.es/ecma262/#sec-completion-record-specification-type), [Property Descriptor](https://tc39.es/ecma262/#sec-property-descriptor-specification-type), [Environment Record](https://tc39.es/ecma262/#sec-environment-records), [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure), [Data Block](https://tc39.es/ecma262/#sec-data-blocks) 이 포함된다.

Specification type의 값은 스펙 정의를 위한 인위로 만들어진 값이며, ECMAScript 구현체의 특정 엔티티에 반드시 대응될 필요는 없다.

Specification type의 값은 ECMAScript 표현식을 평가한 중간결과를 표현하는데 사용될 순 있지만, 객체의 프로퍼티나 ECMAScript 언어의 변수로 저장되어 질 수는 없다.

만약 `Abstract Closure`가 `Completion Record`를 반환하면 `Completion Record`의 타입은 정상이거나 예외를 던져야 한다.

`Abstract Closures`은 다음 예제와 같이 알고리즘의 일부로써 인라인으로 생성된다.

1. Let addend be 41.
2. Let closure be a new Abstract Closure with parameters (x) that captures addend and performs the following steps when called:
  a. Return x + addend.
3. Let val be closure(1).
4. Assert: val is 42.

1. *addend* 를 41로 할당한다.
2. *closure* 를 *addend* 를 기억하고, 호출됐을 때 다음과 같은 동작을 수행하는 파라미터 (x)를 받는 `Abstract Closure`로 할당한다.
  + a. 

https://tc39.es/ecma262/#sec-abstract-closure
