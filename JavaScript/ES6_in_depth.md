# ES6 in depth



## 1. 들어가며

ECMAScript 란 JavaScript의 표준. ES5 는 2009년에 ES6 는 2015년에 제정되었다. 현재는 7까지 나온듯 



## 2. Iterator 와 for-of 



### 2.1 for-of 

기존에 순회하는 내장 함수는 for-in 과 es5에서 추가된 foreEach() 



우선 forEach()는 `return` `break` 등으로 중간에 탈출 불가능 

for-in은 몇가지 단점이 있는데. 

- index로 할당되는 값이 문자열 
- 확장된 프로퍼티를 가지고 있다면 그마저도 돌고 
- 해당 객체의 프로토타입의 속성까지 돌고 
- 결정적으로 환경에 따라 순회하는 순서가 무작위다..?



이러한 기존 순회 내장 함수들의 단점을 해결하여 배열의 순회만을 위한 내장 함수가 ES6 추가 되었는데 그게 바로 `for-of` 사용법은 for-in 과 같지만 다음과 같은 특징이 있다. 



forEach 의 단점 해결 -> `break` , `continue`, `return` 사용 가능 

for-in 의 단점해결 -> 배열의 값(data) 만을 순회 



정리하자면 for-in 은 객체의 순회를 위해 for-of 는 배열의 순회를 위해 존재한다. 유사 배열도 가능하다. 



### 2.2 iterator 



for-of 구문은 다른 언어와 비슷하게 콜렉션에 내장된 iterator 객체를 사용한다. 자바에서 `next()`, `hasNext()` 를 지원하는것과는 다소 대조적으로 `next()` 에서 전부 해결 이 함수가  {done : false, value : 4} 이런식으로 둘다 나타낸다. 



경우에 따라 `return()`, `throw(exc)` 도 구현할 수 있는데 `return()` 은 `break`, `continue`, `return` 등이 호출되었을 때 할당받은 리소스를 해제하는데 사용되고. `throw` 는 for-of 구문에서는 사용되지 않는다. 



이 iterator 는 콜렉션의 프로퍼티에 [Symbol.iterator] 로 이터레이터 객체를 반환해주는 함수를 구현해주면 되는데 Symbol 은 es6 에서 추가된 하위호환성을 지키기 위해 내부적으로 사용하는 프로퍼티의 유일값,  



이 iterator 는 es6 에 추가된 다른 많은 문법에서도 사용된다. 



예제코드  

```javascript 
var zeroesForeverIterator = {  
	[Symbol.iterator]: function () {  
		return this;  
	},
	next: function () {  
		return {done: false, value: 0};  
	} 
};
```



```javascript
for (VAR of ITERABLE) {  
	STATEMENTS 
}
```



```javascript 
var $iterator = ITERABLE[Symbol.iterator]();  
var $result = $iterator.next();  
while (!$result.done) {  
	VAR = $result.value; 
	STATEMENTS 
	$result = $iterator.next(); 
}
```



## 3. 제너레이터 (Generator) 



### 3.1 제너레이터란? 



중간에 멈출 수 있는 함수. function* 로 선언하며 yield 키워드를 만나면 해당값을 반환하고 다음 요청이 있을때까지 멈춘다. 



### 3.2 제너레이터가 하는일 



제너레이터를 호출하면 멈춰진 제너레이터 객체를 호출하고 

`next()` 를 호출하면 이터레이터와 마찬가지로 `{done:_, value:_}` 객체를 반환, value 에는 yield 키워드로 반환한 값이 들어있다. 



자체적으로 이터레이터라 [Symbol.iterator] 가 this를 반환하도록, next()가 yield 가 반환하는 값을 반환하도록 되어있다. 



기술적으로는 yield 를 호출하면 스택 프레임이 스택에서 제거되지만 이 스택프레임을 제너레이터가 기억하고 있어서 다음번에 호출되면 다시 이어서 실행 -> 스레드가 아니다 



### 3.3 제너레이터는 이터레이터다 



객체를 이터러블하게 만들기 가능, 제너레이터 함수를 구현하고 이걸 [Symbol.iterator] 에 등록 

배열 생성 함수를 간단하게 만듬 

거대한 데이터 처리 -> 무한 시퀀스 생성 가능 

복잡한 루프 구문 리펙터링 -> 값을 생성하는 부분을 제너레이터로 분리 

이터레이터를 다루는 도구 filter 나 map 을 요긴하게 구현할 수 있다. 



=> ES6 에서 데이터나 루프를 처리하는 새로운 표준인 이터레이터를 쉽게 만들수 있다. 



하지만 이것보단 비동기처리가 꽃이라는데. 콜백헬을 없애준다는데..



