#  proposal-unified-intl-numberformat

## 들어가며
이번에 `stage4` 에 합류한 `Unified Number Format` 에 대해 알아보자.

## proposal-unified-intl-numberformat ?
이번 tc39 미팅에서 `stage4` 에 추가된 `proposal-unified-intl-numberformat` 는 `Intl.NumberFormat` 에 표기 단위 표현와 간략한 십진법 표현 그리고 기타 현지화된 숫자 포맷팅 표현에 관한 기능을 추가한다.

### Background / Motivation
ECMA-402 (자바스크립트 Intl 표준 라이브러리) 에 숫자 포맷팅에 관련된 기능을 추가하자는 많은 의견이 있었다.
이러한 기능들은 개별 사용자와 구글에게 모두 중요하다. i18n 을 제대로 지원하기 위해서는 큰 규모의 로케일 데이터를 함께 전송해야 하기 때문에, 이러한 기능들을 JavaScript의 API로 노출하는것은 전송해야 할 데이터를 줄일 수 있고, 이는 i18n 지원에 필요한 진입장벽을 낮출 것이다.

`Intl`의 생성자에 여러 기능을 포함시켜 복잡하게 만드는 것 보단 `Intl.NumberFormat` 의 스펙을 변경하여 통합하는 방식으로 더 쉽게 이 추가 기능을 지원할 것을 제안한다.

> 자세한 논의 내용은 이 [스레드](https://github.com/tc39/ecma402/issues/215) 를 참고하면 된다.

## I. Units
표기 단위는 다음과 같이 포맷팅 할 수 있다.

```JavaScript
(299792458).toLocaleString("en-US", {
    style: "unit",
    unit: "meter-per-second",
    unitDisplay: "short"
});
// ==> "299,792,458 m/s"
```

## II. Scientific and Compact Notation
과학 표기 단위나 간결하게 표현하기 위해서는 새로 추가된 옵션인 `notation` 을 추가하면 된다.

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

- `notation` 은 다음중 하나의 값을 받을 수 있다. "standard" (default), "scientific", "engineering", "compact"
- `compactDisplay` 은 `notation` 이 "compact" 일 경우멘 사용할 수 있으며 "short" (default) 또는 "long" 을 받을 수 있다

반올림 관련 설정 (최소/최대 정수/소수 자릿수) 는 선택한 표기법에 따라 숫자를 변환 한 후에 적용이 된다.

`notation` 이 "compact" 이고 별도의 반올림 옵션이 주어지지 않으면, 정수에 가장 근접하게 반올림하되, 두개의 유효한 숫자를 유지하는 특수한 반올림 전략이 사용된다.

예를들어 123.4K는 123K로 반올림되고, 1.234K는 1.2K로 반올림된다.

만약 `notation`이 "compact" 이고 별도의 반올림 설정이 없다면 이 반올림 전략이 사용되고 있는지 `resolvedOptions` 을 통해 확인 할 수 있다.

`notation` 은 다른 옵션과 조합되어 사용될 수 있다.

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

## III. Sign Display


> 제안의 자세한 내용은 [다음](https://github.com/tc39/proposal-unified-intl-numberformat)을 참고한다.
