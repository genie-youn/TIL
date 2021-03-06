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
var preifx = "foo";
var myOjb = {
	[prefix + "bar"] : "hello";
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



### 3.4 순회



ES5 부터는 `foreach()`,` some()`, `every()` 와 같은 배열 순회 헬퍼가 도입됐다.



```javascript
foreach(); // 전체 순회

every(); // 콜백 함수가 false 를 반환할 때 까지

some();  // 콜백함수가 true 를 반환할 때 까지

```



es6 부터는 프로퍼티 값을 순회하기 위한 `for..of` 구문을 제공한다.



### 3.5 정리하기



자바스크립트 객체는 리터럴 형식과 생성자 형식 두가지 형태를 가진다. 대부분 리터럴 형식을 쓰는게 좋지만 생성시 옵션을 더 주기위해 생성자 형식을 쓰는 경우도 더러 있다.



많은 사람들이 자바스크립트의 모든것이 객체다 라고 말하지만 사실과 다르다. 객체는 6개의 원시타입중 하나고 함수를 비롯한 하위 타입이 있다.



객체는 키/값의 쌍을 모아놓은 저장소고 값은 프로퍼티를 통해 (.propName / [“propName”]) 접근할 수 있다. 프로퍼티에 접근하면 엔진 내부에서는 기본 [[Get]] (or [[Set]]) 연산을 호출하는데 객체 자체에 포함된 프로퍼티뿐 아니라 필요하면 [[Prototype]] 체인을 참조하여 순회하며 찾는다.



## 4 클래스와 객체의 혼합



클래스 지향의 디자인 패턴은 자바스크립트의 객체 체계와는 태생부터 잘 맞지 않는다.



### 4.1 클래스 이론



클래스와 상속은 특정 형태의 코드와 구조를 형성하여 실생활 영역의 문제를 소프트웨어로 모델링 하기위한 방법이다.



### 4.1.1 클래스 디자인 패턴



클래스 역시 디자인 패턴중 하나이다. 너무 기본적인 형태라 마치 모든것의 근본인 것 처럼 받아들여지는데, 함수형 프로그래밍을 떠올려볼것.  



#### 4.1.2 자바스크립트 클래스



자바스크립ㅌ트에선 어떤가? 오래전부터 클래스와 비슷하게 생긴 구문 (`new` 나 `instanceof` 와 같은)을 갖추고 있고 es6 에선 아예 `class` 키워드가 명세에 추가 되었다,.

하지만 단도직입 적으로 자바스크립트에는 클래스가 없다.



클래스는 디자인 패턴이므로 적잖이 공을 들이면 고전적인 클래스 기능과 얼추 비슷하게 구현할 수야 있다.



이장은 자바스크립트가 제공하는 클래스라는 환상을 유지하기 위해 치뤄야 하는 댓가에 대해 설명한다.



### 4.2 클래스 체계



Stack 클래스는 그저 모든 스택이라면 마땅히 해야 할 기능을 추상화 한 것이지 그 자체가 스택은 아니다. Stack 클래스를 인스턴스화 해야 비로소 작업을 수행할 구체적인 자료구조가 마련된다.



#### 4.2.1 건축



건축에 비유하면 클래스는 설계도 (청사진) 인스턴스는 실제 지어진 건물이라 할 수 있겠다.



#### 4.2.2 생성자



### 4.3 클래스 상속



#### 4.3.1 다형성



기존 클래스 지향 언어에서 ‘상대적 다형성’ 이라고 표현한 이유는 한 메소드가 상위 수준의 상속체계에서 다른 메서드를 참조할 때 접근할 상속 수준에 대한 절대적 기준이 없는 상태에서 ‘한 수준 더 상위로’ 를 외치며 상대적 레퍼런스를 거슬러 올라가기 때문이다.



자식 클래스가 마치 부모 클래스에 연결된 양 다형성을 혼동하지 않기를 바란다.

자식은 그저 부모에게서 자신이 필요한 내용을 배껴올 뿐이다. 클래스 상속은 한마디로 복사다. (ㅁ루론 부모 클래스를 가르키는 상대적 레퍼런스 super 는 주어진다.)



#### 4.3.2 다중상속



복잡하고 많은 문제를 갖고 있는 다중상속을 자바스크립트는 지원하지 않는다.

다만 사람들은 다중상속을 흉내내기 위해 많은 노력을 해 왔다.



### 4.4 믹스인



자바스크립트는 객체를 상속받거나 인스턴스화를 해도 자동으로 복사되지 않는다.

자바스크립트는 클래스란 개념이 없고 오로지 객체만 존재한다.

그리고 객체는 다른 객체에 (상속받을 때) 복사되는게 아닌 연결된다.



mixin 은 이 객체각 (상속을 통한) 복사를 흉내낸 기능이며 명시적 믹스인과 암시적 믹스인이 존재한다.



#### 4.4.1 명시적 믹스인



다음과 같이 mixin 을 구현할 수 있겠다. ( 여타 라이브러리나 프레임워크에선 extends 란 키워드를 자주 사용한다.)



```javascript
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		if ( !(key in targetObj) ) {
			targetObj[key] = sourceObj[key];
		}
	}
	return targetObj;
}
```



##### 다형성 재고



`Vehicle.drive.call( this );` 같은 코드를 명시적 의사다형성 [^_Explict Pseudopolymorphism_] 이라 부른다. 상대적 다형성과는 반대로 콕집어 부모의 함수를 호출한다.



ES6 이전에 자바스크립트는 상대적 다형성을 제공하지 않는다.

따라서 `drive()` 란 이름의 함수가 Vehicle 과 Car 양쪽에 모두 존재한다면

(상대적이 아닌) 절대적 레퍼런스를 사용할 수 밖에 없다 ( Vehicle.drive.call( this ) )



명시적 다형성은 더 복잡하고 더 읽기 어려운 코드가 된다. 장점보다는 비용이 훨씬 더 많이 들기 때문에 가능한 한 쓰지 않는게 좋다.



자바스크립트 함수는 복사할 수 없다. 복사되는건 함수 공유 객체를 가리키는 사본 레퍼런스이다.  



명시적 믹스인은 코드 가독성에 도움이 될 때만 사용하되, 점점 코드를 추적하기 어렵고 객체간 관계가 난해해진다면 사용을 중단해라.



##### 기생상속 [^parastic inheritance]



기생상속은 더글라스 크록포드가 작성한 명시적 믹스인 패턴의 변형으로 명시적 / 암시적 요소를 모두 갖고 있다.



```javascript
function Vehicle() {
	this.engines = 1;
}



Vehicle.prototype.ignition = function() {
	console.log(“엔진을 켠다”);
}



Vehicle.prototype.drive = function() {
	this.ignition();
	console.log(“방향을 맞추고 앞으로 간다”);
}



function Car() {
	var car = new Vehicle();
	car.wheels = 4;

    // Vehicle::drive() 의 내부 레퍼런스 생성
	var vehDrive = car.drive;

	// Vehicle::drive() 오버라이드
	car.drive = function() {
		vehDrive.call( this );
		console.log(this.wheels + “개의 바퀴로 굴러간다.”);
	}
	return car;
}

var car = new Car(); // new 없이 호출해도 똑같다. 저 함수가 새 객체를 반환하므로.. 오히려 불필요한 객체생성과 가비지컬렉션을 유발한다
myCar.drive();
```

#### 4.4.2 암시적 믹스인

```javascript
var Something = {
  cool : function () {
    this.greeting = "Hello world";
    this.count = this.count ? this.count + 1 : 1;
  }
}

Something.cool();
Something.greeting; // "Hello world"
Something.count; // 1

var Another = {
  cool : function () {
    Something.cool.call(this);
  }
}

Another.cool();
Another.greeting; // "Hello world"
Another.count; // 1
```

`Something.cool.call(this)` 같은 코드는 상대적 레퍼런스가 되지 않아 유연하긴 하나 불완정하므로 신중히 사용해야 한다. 대부분의 경우에 깔끔하고 유지보수하기 쉬운 코드를 위해 사용하지 않는 편이 좋다.

### 4.5 정리하기

클래스는 디자인 패턴중 하나이고 자바스크립트 클래스는 전통적인 클래스 지향의 소프트웨어 디자인이 가능한 언어의 클래스와는 사뭇 다르다.

클래스는 복사를 의미한다. 인스턴스화 될 때는 클래스 -> 인스턴스로 상속할 때는 부모 -> 자식으로 복사가 일어난다.

자바스크립트는 객체간 사본을 자동으로 생성하지 않으며 믹스인을 통해 상속을 흉내낼 수 있다. 하지만 명시적 의사다형성 (상대적 레퍼런스가 아니기 때문에 OtherObj.methodName.call(this)) 처럼 가독성이 떨어지는 코드를 만든다.

명시적 믹스인은 클래스의 복사기능과 같지 않다. 이는 객체 자체가 아니라 공유된 레퍼런스만 복사하기 때문 그래서 예기치 못한 문제가 발생할 수 도 있다.

일반적으로 자바스크립트에서 클래스를 모방하는건 좋은 해결책은 아니다.

## 5 프로토타입

### 5.1 [[Prototype]]

자바스크립트 객체는 [[Prototype]] 이라는 내부 프로퍼티가 있고 객체 생성시에 할당되며 다른 클래스를 참조하는 단순 레퍼런스로 사용한다.
[[Prototype]]이 텅 빈 객체도 생성은 가능하다.

자바스크립트에서 `obj.propName` 으로 접근하면 [[Get]] 내부 연산을 호출하는데 해당 객체에 찾으려는 프로퍼티가 존재하지 않으면 프로토타입 체인을 끝까지 순회하며 찾고, 그래도 없다면 `undefined` 를 반환한다.

> ES6 의 Proxy 개입하면 이야기가 달라진다.

`for ... in` 구문의 `in` 연산자도 마찬가지로 프로토타입 체인 전체를 순회한다.

#### 5.1.1 Object.prototype

모든 자바스크립트 객체는 `Object.prototype` 객체의 '자손' 이므로 프토토타입 체인은 이 객체에서 끝난다.
다수의 공통 유틸리티를 포함하고 있다. `toString()` 이라던지 `valueOf()` 라던지.

#### 5.1.2 프로퍼티 셋팅과 가려짐

객체 프로퍼티 셋팅은 단지 어떤 객체에 프로퍼티를 새로 추가하거나 기존 프로퍼티 값을 바꾸는것 이상의 의미가 있다.

```javascript
myObject.foo = "bar";
```

위 예제에서 `foo`가 `myObject` 의 직속으로 존재한다면 단순히 값을 변경하지만 존재하지 않을 경우 [[Prototype]] 체인을 순회하고 상위에 `foo` 프로퍼티가 존재하지 않을 경우에만 프로퍼티를 새로 생성하고 값을 할당한다. 만약 체인 상위에 `foo` 프로퍼티가 존재한다면 이 할당문을 조금 골때린다.

이 `foo` 라는 프로퍼티가 `myObject` 에도 존재하게 되고 [[Prototype]] 체인에도 존재하게 된다면 하위부터 찾기 때문에 항상 `myObject` 객체의 프로퍼티만 찾게 된다. 즉 프로토타입 체인의 프로퍼티가 이 객체의 프로퍼티에 의하여 가려지게 된다. 이를 가려짐 [^Shadowing] 이라고 한다.

위의 골때리는 상황을 이어서 보자. `myObject` 에 `foo` 라는 프로퍼티가 존재하지 않고 [[Prototype]] 체인에 `foo` 프로퍼티가 존재할 경우 다음과 같은 세가지 경우의 수가 존재한다.

1. [[Prototype]] 체인의 상위 수준에서 foo 라는 이름의 일반 데이터 접근 프로퍼티가 존재하는데, 읽기 전용 (writable:false) 이 아닐 경우, myObject의 직속 프로퍼티 foo 가 새로 추가되어 결국 '가려짐 프로퍼티'가 된다.
> 즉 writable 이 true 일 경우 재정의한다고 이해하면 될 것 같다.

2. [[Prototype]] 체인의 상위 수준에서 foo 라는 이름의 일반 데이터 접근 프로퍼티가 존재하는데, 읽기 전용 (writable:false) 일 경우 이 프로퍼티를 세팅하거나 myObject 객체에 가려짐 프로퍼티를 생성하는 따위의 일은 일어나지 않는다. 엄격모드에선 에러가 나고 비엄격모드에서는 조용히 무시한다. **가려짐은 발생하지 않는다.**

3. [[Prototype]] 체인의 상위 수준에서 발견된 foo가 세터일 경우 항상 이 세터가 호출된다. myObject에 가려짐 프로퍼티 foo 를 추가하지 않으며 foo 세터를 재정의 하는일 또한 없다.

2번이 조금 특이한데, 프로토타입 체인의 상위에 읽기 전용으로 선언된 프로퍼티가 존재하면 하위수준에서 생성되지 못하게 한다. (가려지지 않게 한다) 이는 지극히 클래스 지향 프로퍼티라는 환상을 강화하려는 의도에서 비롯되었다. 마치 프로토타입 체인 상위 객체로부터 상속받은 프로퍼티를 굳히는 것은 타당해보이나, 자바스크립트에서 상속에 따른 복사따위는 전혀 일어나지 않는다는것을 고려하면 ([[Prototype]] 이라는 프로퍼티로 상위 객체를 단순히 참조할 뿐이다) 다소 억지스럽다. 그나마 = 할당에만 이런 제약이 있을 뿐 `Object.defineProperty` 로 가릴려면 충분히 가릴 수 있다.

메서드 간 위임이 필요한 상황이면 메서드 가려짐으로 인해 보기 안 좋은 명시적 의사다형성이 유발된다. 가려짐은 그 이용 가치에 비해 지나치게 복잡하고 애매하니 될 수 있으면 사용하지 말것. **작동 위임 패턴** 을 적용하면 좀 더 깔끔하게 가려짐을 대체할 수 있다.

가려짐은 미묘하게 암시적으로 발생할 수 있으니 이런 경우는 더 유의해야 한다.

```javascript
var anotherObject = {
  a : 2
};

var myObject = Object.create( anotherObject );

anotherObject.a;   // 2
myObject.a;        // 2

anotherObject.hasOwnProperty("a"); // true
myObject.hasOwnProperty("a");      // false

myObject.a++;      // 암시적 가려짐

anotherObject.a;   // 2
myObject.a         // 3

myObject.hasOwnProperty("a");   // true
```

겉보기엔 `myObject.a++` 가 `anotherObject.a` 를 찾아서 1을 증가시킬것 같지만 이 연산을 풀어쓰면
`myObject.a = myObject.a + 1` 일 뿐이다. 즉 [[Get]]으로 프로토타입 체인을 뒤져서  anotherObject.a 의 값인 1을 가져오고 여기에 1을 더해서 [[Put]] 으로 myObject에 가려짐 프로퍼티인 a를 새로 생성한 뒤 할당한다.

그러므로 위임을 통해서 프로퍼티를 수정하려고 할 땐 조심해야한다. anotherObject.a 를 1 증가시키려면 `anotherObject.a++` 밖에 방법이 없다.

### 5.2 클래스

자바스크립트에는 다른 클래스 지향 언어와 달리 클래스라는 추상화된 패턴이나 설계가 존재하지 않는다. 다만 객체만 있을 뿐이다.

#### 5.2.1 클래스 함수

자바스크립트의 특성으로 인해 클래스처럼 생긴 무언가를 만들려는 꼼수가 계속 되었다. 자바스크립트의 '일종의 클래스' 같은 독특한 동작은 모든 함수가 기본으로 프로토타입이라는 공용/열거 불가 프로퍼티를 가진다는 특성에 기인한다.

```javascript
function Foo() {

}

Foo.prototype; // {}
```

이 `Foo.prototype` 객체를 Foo의 프로토타입 이라고 한다. 이 객체는 `new Foo()` 로써 만들어진 모든 객체는 결국 이 Foo의 프로토타입 객체와 [[Prototype]] 링크로 연결된다.

```javascript
function Foo() {

}

var a = new Foo();
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

`new Foo()` 로 a 가 생성될 때 일어나는 네가지 사건중 하나가 Foo.prototype 이 가르키는 객체를 내부 [[Prototype]] 에 연결하는 것이다.

>
1. 새 객체가 툭 만들어진다.
2. 새로 생성된 객체의 `[[Prototype]]` 을 연결한다.
3. 새로 생성된 객체는 함수 호출시 `this` 로 바인딩 된다.
4. 이 함수가 자신의 또다를 객체를 반환하지 않는 한 `new` 와 함께 호출된 함수는 방금 생성된 객체를 반환한다.

클래스 지향 언어에서는 붕어빵 틀에서 붕어빵을 굽듯 한 클래스를 여러개의 인스턴스로 인스턴스화 할 수 있다. 인스턴스화가 '클래스 작동 계획을 실제 객체로 복사하는 것' 이므로 인스턴스화마다 복사가 일어난다.

하지만 자바스크립트에선 하나의 클래스에서 여러 인스턴스를 생성할 수 없다. 어떤 공용 객체에 [[Prototype]] 으로 연결된 다수의 객체를 생성하는건 가능하지만 기본적으로 어떤 복사가 일어나는게 아니라 같은 객체를 참조할 뿐이라 결과적으로 자바스크립트에서 객체들은 서로 완전히 떨어져 분리되는것이 아니라 끈끈하게 연결된다.

`new Foo()` 로 새로운 객체 (a) 를 만들고 이 객체는 Foo.prototype 객체와 내부적인 [[Prototype]] 프로퍼티로 연결된다. 인스턴트화 과정따위는 존재하지 않으며 어떤 '클래스'로부터 작동을 복사하여 다른 객체에 넣은 적이 없다. 두 객체를 연결할 뿐이다.

많은 개발자가 잘 모르는 한 가지 비밀을 소개한다. 사실 `new Foo()` 호출 자체는 이러한 '링크'의 생성 프로세스와 거의 관련이 없다.
> 그럼 뭐랑 관련이 있지?

말하자면 일종의 부수효과이다. new Foo() 는 결국 새 객체를 다른 객체와 연결하기 위한 간접적인 우회 방법인 셈이다.
더 직접적인 방법에는 `Object.create()` 가 있다.

##### 이름에는 무엇이 들어 있을까?

자바스크립트는 어떤 객체 ('클래스') 에서 다른 객체 ('인스턴스') 로 복사하는 것이 아니라 단순히 연결한다.
[[Prototype]] 체계를 다른 말로 프로토타입 상속이라고 하며 흔히 클래스 상속의 동적버전이라고 말한다. 클래스 지향 세계에서 지극히 일반적인 상속 개념을 동적 스크립트 언어에 맞게 그 의미를 조금 변형한 것이다.

상속이라는 단어는 이미 많은 선례를 남긴 강력한 단어이다. 자바스크립트는 동작 방식이 정 반대인 터리 구태어 '프로토타입' 을 앞에 붙여 구별하려 했지만 의도와 다르게 20년동안 혼란만 낳았다.

상속에 프로토타입을 붙여 의미를 뒤집어 버린것은 오히려 자바스크립트 체계를 이해하는데 더 어려움을 준다.
상속은 기본적으로 복사를 수반하지만, 자바스크립트는 객체 프로퍼티를 복사하지 않는다. 대신 두 객체에 링크를 걸어두고 한쪽이 다른 쪽의 프로퍼티/함수에 접근할 수 있게 위임한다. 위임이 자바스크립트 객체-연결 체계를 더 정확하게 나타낸 용어이다.

#### 5.2.2 생성자

앞의 예제로 돌아가자.

```javascript
function Foo() {

}
var a = new Foo();
```

위 코드에서 Foo 가 마치 클래스인것 처럼 보여지는 이유는 new 키워드와 진짜 클래스가 생성될 때 생성자가 호출 되듯이 `Foo()` 함수가 실제로 호출됐기 때문이다.

다음 코드를 보자

```javascript
function Foo() {

}
Foo.prototype.constructor === Foo; // true
var a = new Foo();
a.constructor === Foo; // true
```

`Foo.prototype` 객체에는 기본적으로 (첫째줄 함수 선언시) 열거 불가한 공용 프로퍼티 `.constructor` 가 셋팅되는데, 이는 객체 생성과 관련된 함수 `Foo` 를 다시 참조하기 위한 프로퍼티이다. 마찬가지로 new 키워드로 생성한 객체 a도 `.constructor` 프로퍼티로 이 `Foo` 함수를 가르킬 수 있다.
> 하지만 a 가 constructor 프로퍼티를 소유하고 있는것은 아니다. 프로토타입 체인을 통해 Foo.prototype 의 constructor 프로퍼티를 참조하는 것

자바스크립트는 관례로 '클래스' 를 대문자로 시작하니 `foo` 가 아닌 `Foo` 로 표기한건 처음부터 클래스를 의도했다는 단서다.
> es6 부턴 class 문법이 생겼으니 이제 이렇게 안쓰려나?

##### 생성자냐 호출이냐?

위 예제에서 Foo 를 호출하는 순간 객체가 툭 하게 생겼으니 생성자로 믿고 싶을것이다. 하지만 Foo 는 단순한 함수일 뿐이다. 함수는 결코 생성자가 아니지만 앞에 new 지시자를 붙여 호출하는 순간 이 함수는 '생성자 호출'을 한다. new 키워드가 일반 함수 호출을 도중에 가로채서 원래 수행할 작업 외에 객체 생성이라는 추가적인 작업을 지시하는 지시자다.

```javascript
function NothingSpecial() {
  console.log("신경쓰지마");
}

var a = new NothingSpecial();
a; // {}
```

`NothingSpecial` 은 평범한 함수이다. 그런데 이 함수를 new 로 호출하면서 객체를 생성하고 부수효과로 생성된 객체를 a 에 할당하는 '생성자 호출' 을 하게 되는것이다. 즉, 자바스크립트 함수 앞에 new 를 붙이면 모두 '생성자' 라고 할 수 있지만 new 뒤에 오는 함수는 단순한 함수일 뿐이다.

#### 5.2.3 체계

이 외에도 클래스를 흉내내기 위한 노력은 더 존재한다.

```javascript
function Foo(name) {
  this.name = name;
}

Foo.prototype.myName = function () {
  return this.name;
}

var a = new Foo("a");
var b = new Foo("b");

a.myName(); // "a"
b.myName(); // "b"
```

Foo.prototype 객체에 myName 이라는 프로퍼티(함수) 를 추가하고 a.myName 을 호출하면 마치 프로토타입의 프로퍼티가 a 라는 객체로 복사된것 처럼 보이지만 사실은 프로토타입 체인을 타고 올라가서 Foo.prototype.myName 에게 실행을 위임한것이다.

##### 돌아온 '생성자'

앞서 .constructor 프로퍼티를 소유하고 있는것이 아닌 해당 객체의 prototype 객체가 갖고 있는 .constructor 프로퍼티에게 위임된다고 했다. 이는 편리해 보이지만 .constructor 프로퍼티가 "~에 의해 생성됨" 을 의미한다고 가정한다면 큰 혼란에 빠지게 된다. 몇가지 경우의 수를 보자

1. `.constructor` 프로퍼티는 기본으로 선언된 함수에 의해 생성된 프로토타입 객체에만 존재한다. 새로운 객체를 생성한뒤 기본 프로토타입 객체 레퍼런스를 변경하면 변경된 레퍼런스에 옮겨 붙진 않는다.

```javascript
function Foo() {
  //
}
Foo.prototype = {
  //
};

var a1 = new Foo();
a1.constructor === Foo; // false
a1.constructor === Object; // true
```
