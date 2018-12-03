# You Don't Know Js - 2



# 1. this 와 객체 프로토타입 



## 1. this 라나 뭐라나 



this 는 함수 스코프내에 자동으로 설정된다. 



### 1.1 this 를 왜? 



명시적인 파라미터로 컨텍스트를 넘기는 것 보다 객체 레퍼런스를 암묵적으로 함께 넘기는 ‘this’ 체계가 API 설계상 더 깔끔한 코드를 만들고 재사용성이 나을때가 많다. 

객체와 프로토타입을 배우고 나면 더 명확히 느낄것 



### 1.2 헷갈리는 것들 



사람들은 보통 this를 두가지로 오해를 많이한다. 함수객체 자기 자신과, 함수의 스코프 



#### 1.2.1 자기 자신 



자기참조는 물론 필요하다. 재귀호출을 해야 할 때라던지 이벤트 최초 호출시 바인딩된 함수 자기 자신을 언바인딩 할때라던지, 그렇지만 this 는 그럴때 쓰기위한게 아니다. 



예로 

```javascript
function foo (num) { 
    console.log(num);
    this.count++; 
} 

foo.count = 0; 

for (var i = 0 ; i < 10 ; i++) { 
    if (i > 5) { 
        foo(i); 
    } 
} 
```



를 하면 출력은 5번 되지만 count 는 0이다. 

엄한 전역객체의 count가 생겼는데 심지어 이시점에 얘의 값은 NaN 이된다.. 



단순히 이걸 회피하기 위해 var data = { count : 0 } 이런 외부 객체를 만드는건 너무 임시방편이고 

이런 이름이 있는 함수는 렉시컬 스코프에 의해 자신의 이름으로 찾을 수 있지만 (foo.count++) 익명함수는 이럴 방법도 없다. 



arguments.callee 로 호출 가능하나 이건 비표준이 되었으니 쓰지말자



아니면 this 가 foo 객체를 가르키도록 this 바인딩을 한다. 

foo.call(foo, i); 



#### 1.2.2 자신의 스코프 



두번째로 많이 하는 오해가 함수가 참조하는 스코프로 이해하는 것인데 이것도 땡이다. 



스코프 “객체” 는 자바스크립트 구현체인 “엔진” 의 일부로 일반 자바스크립트 코드로는 참조할 수 없다. 

this 와 렉시컬 스코프사이의 연결통로 따위는 존재하지 않는다. 



### 1.3 this 란 무엇인가. 



this 는 작성 시점이 아닌 런타임 시점에 바인딩 되며, 함수 호출 당시 상황에 따라 컨텍스트가 결정된다. 함수 선언과 상관없이 어떻게 함수를 호출했는가가 결정짓는다. 



어떤 함수를 호출하면 활성화 레코드, 즉 실행 환경이 만들어지고 이 실행 환경에는 호출된 근거와 (콜스택), 호출 방법, 전달된 인자등의 정보가 담기는데 this 레퍼런스는 이들중 하나이다. 함수가 실행되는 동안 살이있다. 



## 2. this가 이런 거로군!



### 2.1 호출부



this 의 바인딩 객체를 알려면 함수의 호출부를 알아야한다.

생각보다 호출부를 찾는게 쉽지 않은 경우가 있는데, 이때 중요한건 호출스택을 잘 확인하는것. 함수의 호출부는 현재 실행중인 함수 직전의 호출코드 내부에 있다.



#### 2.2.1 기본 바인딩



함수 단독 실행



나머지 규칙에 해당하지 않을 경우 적용되는 this 바인딩 기본 규칙이다.

이때 엄격모드일 경우 undefined가 비엄격 모드일 경우 전역객체가 바인딩 되고 엄격모드는 호출부가 아닌 함수 본문이 엄격모드여야 한다.



#### 2.2.2 암시적 바인딩



호출부에 컨텍스트 객체가 있는지 확인한다. 즉 객체의 함수 레퍼런스 소유 / 포함 여부를 확인한다.



```javascript
function foo() {
	console.log(this.a);
}

var obj = { 
	a : 42;
	foo : foo;
} 

obj.foo(); // 2 
```



obj가 실제로 foo 를 갖고 있는건 아니지만 (함수 레퍼런스를 참조할 뿐) obj.foo() 의 호출부에서 obj 라는 컨텍스트를 통해 접근하기 때문에 함수 레퍼런스를 소유/포함 한다고 할 수 있다. 이땐 이 obj 가 this 에 바인딩 된다. 



