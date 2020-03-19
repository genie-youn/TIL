# proposal-unified-intl-numberformat 맛보기

## 들어가며
얼마 전 [tc39의 2월 미팅](https://github.com/tc39/notes/blob/master/meetings/2020-02/february-5.md#unified-number-format-for-stage-4)에서 `proposal-unified-intl-numberformat`가 스테이지4로 승격되었다.

`proposal-unified-intl-numberformat`는 `Intl.NumberFormat`에 측정 단위와 회계적 표현 등 숫자를 형식화 하는 기능을 제안한다.

이 아티클에선 `Intl`과 제안하고 있는 `Unified Numberformat`에 관한 MDN을 번역하고, 한국어 예제를 간략하게 추가합니다.

### Intl?
`Intl` 객체는 ECMAScript의 국제화를 위한 API [ECMA-402](https://www.ecma-international.org/ecma-402/1.0/#sec-8) 로 정의된 각 지역의 언어에 맞는 문자열 비교,숫자와 날짜 및 시간에 대한 형식화를 제공하는 객체로 다음과 같은 생성자를 포함하는 네임스페이스이다.

#### Intl.Collator
각 언어에 맞는 문자열 비교에 대한 기능을 제공하는 `Collator` 객체를 생성하는 생성자이다.

`Collator` 객체는 두개의 문자열을 비교하는 `compare()` 와 객체의 설정 정보를 반환하는 `resolvedOptions()` 두개의 메서드를 가지고 있다.

독일어(de) 와 스웨덴어(sv) 는 동일하게 ä 를 사용하지만 독일어는 a 다음에, 스페인어에선 z 다음에 정렬된다.

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

> 역주: `Array.prototype.sort` 는 인자로 `compareFunction`을 따로 지정하지 않으면 각 문자의 유니코드 포인트 값에 따라 정렬한다. 따라서 한국어는 별도의 인자 없이도 정렬이 가능하다.
```JavaScript
console.log(["오해영","박도경","한태진","에릭","서현진","전혜빈"].sort());
// ["박도경", "서현진", "에릭", "오해영", "전혜빈", "한태진"]
```

> Collator에 관한 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Collator) 를 참고

#### Intl.DateTimeFormat
각 언어에 맞는 시간과 날짜 서식을 적용할 수 있는 객체의 생성자이다.

```Javascript
const date = new Date(Date.UTC(2012, 11, 20, 3, 0, 0));
// Results below assume UTC timezone - your results may vary

console.log(new Intl.DateTimeFormat('en-US').format(date));
// "12/20/2012"

console.log(new Intl.DateTimeFormat('en-GB').format(date));
// "20/12/2012"

console.log(new Intl.DateTimeFormat('ko-KR').format(date));
// "2012. 12. 20."
```

> DateTimeFormat에 관한 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat)를 참고

#### Intl.ListFormat
주어진 리스트를 각 언어에 맞게 문자열로 변경할 수 있는 객체의 생성자이다.

```Javascript
const vehicles = ['Motorcycle', 'Bus', 'Car'];

const formatter = new Intl.ListFormat('en', { style: 'long', type: 'conjunction' });
console.log(formatter.format(vehicles));
// "Motorcycle, Bus, and Car"

const formatter2 = new Intl.ListFormat('ko', { style: 'long', type: 'conjunction' });
console.log(formatter.format(vehicles));
// "Motorcycle, Bus, 및 Car"

```

> ListFormat에 관한 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ListFormat)를 참고

#### Intl.NumberFormat
각 언어에 맞는 숫자 서식을 적용할 수 있는 객체의 생성자이다.

```Javascript
const number = 123456.789;

console.log(new Intl.NumberFormat('de-DE', { style: 'currency', currency: 'EUR' }).format(number));
// "123.456,79 €"

console.log(new Intl.NumberFormat('ja-JP', { style: 'currency', currency: 'JPY' }).format(number));
// "￥123,457"

console.log(new Intl.NumberFormat('ko-KR', { style: 'currency', currency: 'KRW' }).format(number));
// ₩123,457

console.log(new Intl.NumberFormat('ko-KR', { style: 'percent' }).format(1.45));
// 145%

```

