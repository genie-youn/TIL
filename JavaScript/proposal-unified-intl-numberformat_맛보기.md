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

#### Proposal Unified Intl Numberformat

`Proposal Unified Intl Numberformat`는 위에서 소개했던 `Intl.NumberFormat` 에 추가적으로 m/s, m<sup>3</sup> 등 과 같은 측정 단위에 대한 표현이나 숫자에 대한 과학,공학,회계적 표현을 가능하게 하는 기능을 추가할 것을 제안합니다.

> 자세한 논의 내용은 이 [스레드](https://github.com/tc39/ecma402/issues/215) 를 참고하세요.

`Intl`의 생성자에 여러 기능을 포함시켜 복잡하게 만드는 것 보단 `Intl.NumberFormat` 의 스펙을 변경하여 통합하는 방식으로 더 쉽게 이 추가 기능을 지원할 것을 제안합니다.

#### Units
측정 단위는 다음과 같이 표현할 수 있습니다.

```JavaScript
(299792458).toLocaleString("en-US", {
    style: "unit",
    unit: "meter-per-second",
    unitDisplay: "short"
});
// ==> "299,792,458 m/s"
```

#### Scientific and Compact Notation
과학 표기 단위나 간결하게 표현하기 위해서는 새로 추가된 옵션인 `notation` 을 추가하면 됩니다.

```JavaScript
(987654321).toLocaleString("en-US", {
    notation: "scientific"
});
// ==> 9.877E8

(987654321).toLocaleString("en-US", {
    notation: "engineering"
});
// ==> 987.7E6

(987654321).toLocaleString("en-US", {
    notation: "compact",
    compactDisplay: "long"
});
// ==> 987.7 million
```

- `notation` 은 다음중 하나의 값을 받을 수 있습니다. "standard" (default), "scientific", "engineering", "compact"
- `compactDisplay` 은 `notation` 이 "compact" 일 경우멘 사용할 수 있으며 "short" (default) 또는 "long" 을 받을 수 있습니다.

반올림 관련 설정 (최소/최대 정수/소수 자릿수) 는 선택한 표기법에 따라 숫자를 변환 한 후에 적용이 됩니다.

`notation` 이 "compact" 이고 별도의 반올림 옵션이 주어지지 않으면, 정수에 가장 근접하게 반올림하되, 두개의 유효한 숫자를 유지하는 특수한 반올림 전략이 사용됩니다.

예를들어 123.4K는 123K로 반올림되고, 1.234K는 1.2K로 반올림 됩니다.

만약 `notation`이 "compact" 이고 별도의 반올림 설정이 없다면 이 반올림 전략이 사용되고 있는지 `resolvedOptions` 을 통해 확인 할 수 있습니다.

`notation` 은 다른 옵션과 조합되어 사용될 수 있습니다.

```JavaScript
(299792458).toLocaleString("en-US", {
    notation: "scientific",
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
    style: "unit",
    unit: "meter-per-second"
});
// ==> 3.00E8 m/s
```
