새로 배포한 코드가 특정 브라우저에서 동작하지 않는 일이 발생함.
`Array.prototype.flatMap`이 지원하지 않는 브라우저라 생긴일. 놀랍게도 아직 표준이 아니라 [proposal](https://tc39.github.io/proposal-flatMap/#sec-Array.prototype.flatMap) 단계.

지원하는 브라우저는 다음과 같다.
```
chrome 69+
Firefox 62+
Safari 12+
Opera 56+
```

한가지 이상한점은 Evergreen browser 인줄 알았던 크롬과 파폭이 최신버전이 아닌 경우가 있었는데,
왠지 모르겠네

무튼
https://github.com/es-shims/Array.prototype.flatMap 를 심었다.

```javascript
import flatMap from "array.prototype.flatmap";
flatMap.shim();
```
`​#shim()` 을 호출하면

```javascript
module.exports = function shimFlatMap() {
	var polyfill = getPolyfill();
	define(
		Array.prototype,
		{ flatMap: polyfill },
		{ flatMap: function () { return Array.prototype.flatMap !== polyfill; } }
	);
	return polyfill;
};

module.exports = function getPolyfill() {
	return Array.prototype.flatMap || implementation;
};
```
`#getPolyfill()` 을 호출했을때 `Array.prototype.flatMap` 이 존재하지 않으면, polyfill 을 반환하게 된다.
