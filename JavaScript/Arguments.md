# Arguments



## 1. Javascript 에서 arguments.callee 를 사용하지 말아야 하는 이유



과거 자바스크립트에서는 함수 표현식에 이름을 붙이는게 가능하지 않았다. 

```javascript
// This snippet will work:
 function factorial(n) {
     return (!(n>1))? 1 : factorial(n-1)*n;
 }
 [1,2,3,4,5].map(factorial);


 // But this snippet will not:
 [1,2,3,4,5].map(function(n) {
     return (!(n>1))? 1 : /* what goes here? */ (n-1)*n;
 });
```

그래서 `arguments.callee` 를 통하여 재귀함수를 구현했는데

```javascript
[1,2,3,4,5].map(function(n) {
     return (!(n>1))? 1 : arguments.callee(n-1)*n;
 });
```

이는 몇가지 문제점을 갖고있다.

우선 `callee` 를 사용하게 되면 해당함수는 런타임시 결정되므로 인라인 최적화를 할 수 가 없어 성능에 악영향을 끼친다.

또한 해당 함수의 calling context에 의존하게 되기 때문에 캡슐화를 해친다.

```javascript
function foo() {
    arguments.callee; // 이 함수를 가리킨다.
    arguments.callee.caller; // 이 함수를 호출한 부모함수를 가리킨다.
}

function bigLoop() {
    for(var i = 0; i < 100000; i++) {
        foo(); // 원래 인라인 돼야 하는디...
    }
}
```



또한 아래 예제에서 볼 수 있듯이 `this` 의 바인딩이 변경된다.

```javascript
var global = this;
var sillyFunction = function (recursed) {
    if (!recursed)
        return arguments.callee(true);
    if (this !== global)
        alert("This is: " + this); // [object Arguments]
    else
        alert("This is the global");
}
sillyFunction();
```



es3 부터 함수 표현식에 이름을 주어 이를 해결할 수 있다.

```javascript
[1,2,3,4,5].map(function factorial(n) {
     return (!(n>1))? 1 : factorial(n-1)*n;
 });
```



```javascript
var global = this;
var sillyFunction = function alertThis(recursed) {
    if (!recursed)
        return alertThis(true);
    if (this !== global)
        alert("This is: " + this); 
    else
        alert("This is the global"); // this : global
}
sillyFunction();
```

위 코드는 성능을 해치지도, `this` 와 네임스페이스를 망치지도 않으면서 어디서든 이 함수를 호출할 수 있게 해준다.

추가로 `arguments object` 에 대한 접근은 굉장히 비싼 연산이다.



참고

https://stackoverflow.com/questions/103598/why-was-the-arguments-callee-caller-property-deprecated-in-javascript

http://bonsaiden.github.io/JavaScript-Garden/ko/#function.arguments