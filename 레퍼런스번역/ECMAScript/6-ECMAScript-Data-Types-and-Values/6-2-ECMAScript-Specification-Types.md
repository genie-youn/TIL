# 6.2 ECMAScript Specification Type
Specification Type 은 알고리즘 내에서 ECMAScript 언어의 구문과 타입에 대한 의미를 설명하기 위한 메타값에 해당한다.

이 Specification Type 에는 [Reference](https://tc39.es/ecma262/#sec-reference-specification-type), [List](https://tc39.es/ecma262/#sec-list-and-record-specification-type), [Completion](https://tc39.es/ecma262/#sec-completion-record-specification-type), [Property Descriptor](https://tc39.es/ecma262/#sec-property-descriptor-specification-type), [Environment Record](https://tc39.es/ecma262/#sec-environment-records), [Abstract Closure](https://tc39.es/ecma262/#sec-abstract-closure), [Data Block](https://tc39.es/ecma262/#sec-data-blocks) 이 포함된다.

Specification type의 값은 스펙 정의를 위한 인위로 만들어진 값이며, ECMAScript 구현체의 특정 엔티티에 반드시 대응될 필요는 없다.

Specification type의 값은 ECMAScript 표현식을 평가한 중간결과를 표현하는데 사용될 순 있지만, 객체의 프로퍼티나 ECMAScript 언어의 변수로 저장되어 질 수는 없다.

## 6.2.7 The Abstract Closure Specification Type

The Abstract Closure specification type is used to refer to algorithm steps together with a collection of values.

*Abstract Closure* specification type 은 값들의 집합과 함께 알고리즘 단계를 참조하는데 사용된다.

Abstract Closure 는 메타값으로 *closure(arg1, arg2)* 와 같이 함수를 사용하둣 호출할 수 있다.

In algorithm steps that create an Abstract Closure, values are captured with the verb "capture" followed by a list of aliases. When an Abstract Closure is created, it captures the value that is associated with each alias at that time. In steps that specify the algorithm to be performed when an Abstract Closure is called, each captured value is referred to by the alias that was used to capture the value.

만약 `Abstract Closure`가 `Completion Record`를 반환하면 `Completion Record`의 타입은 정상이거나 예외를 던져야 한다.

`Abstract Closures`은 다음 예제와 같이 알고리즘의 일부로써 인라인으로 생성된다.

1. *addend* 를 41로 할당한다.
2. *closure* 를 *addend* 를 기억하고, 호출됐을 때 다음과 같은 동작을 수행하는 파라미터 (x)를 받는 `Abstract Closure`로 할당한다.
  + a. return *x* + *addend*
3. *val* 에 *closure(1)* 을 할당
4. *val* 이 42인지 확인


https://tc39.es/ecma262/#sec-abstract-closure
