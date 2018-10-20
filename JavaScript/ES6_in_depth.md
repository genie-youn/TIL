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

\```javascript 

var zeroesForeverIterator = {  

​    [Symbol.iterator]: function () {  

​        return this;  

​    },

​    next: function () {  

​        return {done: false, value: 0};  

​    } 

};

\```



\```javascript

for (VAR of ITERABLE) {  

​    STATEMENTS 

}

\```



\```javascript 

var $iterator = ITERABLE[Symbol.iterator]();  

var $result = $iterator.next();  

while (!$result.done) {  

​    VAR = $result.value; 

​    STATEMENTS 

​    $result = $iterator.next(); 

}

\``` 