체이닝 될때는 맨 마지막 객체를 참조한다. 



##### 암시적 소실 



소실되면 기본 규칙을 따른다.  

```javascript
function foo() { 
	console.log(this.a);
} 

var obj = { 
	a : 2,
	foo : foo
} 



var a = “ㅎㅎ”; 

var bar = obj.foo; 

bar() // “ㅎㅎ”; 
```



var bar = obj.foo 는 foo() 를 가르키는 또다른 레퍼런스를 갖는다. obj.foo 를 가르키는게 아니라 (js는 변수끼리 참조할 수 있는 방법따윈 존재하지 않는다) 

그래서 그냥 foo() 한것과 마찬가지. 그래서 기본규칙을 탄다. 



#### 2.2.3 명시적 바인딩 



call() , apply() 메소드를 사용한다. 

두 메서드는 this에 바인딩할 객체를 첫번째 파라미터로 받아 함수 호출시 이 객체를 this 에 명시적으로 바인딩 한다. 

원시값을 넘기면 박싱된다. 



##### 하드 바인딩 



자주 사용하는 패턴이라 `Function.prototype.bind` 로 추가되었다. 

얘는 기존 함수에 파라미터로 받은 객체가 하드바인딩된 함수를 반환한다. (무조건 `this` 를 이 파라미터로 바인딩한) 



##### API 호출 컨텍스트 



`Array.prototype.forEach` 처럼 `this` 를 자동으로 바인딩해주는 것 



#### 2.2.4 new 바인딩 



흔히 오해들 하는데 자바스크립트에서 `new` 연산자는 의미상 클래스 지향적인 기능의 생성자와 아무런 상관이 없다..



자바스크립트의 생성자는 `new` 연산자가 있을 때 호출되는 일반 함수에 불가하다. 

클래스에 붙은것도 아니고 클래스를 인스턴스화 하지도 않는다. 



생성자 호출시 다음과 같은 일이 자동으로 일어난다. 



1. 새 객체가 툭 만들어진다. 
2. 새로 생성된 객체의 `[[Prototype]]` 을 연결한다. 
3. 새로 생성된 객체는 함수 호출시 `this` 로 바인딩 된다. 
4. 이 함수가 자신의 또다를 객체를 반환하지 않는 한 `new` 와 함께 호출된 함수는 방금 생성된 객체를 반환한다. 



### 2.3 모든일은 순서가 있는 법 



`new` 바인딩 -> 명시적 바인딩 -> 암시적 바인딩 -> 기본 바인딩 순으로 적용된다. 



`new` 바인딩으로 하드 바인딩을 오버라이딩 하려는 이유는 뭘까? 

함수 인자를 전부 또는 일부만 미리 셋팅하기위해 -> 커링 

`bind()` 함수는 최초 `this` 바인딩 이후 전달된 인자를 원함수의 기본인자로 고정하는 역할을 한다. 

`bind()`의 두번째 인자부터가 `new` 의 인자 보다 먼저 인자로 잡힌다. 



```javascript
function foo (p1, p2) { 
	this.val = p1 + p2;
} 



var bar = foo.bind(null, “p1”); 
var baz = new bar(“p2”); 
baz.val; // p1p2 
```



#### 2.3.1 this 확정 규칙 



1. `new` 함수로 호출했는가? -> 새로 생성된 객체가 `this`다 
2. `apply`, `call`, `bind` 로 호출했는가? -> 명시적으로 지정된 객체가 `this`다
3. 함수를 컨텍스트, 즉 소유하거나 포함하는 형태로 호출 했는가? -> 이 컨텍스트 객체가` this` 다 
4. 그 외의 경우에는 기본값으로 바인딩된다. 



### 2.4 바인딩 예외 



규칙엔 예외가 있다. 



#### 2.4.1 this 무시 



`call`, `apply`, `bind` 메서드에 첫번째 인자로 `null` 혹은 `undefined` 를 넘기면 `this` 바인딩은 무시되고 기본바인딩 규칙이 적용된다. 

왜 `null` 같은 값으로 `this` 바인딩을 하려 하냐면 다음 두가지다 



```javascript
function foo (a,b) { 
	console.log(“a:” + a + “, b :” + b);
} 

foo.apply(null, [1,2]); // a: 2, b: 3 배열을 넣으면 파라미터로 폃쳐진다 / es6부터 …(Spread Operation) 으로 대체가능하다 foo(…[1,2]) 

var bar = foo.bind(null, [1,2]); // 커링한다. 아직 대체할 연산자는 존재하지 않는다. 
bar (3); // a : 2, b : 3 
```



