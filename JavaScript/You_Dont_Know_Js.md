# You Don't Know Js



##1 타입 



EcmaSacript 의 언어 타입은 undefined, null, number, string, object, boolean, symbol 이 존재한다. 



### 1.1 타입 그 실체를 이해하자 



coercion -> 타입 강제 변환 



### 1.2 내장 타입 



null 은 type이 object 이다. 

falsy 하고 type이 object 인지 체크해야 null 체크 가능하다. 

typeof 의 결과물로 나오는 function 은 object 의 하위 타입이다. 

함수 = 호출 가능한 [내부적인 call 프로퍼티로] object 

배열은 몇가지 특성 (length 가 자동으로 생성된다든지) 하는 object 의 특이한 하위타입 



### 1.3 값은 타입을 가진다 



js 의 변수에는 타입이 없다. 변수가 갖는 값에만 타입이 존재하며 typeof 는 이 변수에 들어있는 값의 타입은? 으로 해석해야한다. 



#### 1.3.1 값이 없는 vs 선언되지 않은 



값이 없는 변수의 값은 undefined 이다. 

undefined 와 is not declare 는 명확히 다르지만 typeof 를 때리면 둘다 undefined 이다. 

직접 사용하면 ReffereceError 를 뱉는다. 

이는 typeof 의 safe-guard 이다. 전역변수 체크를 위한.. 



#### 1.3.2 선언되지 않은 변수 



존재하지 안흔 기능을 추가하기위해 폴리필을 정의하려면 

atob 선언문에서 var를 빼라 



그 이유는 코드실행 (if 문)을 건너뛰어도 선언 자체가 최상위 스코프로 호이스팅 되면서 특수한 타입의 전역변수 (호스트 객체 window) 가 중복선언되어 예외를 뱉는다. 



typeof 안쓰려면 window.~ 로 해도 되는데 (선언되지 않은 객체의 어떤 프로퍼티로 접근할때는 선언되지 않아도 예외 안뱉음) 권장하진 않는다. 



### 1.4 1장 정리 



js 에는 7가지 타입 (null, undefined, number, string, boolean, object, symbol) 이 존재 

변수에는 타입이 없고 변수가 갖는 값에만 타입이 있다. 

js 에서 `undefined` 와 `undeclare` 는 명확히 다르지만 typeof 의 결과는 둘다 undefined 이다. 