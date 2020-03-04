#  proposal-unified-intl-numberformat

## 들어가며
이번에 `stage4` 에 합류한 `Unified Number Format` 에 대해 알아보자.

## proposal-unified-intl-numberformat ?
이번 tc39 미팅에서 `stage4` 에 추가된 `proposal-unified-intl-numberformat` 는 `Intl.NumberFormat` 에 측정 단위 표현와 간략한 십진법 표현 그리고 기타 현지화된 숫자 포맷팅 표현에 관한 기능을 추가한다.

### Background / Motivation
ECMA-402 (자바스크립트 Intl 표준 라이브러리) 에 숫자 포맷팅에 관련된 기능을 추가하자는 많은 의견이 있었다.
이러한 기능들은 개별 사용자와 구글에게 모두 중요하다. i18n 을 제대로 지원하기 위해서는 큰 규모의 로케일 데이터를 함께 전송해야 하기 때문에, 이러한 기능들을 JavaScript의 API로 노출하는것은 전송해야 할 데이터를 줄일 수 있고, 이는 i18n 지원에 필요한 진입장벽을 낮출 것이다.

Rather than complicate Intl with more constructors with heavilly overlapping functionality, this proposal is to restructure the spec of Intl.NumberFormat to make it more easilly support additional features in a "unified" way.


Additional background: prior discussion



> 제안의 자세한 내용은 [다음](https://github.com/tc39/proposal-unified-intl-numberformat)을 참고한다.