그치만 `null` 을 애요하는건 예기치 못한 기본바인딩을 일으켜 전역객체를 참조하거나 변경하는 돌발 상황이 일어날 수 있다. 



##### 더 안전한 this 



더 안전하게 가자면 side-effect 가 전혀 없는 DMZ 객체. 즉 내용이 하나도 없으면서 전혀 위임되지 않은 객체가 필요하다. 



DMZ 객체를 만드는 가장 쉬운 방법은 `Object.create(null)` 을 사용하는것 `{}` 는 `Object.prototyp`으로 위임하기 때문에 이보다 더 비어있는 객체라고 할 수 있겠다. 



#### 2.4.2 간첩 레퍼런스 



의도적이든 아니든 한가지 더 유의할 것은 간접 레퍼런스 Indirect Reference 를 생성하는 경우로, 함수를 호출하게 되면 무조건 기본 바인딩 규칙이 적용된다. 



간접레퍼런스는 함수 할당문에서 빈번하게 나타난다. 



```javascript 
function foo() { 
	console.log(this.a);
} 

var a = 2; 
var o = {a: 3, foo: foo}; 
var p = {a:4}; 

o.foo(); // 3 
(p.foo = o.foo)() // 2 
```



p.foo = o.foo 의 결과는 원함수 객체의 레퍼런스이므로 p.foo() / o.foo()가 아니라 foo() 가 실행된다.



#### 2.4.3 소프트 바인딩



하드 바인딩은 함수의 유연성을 많이 떨어뜨리기 때문에 사람들은 소프트 바인딩 유틸리티를 고안했다



```javascript
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function (oj) {
		var fn = this;
		// 커링된 인자는 죄다 포착한다
		var curried = [].slice.call(arguments, 1);
		var bound = function() {
			return fn.apply(
				(!this || this === (window || global)) ?
					obj : this
				curried.concat.apply(curried, arguments);
			);
		};
		bound.prototype = Object.create(fn.prototype);
		return bound;
	};
}
```



### 2.5 어휘적 this 



위 규칙이 적용되지 않는 케이스가 es6에 추가된 화살표함수이다. 

4가지 표준 규칙 대신 에두른 스코프 Enclosing scope 를 보고 알아서  `this` 를 바인딩 한다 



```javascript 
function foo() { 
	return (a) => {
		console.log(this.a);
	}
} 

var obj1 = { 
	a : 2
} 

var obj2 = { 
	a : 3
} 

var bar = foo.call(obj); 
bar.call(obj2); // 2!!!! 3이 아니다!!!  
```



`foo()` 내부에서 생성된 화살표 함수는 `foo()` 호출시 `this`를 무조건 어휘적으로 포착한다. `foo()` 는 obj 에 `this` 가 바인딩 된다. 화살표 함수의 어휘적 바인딩은 절대로 오버라이딩 될 수없다. 



### 2.6 정리 



함수 실행에 있어 `this` 는 함수가 어떻게 호출되는가에 의해 결정된다. 호출부를 식별한 후 다음 네가지 규칙을 따른다. 



1. `new` 로 생성된 새로운 객체가 바인딩 된다. 
2. `apply` 나 `call` 로 명시적으로 전달된 객체가 바인딩 된다. 
3. 호출된 함수를 포함 / 소유하는 컨텍스트 객체가 존재할 경우 해당 객체가 바인딩 된다. 
4. 엄격 모드에선 `undefined` 가 비엄격 모드에서는 전역 객체가 바인딩 된다. 



실수로 기본 바인딩이 되지 않도록 주의하자. 비어있는 DMZ 객체를 `this` 로 바인딩 하고 싶으면 `Object.create(null)` 을 사용하자. 이는 커링할 때 (인자를 기본적으로 주어줄 때) 유용하다. 



ES6의 화살표 함수는 표준 `this` 바인딩 규칙을 따르지 않고 렉시컬 스코프에서 `this` 를 상속한다. 즉 함수의 에두른 스코프에서 상속 받으며 이는 `new` 조차로도 오버라이딩 할 수 없다. 과거 `var _self = this` 꼼수를 대체한것. 



## 3. 객체 



객체는 정확히 뭐고, 왜 가르켜야 하는가 



### 3.1 구문 



객체는 선언적 Declarative (리터럴 Literal) 형식과 생성자 형식 두가지로 정의한다. 