## 4. 템플릿 문자열 



 ` (backtick) 을 사용 

`hello ${user.name} last login ${user.lastlogin}` 이런거 가능 

태그된 템플릿으로 유용한 기능이 있는 태그 생성 가능 



## 5, 레스트 파라미터와 디폴트 파라미터 



### 5.1 레스트 파라미터 



RestParameter 는 인자의 갯수가 가변인 함수 (variadic function) 을 구현하는데 사용 

선언된 인자를 기존 규칙대로 넣고 나머지 인자는 배열에 담아서 넘긴다. 



```
functon containsAll(hystack, …neddles)
```



배열에 담길 때 기존 인자는 없으면 undefined 지만 얘는 빈배열 떨군다 굿 



### 5.2 디폴트 파라미터 



함수에 인자에 기본값을 줄때 



```
function animalSentence(animal = “tiger”)
```



디폴트 파라미터는 호출될 때 계산되며 왼쪽에서 순서대로 계산, 즉 맘만 먹으면 앞의 파라미터를 사용해서 삼항연산도 가능하다.. 



### 5.3 Arguments를 쓰지 말것 



쓰지마라, 가독성에도 않좋고 Javascript vm 성능 최적화에도 안좋다 



## 6. Destructuring 할당 



디스트럭쳐링 할당은 객체나 배열을 펼쳐서 변수에 할당하는것 



### 6.1 배열 디스트럭쳐링 



var [first, second, thrid] = someArray; 

var [, , thrid] = someArray; 

var [head, …tail] = someArray; 



이터레이터에도 사용가능 

```javascript
function* fibs() { 
	var a = 0;
	var b = 0;
	while(true) {
		yield a;
		[a, b] = [ b, a+b ];
	}
} 

var [first, second, thrid] = fibs(); 
```



### 6.2 객체 디스트럭쳐링 



```javascript
var { foo, bar } = { foo : “lorem”, bar : “ipsum”}; 

({foo, bar}) = { foo : “lorem”, bar : “ipsum”}; 
```



디폴트값 설정 가능 



### 유용팁 



함수 인자를 정의할 때 API 를 만들때 인자를 순서대로 받지말고 객체를 받아서 디스트럭쳐링 



`function api({url, target, method}); `



설정객체 (디폴트 사용) 



이터레이션 프로토콜 

`for (var [key,value] of map) `

`for (var [key] of map) `

`for (var [,value] of map) `



여러개의 값을 리턴할때 



## 7. 화살표 함수 



js 에선 함수도 인자로 전달 가능한 일급시민, 그래서 필요할때 익명함수를 써서 전달하곤 하는데 함수를 간단하게 표현하기 위한게 람다. js는 => 로 나타내어 화살표 함수라고 한다. 



Identifier => Expression 으로 나타내고, `return`, `function` 중괄호같은게 필요 없어진다. 

여러개 인자 (레스트 파라미터, 디폴트 파라미터, 디스트럭쳐링등) 은 괄호로 묶어주면 된다. 



화살표 바로 다음으로 오는 { 는 블럭으로 인식하기 때문에 객체 리터럴을 반환할땐 괄호로 싸줄것 a => ({a:”b”}) 



### this? 



화살표 함수는 고유한 this 를 갖지 않으며 자신을 감싸고 있는 scope 로부터 전달받는다.  

object.method() 로 호출되는 함수 (객체의 프로퍼티인 함수) 는 의미있는 this를 자동으로 전달받으므로 쓰지 말것 

그외에는 전부 화살표함수를 쓸것 

Arguments 객체 또한 전달되지 않는다. 



## 8. 심볼 



심볼은 프로그램이 이름 충돌의 위험 없이 프로퍼티에 키로 사용하기 위해 생성하고 사용 가능한 값 



element 클래스에 .isMoving()을 추가한다고 생각하면 누가 이미 정의해두었거나, 나중에 정의되거나 하면 골때린다. 이때 



```javascript
var isMoving = Symbol(“isMoving”) // new 키워드는 필요 없으며 인자로 받는 문자열은 주석으로 사용된다. 

