# proposal-unified-intl-numberformat 맛보기

## 들어가며
이번 tc39 미팅에서 `stage4` 로 승격된 `proposal-unified-intl-numberformat` 는 `Intl.NumberFormat` 에 **현지화된** 측정 단위, 회계 통화 표현등 숫자를 포맷팅 하는 기능을 추가하는 것을 제안합니다.

### Intl?
`Intl` 객체는 ECMAScript의 국제화를 위한 API [ECMA-402](https://www.ecma-international.org/ecma-402/1.0/#sec-8) 로 정의된 각 지역의 언어에 맞는 문자,숫자,날짜,시간 비교를 제공하는 객체입니다.

Intl.Collator
콜레이터 생성자입니다. 콜레이터 객체는 각 언어에 맞는 문자열 비교를 가능하게 해줍니다.
Intl.DateTimeFormat
각 언어에 맞는 시간과 날짜 서식을 적용할 수 있는 객체의 생성자입니다.
Intl.ListFormat
각 언어에 맞는 목록 서식을 적용할 수 있는 객체의 생성자입니다.
Intl.NumberFormat
각 언어에 맞는 숫자 서식을 적용할 수 있는 객체의 생성자입니다.
Intl.PluralRules
각 언어에 맞는 복수형 규칙 및 단수 복수 구분 서식을 적용할 수 있는 객체의 생성자입니다.
Intl.RelativeTimeFormat
각 언어에 맞는 상대 시간 서식을 적용할 수 있는 객체의 생성자입니다.