결과적으로 생성되는 객체는 같으나 리터럴은 여러 프로퍼티를 한번에 줄 수 있는 반면, 생성자 형식은 한번에 하나의 프로퍼티만 선언 가능하다. 그래서 생성자 형식으로 객체를 생성하는 일은 거의 없다. 



### 3.2 타입 



“자바 스크립트의 모든것은 객체다” 라는 말은 명백히 잘못됐다. 단순원시타입(null, undefined, boolean, number, string ) 는 객체가 아니다 `typeof null` 이 object 를 반환해 빠지는 착각일뿐 



복합원시타입은 객체의 독특한 하위타입이다. (function - 호출가능한 객체, 배열) 

자바스크립트 함수는 기본적으로 객체이고 일급이여서 다른 객체와 똑같이 취급된다. 

1급 (First class) 인자로 전달가능하고 반환값으로 반환받을 수 잇으며 변수에 할당하거나 자료구조에 저장할 수 있다. 



#### 3.2.1 내장 객체 



내장 객체라고 부르는 객체 하위타입도 있다. 이름만 보면 대응되는 단순 원시타입과 직접 연관되어 보이지만 실제로는 복잡한 관계에 놓여있다. 



String, Number, Boolean, Object, Funciton, Array, Date, Regexp, Error 



### 3.3 내용 



객체는 특정 위치에 저장된 모든 타입값, 즉 프로퍼티로 내용이 채워진다. 

엔진이 값을 저장하는 방식은 구현 의존적이기는 하지만 보통 객체 컨테이너에 담지 않는게 일반적이다. 컨테이너는 실제 프로퍼티 값을 가리키는 레퍼런스가 담겨있다. 



#### 3.3.1 계산된 프로퍼티명 



ES6에 추가된 구문 



```javascript
var preifx = “foo”; 
var myOjb = { 
	[prefix + “bar”] : “hello"
} 

myObj[“foobar”]; // “hello” 
// 아마 심볼을 위해서 많이 쓰게 될것이다. 
var symbolObj = { 
	[Symbol.something] : “helloworld”
} 
```



#### 3.3.2 프로퍼티 vs 메소드 



엄밀히 말해 함수는 객체에 속하는 것이 아니며, 객체 레퍼런스로 접근한 함수를 메소드라 칭하는 건 과대해석이다. 

함수 표현식을 객체 리터럴에 한 부분으로 선언해도 이 함수가 객체에 달라붙는 것은 아니며 해당 함수 객체를 참조하는 레퍼런스가 하나 더 생길 뿐이다. 



#### 3.3.3 배열 



일반 객체보다 값을 저장하는 방법과 장소가 더 체계적이다. 

용도에 맞게 쓰자 



#### 3.3.4 객체복사 



기본적으로 동일한 원본을 가르키는 또다른 레퍼런스를 할당하는 얕은 복사가 이루어진다. 

ES6 부터는 `Object.assign()` 메소드를 사용하여 얕은 복사를 할 수 있다, ( 순회하며 = 할당을 하기 때문에 writable 같은 특수한 프로퍼티의 속성은 복사되지 않는다,) 



#### 3.3.5 프로퍼티 서술자 



ES5 부터 모든 프로퍼티는 프로퍼팉 서술자로 표현된다. 



```javascript
var myObject = { 
	a : 2
}

Object.getOwnPropertyDescriptor(myObject, “a”);


// {
//     value : 2,
//     writable : true,
//     enumerable : true,
//     configurable: true
// }
```



`Object.defineProperty()` 로 새로운 프로퍼티를 추가하거나 기존의 프로퍼티의 특성을 변경할 수 있다 (configurable 이 ture 일 경우)



##### 쓰기가능



프로퍼티의 쓰기가능 여부는 `writable` 로 조정한다. 쓰기 금지된 값을 수정하면 조용히 실패하며 엄격 모드에서는 TypeError 를 뱉는다.



##### 설정가능



`configurable` 이 true 는 `Object.defineProperty()` 로 프로퍼티 서술자를 변경할 수 있다. 한번 false 로 설정되면 돌이킬 수 없으며 `delete` 도 불가능 하다.





##### 열거가능성 



`for..in` 구문에서 노출되게 하냐 마냐 



##### 불변성 