element[isMoving] = true; // 심볼 프로퍼티는 닷(.) 으로 접근할 수 없다. 
```



이런식으로 사용한다. 심볼값은 실행 스코프에만 유효하기 때문에 충돌을 예방한다. 모듈에서 정의하면 해당 모듈에만 적용됨 



js의 일반적인 객체 조사에는 감춰진다. `Object.getOwnPropertySymbols()` 로 찾아야함 



생성방법은 세가지 

`Symbol()` -> 매번 다른 심볼 생성 인자로 주어진 문자열이 같더라도 

`Symbol.for()` -> 심볼 레지스트리에서 가져옴 공유 가능 

`Symbol.iterator` -> 정의된 표준 심볼 



새로운 기능이 추가되도 예전코드와의 충돌을 막는다.  



## 9 컬렉션 (Collections) 



Set과 Map은 특별히.. 

WeakSet과 WeakMap이 새로웠는데, 기존의 Map과 Set은 원본 객체가 소멸하더라도, 맵과 셋이 멤버로 참조하고 있으면.(모든 key 와 value를 강력하게 참조한다.) GC 대상이 되지 않아 메모리 릭의 주범이 된다. 

이를 해결하기 위해 WeakSet 과 WeakMap을 만들었다. WeakSet의 value 와 WeakMap의 key는 꼭 객체여야하고 몇가지 메소드들이 제한되고 이터러블하지 않는 등 제약이 좀 있지만 Map, Set과 유사하게 동작한다. 

(이터러블하지 않아서 명시적으로 객체를 찾아야한다.) 



대신 얘들은 멤버의 원본 객체가 소멸되면 함께 소멸된다. (레퍼런스를 보관하지 않음) 그래서 GC는 Weak-Collection과 무관하게 메모리를 관리 



구현이 Ephemeronn table로 구현되어 있는데 weak-reference로 하지 않은 이유는 gc에 영향을 안받게 하기위해.. 벤더마다 gc의 구현이 다르므로.. 뭔소린지 모르겠다.



## 9 제너레이터 이어서 



제너레이터의 동작은 동기적으로 싱글-스레드 환경에서 이루어진다. 



이터레이터의 `return()` 메소드는 이터레이터가 완료를 선언하지 않았는데 루프가 종료됐을 경우 랭귀지에 의해 항상 호출된다. 



제너레이터의 yield 가 `next()` 에 자동으로 맵핑된 것처럼, try-catch 의 finally 블럭도 `return()` 에 자동으로 맵핑되어 해당 블럭을 수행한다.  



이터레이터 프로토콜을 사용할 때만 가능하다. 



### 9.1 제너레이터와 비동기



하나도 이해가 안된다.. 정말 하나도 



## 10 프록시 (Proxy) 



객체의 빌트인 메소드를 오버라이딩 할때. 



### 10.1 객체란 무엇인가? 



ECMAScript 에서 Object는 두가지로 정의된다. 

1. Object 타입에 속한 멤버들 
2. 속성들의 집합 



객체들은 공통으로 갖고 있는 메소드들이 몇개 있는데 

`obj.[[Get]](key, receiver) // receiver는 최초 호출 객체 obj 는 리시버의 프로토타입중 하나 `

obj.prop // obj[key] 로 조회할때 



`obj.[[Set]](key, value, receiver)  `

obj.prop = value // obj[key] = value 로 설정할때 



`obj.[[HasProperty]](key) `

key in obj 로 확인할때 



`obj.[[Enumerate]] `

obj 에 존재하는 열거 가능한 속성들 

`for (key in obj) `



`obj.[[getPrototypeOf]]()  `

obj의 프로토타입을 가져올때 obj.__proto__ // obj.getPrototypeOf() 



`functionObj.[[Call]](thisValue, arguments) `

functionObj() // x.method() 로 호출할때, 모든 객체가 함수인건 아니다. 



`constructorObj.[[Construct]](arguments, newTarget) `

생성자 실행. 모든객체가 생성자인건 아니다. 



프록시는 이러한 빌트인 함수를 오버라이딩 할때 사용한다. 



### 10.2 사용법 



`var proxy = new Proxy(target, handler)  `

직관적으로 동작하는 프록시 객체를 만드는건 생각보다 어렵다. read-only 객체를 만든다고 생각해보면 고려할게 참 많다. 



### 10.3 마무리 



사용하면 유용한곳?  -> 로그를 남기거나, 테스트 객체를 만들거나, 불변객체를 만들거나, 레이지로딩 객체를 만들거나 

weakMap에 캐시해서 쓰면 메모리를 절약할 수 있다. 

객체의 불변조건 



## 12. let 과 const 



### 12.1 var 의 한계 



1. js 함수 안에서 선언한 var 의 스코프는 함수 전체이다. 선언한 곳부터 양쪽으로 퍼져나가 함수의 경계까지 이른다. 마치 함수의 최상단으로 var 구문을 가져오는것과 같다. 이를 호이스팅이라고 한다. 
2. 루프안의 변수가 과도하게 참조된다 -> 말하는 고양이 문제.. 클로저.. 



### 12.2 let이 새로운 var 이다. 



- let은 블럭을 기준으로 스코프를 정한다. 
- 글로벌 let 변수는 글로벌 객체의 속성이 아니다. 
- for(let x …) 구문은 매번 x 변수를 새로 바인딩 한다 -> 매번 다른 클로저를 바라본다. 
- 선언전에 참조하면 에러 
- 두번 선언하면 에러 



### 12.3 const 



선언시 할당한 값을 변경할 수 없음 



## 13. 서브클래스 생성 (Subclassinng) 



### 13.1 Javascript의 상속 



프로토타입으로 부터 속성을 상속받는다. 

Object.create()를 자주 써왔다 



### 13.2 서브클래스 만들기 기초 



프로토 타입 체인을 건드리는게 가끔 과하다 싶을 때가 있다. 

class 구문에선 extends 키워드를 사용 



### 13.3 상속된 클래스의 생성자 



js 생성자에서 this 를 가지고 객체를 조작한다. 속성을 추가하거나 초기화하거나.. 그런데 이 this는 new 구문으로 인해 생성되는데 특정 내장객체들은 특이한 내부 메모리 레이아웃을 갖는다.  



따라서 가장 기본이 되는 (밑바닥의) 생성자가 this를 할당해야한다. super() 이전에 this 를 사용할 수없다. 



new.target = 만드려는 실제 타입, Collections의 하위 객체를 만들때 Map 인지 Set인지 등.. 

