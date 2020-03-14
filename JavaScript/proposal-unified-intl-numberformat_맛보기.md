# proposal-unified-intl-numberformat 맛보기

## 들어가며
이번 tc39 미팅에서 `stage4` 로 승격된 `proposal-unified-intl-numberformat` 는 `Intl.NumberFormat` 에 **현지화된** 측정 단위, 회계 통화 표현등 숫자를 형식화 하는 기능을 추가하는 것을 제안합니다.

### Intl?
`Intl` 객체는 ECMAScript의 국제화를 위한 API [ECMA-402](https://www.ecma-international.org/ecma-402/1.0/#sec-8) 로 정의된 각 지역의 언어에 맞는 문자열 비교,숫자와 날짜 및 시간에 대한 형식화를 제공하는 객체입니다.

#### Intl.Collator
각 언어에 맞는 문자열 비교에 대한 기능을 제공하는 `Collator` 객체를 생성하는 생성자입니다.

`Collator` 객체는 두개의 문자열을 비교하는 `compare()` 와 객체의 설정 정보를 반환하는 `resolvedOptions()` 두개의 메서드를 가지고 있습니다.

독일어(de) 와 스웨덴어(sv) 는 동일하게 ä 를 사용하지만 독일어는 a 다음에, 스페인어에선 z 다음에 정렬됩니다.

```JavaScript
function letterSort(lang, letters) {
  letters.sort(new Intl.Collator(lang).compare);
  return letters;
}

console.log(letterSort('de', ['a','z','ä']));
// ["a", "ä", "z"]

console.log(letterSort('sv', ['a','z','ä']));
// ["a", "z", "ä"]
```

> 역주: `Array.prototype.sort` 는 인자로 `compareFunction`을 따로 지정하지 않으면 각 문자의 유니코드 포인트 값에 따라 정렬합니다. 따라서 한국어는 별도의 인자 없이도 정렬이 가능합니다.
```JavaScript
console.log(["오해영","박도경","한태진","에릭","서현진","전혜빈"].sort());
// ["박도경", "서현진", "에릭", "오해영", "전혜빈", "한태진"]
```

> 자세한 내용을 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Collator) 를 참고하세요.

#### Intl.DateTimeFormat
각 언어에 맞는 시간과 날짜 서식을 적용할 수 있는 객체의 생성자입니다.

```Javascript
const date = new Date(Date.UTC(2012, 11, 20, 3, 0, 0));
// Results below assume UTC timezone - your results may vary

console.log(new Intl.DateTimeFormat('en-US').format(date));
// "12/20/2012"

console.log(new Intl.DateTimeFormat('en-GB').format(date));
// "20/12/2012"

console.log(new Intl.DateTimeFormat('ko-KR').format(date));
// "2012. 12. 20.""
```

> 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat)를 참고하세요.

#### Intl.ListFormat
각 언어에 맞는 목록 서식을 적용할 수 있는 객체의 생성자입니다.

```Javascript
const vehicles = ['Motorcycle', 'Bus', 'Car'];

const formatter = new Intl.ListFormat('en', { style: 'long', type: 'conjunction' });
console.log(formatter.format(vehicles));
// "Motorcycle, Bus, and Car"

const formatter2 = new Intl.ListFormat('ko', { style: 'long', type: 'conjunction' });
console.log(formatter.format(vehicles));
// "Motorcycle, Bus, 및 Car"

```

> 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ListFormat)를 참고하세요.

#### Intl.NumberFormat
각 언어에 맞는 숫자 서식을 적용할 수 있는 객체의 생성자입니다.

```Javascript
const number = 123456.789;

console.log(new Intl.NumberFormat('de-DE', { style: 'currency', currency: 'EUR' }).format(number));
// "123.456,79 €"

// the Japanese yen doesn't use a minor unit
console.log(new Intl.NumberFormat('ja-JP', { style: 'currency', currency: 'JPY' }).format(number));
// "￥123,457"

console.log(new Intl.NumberFormat('ko-KR', { style: 'currency', currency: 'KRW' }).format(number));
// ₩123,457
```

> 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat)를 참고하세요.

#### Intl.RelativeTimeFormat
각 언어에 맞는 상대 시간 서식을 적용할 수 있는 객체의 생성자입니다.

```javascript
const rtf1 = new Intl.RelativeTimeFormat('ko', { style: 'narrow' });

console.log(rtf1.format(3, 'quarter'));
// 3분기 후

console.log(rtf1.format(-1, 'day'));
// 1일 전
```

> 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RelativeTimeFormat)를 참고하세요.
