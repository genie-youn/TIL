# 6.2 ECMAScript Specification Type
Specification Type 은 알고리즘 내에서 ECMAScript 언어의 구문과 타입에 대한 의미를 설명하기 위한 메타값에 해당한다.

이 Specification Type 에는 [Reference](https://tc39.es/ecma262/#sec-reference-specification-type), [List](https://tc39.es/ecma262/#sec-list-and-record-specification-type), [Completion](https://tc39.es/ecma262/#sec-completion-record-specification-type), [Property Descriptor](https://tc39.es/ecma262/#sec-property-descriptor-specification-type), [Environment Record](https://tc39.es/ecma262/#sec-environment-records), [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure), [Data Block](https://tc39.es/ecma262/#sec-data-blocks) 이 포함된다.

Specification type의 값은 스펙 정의를 위한 인위로 만들어진 값이며, ECMAScript 구현체의 특정 엔티티에 반드시 대응될 필요는 없다.

Specification type의 값은 ECMAScript 표현식을 평가한 중간결과를 표현하는데 사용될 순 있지만, 객체의 프로퍼티나 ECMAScript 언어의 변수로 저장되어 질 수는 없다.

## 6.2.7 The Abstract Closure Specification Type
*Abstract Closure* specification type 은 값들의 집합과 함께 알고리즘 단계를 참조하는데 사용된다.

Abstract Closure 는 메타값으로 *closure(arg1, arg2)* 와 같이 함수를 사용하둣 호출할 수 있다.


Abstract Closure를 생성하는 알고리즘 단계내에서는 "capture" 라는 동사와 값을 기억할 변수들을 표기하고 이후 수행될 알고리즘을 기술한다.

Abstract Closure가 생성되면 그 시점의 변수의 값을 기억한다.

Abstract Closure를 호출하면, 기억해두었던 값을 참조하여 기술해 둔 알고리즘을 수행한다.


만약 `Abstract Closure`가 `Completion Record`를 반환하면 `Completion Record`의 타입은 정상이거나 예외를 던져야 한다.

`Abstract Closures`은 다음 예제와 같이 알고리즘의 일부로써 인라인으로 생성된다.

1. *addend* 를 41로 할당한다.
2. *closure* 를 *addend* 를 기억하고, 호출됐을 때 다음과 같은 동작을 수행하는 파라미터 (x)를 받는 `Abstract Closure`로 할당한다.
  + a. return *x* + *addend*
3. *val* 에 *closure(1)* 을 할당
4. *val* 이 42인지 확인


https://tc39.es/ecma262/#sec-abstract-closure