자바스크립트에서 뼛속까지 고정된 불변객체를 쓸 일은 거의 없다. 필요한 경우도 있겠지만, 객체를 봉인 또는 동결 해야하는 상황이라면 객체값이 변경되어도 문제가 없는 견고한 프로그램을 설계할 수 있는 다른 방법은 없는지 디자인 패턴 관점에서 재고해라



##### 객체상수



`writable : false` 와 `configurable : false` 이면 객체 프로퍼티를 상수처럼 쓸 수 있다.



##### 확장금지



`Object.preventExtension()` 을 호출하면 객체에 더는 프로퍼티가 추가되지 않는다.



##### 봉인



`Object.seal() = Object.preventExtension() + configurable : false`



##### 동결



`Object.freeze() = Object.seal() + writable : false` 



`Object.freeze()` 를 호출하면 지금까지 영향을 받지 않았던 해당객체가 참조하는 모든 객체를 재귀순회하며  `Object.freeze()` 를 호출해 깊숙히 동결한다. 의도치 않은 공유된 다른 객체까지 동결시키지 않도록 조심할것. 



### 3.3.7 [[Get]] 



```javascript
var myObject = { 
	a : 2
} 

myObject.a; // 2 
```



이 코드에서 `a` 란 프로퍼티를 `myObject` 에서 찾지 않는다. `myObject` 에 대하여 `[[Get]]` 연산을 한다. 

기본적으로 `[[Get]]` 연산은 주어진 이름의 프로퍼티를 먼저 찾아보고 어떻게 해도 찾을 수 없다면 `undefined` 를 반환한다. 

식별자명으로 호출하면 동작방식이 조금 다르다. 해당하는 렉시컬 변수 내 없는 변수를 참조하면 `ReferenceError` 가 발생한다. 

명시적인 값이 `undefined` 인지 프로퍼티를 찾을 수 없어 `undefined` 인지는 구분하는 법은 후에 설명한다 



### 3.3.8 [[Put]] 



이미 존재하는 프로퍼티에 대해선 다음과 같은 알고리즘을 따른다 

1. 프로퍼티가 접근 서술자인가? 맞다면 세터를 호출한다. 
2. 프로퍼티가 `writable : false` 인 데이터인가? 맞다면 비엄격모드에선 조용히 실패하고 엄격모드에선 `TypeError` 가 발생한다. 
3. 이외에는 프로퍼티 값을 셋팅한다. 

객체에 존재하지 않는 프로퍼티라면 알고리즘은 미묘해진다 -> 5장 프로토타입에서 설명한다. 



### 3.3.9 게터와 세터 

ES5 부터는 게터/세터를 통해 객체 수준이 아닌 프로퍼티 수준에서 값을 셋팅하고/조회하는 기본로직을 오버라이딩 할 수 있다. 

프로퍼티가 게터/세터 가 될 수 있게 정의한것을 접근 서술자 `Access Descriptor` 라고 한다. 



```javascript
var myObject = { 
	get a() {
		return 2;
	}
} 

Object.defineProperty( 
	myObject,
	“b”,
	{
		get: function() {
			return this * 2;
		},
		enumerable: true
	}
); 

myObject.a; // 2 
myObject.b; // 4 
```



당연하지만 게터/세터 한쪽만 선언하면 예상외의 결과를 얻을 수 있다. 



#### 3.3.10 존재확인 



```javascript
var myObject = { 
	a : 2
} 

// in 은 이 객체와 [[Prototype]] 체인까지 참조 
(“a” in myObject );  // ture 
(“b” in myObject);  // false 

// hasOwnProperty 는 해당 객체만 참조 
myObject.hasOwnProperty(“a”) // true 
myObject.hasOwnProperty(“b”) // true 
```



in 은 프로퍼티 명이 존재하는 지만 확인한다. 즉 4 in [2,3,4] 는 실행되지 않는다. 



##### 열거가능 



열거가능하다는건 객체 프로퍼티 순회 리스트에 포함된다는걸 의미한다. 



```javascript
var myObject = {}; 

Object.defineProperty( 
	myObject,
	“a”,
	{enumerable : true, value : 2}
); 

Object.defineProperty( 
	myObject,
	“b”,
	{enumerable : false, value : 3}
); 

// 해당 객체만 
myObject.propertyIsEnumerable(“a”); // true 
myObject.propertyIsEnumerable(“b”); // false 

// 주어진 객체만 
Object.keys(myObject); // [“a”] 
Object.getOwnPropertyNames(myObject); // [“a”, “b”] 
```

in 연산자 결과와 동일한 프로퍼티 리스트를 조회하는 기능은 아직 없다. 