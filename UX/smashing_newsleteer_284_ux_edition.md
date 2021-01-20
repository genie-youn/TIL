# smashing newsletter #284 ux edition

## 1. Designing Better Landing Pages

Rob Hope 가 작성한 [Landing page hot tips](https://twitter.com/robhope/status/1265278107088347136) 트위터 스레드를 참고할것.

[Landingfolio](https://www.landingfolio.com/?offset=4) 와 [Land-Book](https://land-book.com/), [OnePageLove](https://onepagelove.com/inspiration/landing-page) 는 좋은 레퍼런스가 많다.

## 2. Getting Push Notifications Right

Stéphanie 가 작성한 [guide to push notifications](https://stephaniewalter.design/blog/the-ultimate-guide-to-not-fck-up-push-notifications/) 는 푸시 알림을 망치는 것을 멈추고, 사용자의 신뢰를 회복하는 방법에 대한 조언이 담겨있다.

사용자에게 알림을 통해 얻을 수 있는 이점을 이해할 수 있게 하고, 페이지가 로드 되었을 때가 아닌 적절한 맥락에서 권한을 요청해라. (사용자에게 이 알림이 왜 필요한지 이해시키고, 필요해 졌을때 권한을 요청하라는 말 같다.)

이커머스 서비스에서 고객이 구매 완료 후 배송 상태에 대한 알림을 받고 싶은지, 항공사가 항공편이 지연될때 이를 알림으로 받고 싶은지 등등

## 3. Alternatives To Spinners On The Web

애니메이션 스피너는 자주 볼 수 있지만, 과연 스피너가 사용자에게 의미가 있는 UX 일까?
이 질문에 대한 의견은 분분 하다. Simon Hearne 는 스피너를 "1990년대 웹의 유산" 이라고 이야기한다.
더 나은 대안은 없는걸까?


Simon Hearne의 결론은 간단하다. 모호한 스피너들을 진행률을 표시하도록 교체하고, 동적인 설명 텍스트를 추가해라.

만약 정말 스피너가 필요하다면, CSS와 SVG (GIF가 아닌) 를 통해 페이지의 브랜드와 테스크에 어울리게 구현해라. 작은 디테일이 큰 차이를 만들어낸다.

자세한 내용은 Simon Hearne의 [아티클](https://simonhearne.com/2020/alternatives-to-spinners/)을 참고해라.

## 4. Improving Security With Password Pasting

[영국의 국립 사이버 보안 센터](https://www.ncsc.gov.uk/blog-post/let-them-paste-passwords)가 권장하는 것 처럼 패스워드 붙여넣기를 허용하여 더 안전하게 만들어라.
패스워드를 붙여넣으면 서로 다르고 복잡한 패스워드를 쉽게 가질 수 있다.

비밀번호 붙여넣기를 비활성화 하는 것이 득보다 더 많은 해를 끼칠 수 있는 이유를 더 깊이 알고 싶다면 Troy Hunt가 쓴 [아티클](https://www.troyhunt.com/the-cobra-effect-that-is-disabling/) 을 참고해라.

비밀번호 입력에 관련된 더 많은 UX 가 궁금하다면 [Safe and Sound – How to Approach Password UX](https://www.toptal.com/designers/ux/password-ux)를 참고해라.

## 5. Custom Enter Key Action Labels

사소한 것일진 모르겠지만, 버츄얼 키보드를 타이핑할 때 즐거움을 유발하는것은 분명한데 사용자가 완료하려고 하는 작업에 따라 [더 많은 컨텍스트를 제공하는 엔터키](https://twitter.com/stefanjudis/status/1249958064041734144) 를 제공하는 것이다. `enterkeyhint` 속성을 사용하면 가능하다.

`done` 이면 엔터키가 체크표시로 바뀌고 `send` 면 쪽지 모양으로, 입력 폼의 다음으로 이동할 땐 `next`를, 다른 페이지로 이동할땐 `go`를 사용하자. `<input enterkeyhint="...">` 로 설정해주면 된다.

> 원글: https://mailchi.mp/smashingmagazine/smashing-newsletter-284-ux-edition