> NumberFormat에 관한 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat)를 참고

#### Intl.RelativeTimeFormat
각 언어에 맞는 상대 시간 서식을 적용할 수 있는 객체의 생성자이다.

```javascript
const rtf1 = new Intl.RelativeTimeFormat('ko', { style: 'narrow' });

console.log(rtf1.format(3, 'quarter'));
// 3분기 후

console.log(rtf1.format(-1, 'day'));
// 1일 전
```

> RelativeTimeFormat에 관한 자세한 내용은 [레퍼런스](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RelativeTimeFormat)를 참고

#### Proposal Unified Intl Numberformat

`Unified Numberformat`는 위에서 소개했던 `Intl.NumberFormat` 에 추가적으로 m/s, m<sup>3</sup> 등 과 같은 측정 단위에 대한 표현이나 숫자에 대한 과학,공학,회계적 표현을 가능하게 하는 기능을 추가할 것을 제안한다.

> 자세한 논의 내용은 이 [스레드](https://github.com/tc39/ecma402/issues/215) 를 참.

`Intl`의 생성자에 여러 기능을 포함시켜 복잡하게 만드는 것 보단 `Intl.NumberFormat` 의 스펙을 변경하여 통합하는 방식으로 더 쉽게 이 추가 기능을 지원할 것을 제안한다.

#### Units
측정 단위는 다음과 같이 표현할 수 있다.

```JavaScript
console.log(new Intl.NumberFormat("en-US",  {
    style: 'unit',
    unit: "centimeter-per-second",
    unitDisplay: "short"
}).format(299792458));
// 299,792,458 cm/s

console.log(new Intl.NumberFormat("ko-KR",  {
    style: 'unit',
    unit: "centimeter-per-second",
    unitDisplay: "short"
}).format(299792458));
// 한국은 표기단위를 붙여서 표현
// 299,792,458cm/s

console.log(new Intl.NumberFormat("ko-KR",  {
    style: 'unit',
    unit: "liter",
    unitDisplay: "short"
}).format(50));
// 50L

console.log(new Intl.NumberFormat("ko-KR",  {
    style: 'unit',
    unit: "liter",
    unitDisplay: "long"
}).format(50));
// 50리터

console.log(new Intl.NumberFormat("ko-KR",  {
    style: 'unit',
    unit: "centimeter-per-second",
    unitDisplay: "short"
}).format(5));
// 5cm/s

console.log(new Intl.NumberFormat("ko-KR",  {
    style: 'unit',
    unit: "centimeter-per-second",
    unitDisplay: "long"
}).format(5));
// 초속 5센티미터
```

#### Scientific and Compact Notation
과학, 공학적 표현이나 간결하게 표현하기 위해서는 새로 추가된 옵션인 `notation` 을 추가하면 된다.

```JavaScript
console.log(new Intl.NumberFormat('ko-KR', { notation: "scientific" }).format(987654321));
// 9.877E8

console.log(new Intl.NumberFormat('ko-KR', { notation: "engineering" }).format(987654321));
// 987.654E6

console.log(new Intl.NumberFormat('ko-KR', { notation: "compact" }).format(987654321));
// 9.9억

console.log(new Intl.NumberFormat('ko-KR', { notation: "compact" }).format(9876543));
// 998만
```

- `notation` 은 다음중 하나의 값을 받을 수 있다. "standard" (default), "scientific", "engineering", "compact"
- `compactDisplay` 은 `notation` 이 "compact" 일 경우멘 사용할 수 있으며 "short" (default) 또는 "long" 을 받을 수 있다.

반올림 관련 설정 (최소/최대 정수/소수 자릿수) 는 선택한 표기법에 따라 숫자를 변환 한 후에 적용이 된다.

`notation` 이 "compact" 이고 별도의 반올림 옵션이 주어지지 않으면, 정수에 가장 근접하게 반올림하되, 두개의 유효한 숫자를 유지하는 특수한 반올림 전략이 사용된다.

예를들어 123.4K는 123K로 반올림되고, 1.234K는 1.2K로 반올림 된다.

만약 `notation`이 "compact" 이고 별도의 반올림 설정이 없다면 이 반올림 전략이 사용되고 있는지 `resolvedOptions` 을 통해 확인 할 수 있다.

또한 `notation` 은 다음과 같이 다른 옵션과 조합되어 사용될 수 있다.

```JavaScript
console.log(new Intl.NumberFormat("ko-KR",  {
    notation: "scientific",
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
    style: 'unit',
    unit: "centimeter-per-second",
    unitDisplay: "short"
}).format(299792458));
// 3.00E8cm/s
```

#### Sign Display

부호는 양수에 대해 표현될 수 있다.
```Javascript
(55).toLocaleString("en-US", {
    signDisplay: "always"
});
// ==> +55
```

통화 회계 부호 표시도 새로운 옵션을 통해 지원된다. 많은 지역에서 마이너스 부호를 붙이는 대신에 괄호로 숫자를 감싸는 방법으로 음수의 통화 회계 숫자를 표현한다.

```Javascript
(-100).toLocaleString("bn", {
    style: "currency",
    currency: "EUR",
    currencySign: "accounting"
});
// ==> (১০০.০০€)
```

`signDisplay`: "auto" (default), "always", "never", "exceptZero"
`currencySign`: "standard" (default), "accounting"

"accounting" 은 통화를 나타내는 값에 회계적 표현을 가능하게 한다. 자세한 내용을 아래 예제를 참고.

`signDisplay`

signDisplay | -1  | -0  | 0   | 1   | NaN
----------- | --- | --- | --- | --- | ---
auto        | -1  | -0  | 0   | 1   | NaN
always      | -1  | -0  | +0  | +1  | +NaN
never       | 1   | 0   | 0   | 1   | NaN
exceptZero  | -1  | 0   | 0   | +1  | NaN

`currencySign`

signDisplay | -1      | -0      | 0       | 1      | NaN
----------- | --------| ------- | ------- | ------ | ----
auto        | ($1.00) | ($0.00) | $0.00   | $1.00  | $NaN
always      | ($1.00) | ($0.00) | +$0.00  | +$1.00 | +$NaN
never       | $1.00   | $0.00   | $0.00   | $1.00  | $NaN
exceptZero  | ($1.00) |  $0.00  | $0.00   | +$1.00 | $NaN

물론 다른 옵션과 함께 사용할 수 있다.

```JavaScript
(0.55).toLocaleString("en-US", {
    style: "percent",
    signDisplay: "exceptZero"
});
// ==> +55%
```

> 이외 제안의 자세한 내용은 [다음](https://github.com/tc39/proposal-unified-intl-numberformat)을 참고.

### 마치며
stage4에 합류하여 곧 표준 스펙으로 만나보게 될 `Unified Numberformat`에 대해 알아보았다.

사실 이 제안을 보고 `Intl`을 처음 알게 되었고, 글로벌 서비스를 위해 로케일 데이터에 대한 지원이 필요한 서비스라면 요긴하게 사용될 스펙일 것 같다.

`Intl`은 각 피쳐별로 다르긴 하지만 대부분의 모던 브라우저에서 사용 가능하며, `Unified Numberformat`의 경우 해당 제안을 낸게 구글이라 그런지.. 크롬에서는 77버전에 구현되었고 나머지 브라우저에선 다음 [폴리필](https://www.npmjs.com/package/@formatjs/intl-unified-numberformat)을 추가하면 사용가능하다.
