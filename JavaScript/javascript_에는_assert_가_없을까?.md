# Javascript 에는 assert 가 없을까?

요즘 TDD 스터디를 진행하고 있는데, 예제 코드를 보다가 자바스크립트에는 `assert` 키워드가 없나? 하는 생각이 들었다.
들어보지 못한것 같다.

우선 MDN에 검색해보니 `console.assert()` 만 보인다.
사용법은 다음과 같다.

```javascript
const errorMsg = 'the # is not even';
for (let number = 2; number <= 5; number +=1) {
  console.log('the # is' + number);
  console.assert(number % 2 === 0, {number, errorMsg})
}

// output:
// the # is 2
// the # is 3
// Assertion failed: {number: 3, errorMsg: "the # is not even"}
// the # is 4
// the # is 5
// Assertion failed: {number: 5, errorMsg: "the # is not even"}
```

인자로 주어진 평가식이 `false` 일 경우 메세지를 콘솔에 출력한다.
콘솔에 메세지가 노출될 뿐 에러를 반환하거나 하지 않으므로 찾던 키워드는 아닌것 같다. 빌트인 키워드는 존재하지 않는것 같으니 제안된 기능이 있는지 확인해보았다.

나와 동일한 생각을 한 사람이 제안한 [Proposal](https://es.discourse.group/t/error-assert/356) 이 존재하나 별 다른 논의가 진행되지 않은것으로 보인다.

그렇다면 인기있는 라이브러리가 있나?

2.6k 의 [power-assert](https://github.com/power-assert-js/power-assert) 가 있긴 한데 더 이상 업데이트 되지는 않아 보인다.

Node는 내장된 [라이브러리](https://nodejs.org/api/assert.html#assert_assert) 가 존재한다.

#### 정리하면
javascript 표준으로 정의된 `assert` 는 없으며, proposal 은 제안되었으나 별 다른 논의가 되질 않고 있다.
Node 는 빌트인 라이브러리를 지원하며 3rd 라이브러리로는 `power-assert` 가 유명하나 더 이상 개발이 안되는 것으로 보인다.

필요하다면 하나 만들어 쓰도록 하자.
