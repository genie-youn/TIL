# Content-Security-Policy

콘텐츠 보안 정책(Content Security Policy, CSP)은 신뢰된 웹 페이지 콘텍스트에서 악의적인 콘텐츠를 실행하게 하는 사이트 간 스크립팅(XSS), 클릭재킹, 그리고 기타 코드 인젝션 공격을 예방하기 위해 도입된 컴퓨터 보안 표준이다.
웹 애플리케이션 보안의 W3C 워킹 그룹의 후보 권고안이며 현대의 웹 브라우저에 폭넓게 지원된다.
CSP는 웹사이트 소유자들이 승인된 콘텐츠 오리진(origin)을 선언할 수 있게 하는 표준 방식을 제공하며, 이를 통해 해당 웹사이트들로부터 브라우저들이 자바스크립트, CSS, 프레임, 웹 워커, 글꼴, 이미지, 그리고 자바 애플릿, 액티브X, 오디오 및 비디오 파일, 그리고 기타 HTML5 기능들을 사용할 수 있게 허용이 가능해진다.

xss는 기본적으로 애플리케이션의 스크립트와 제3자가 악의적으로 주입한 스크립트를 구분하지 못한다는데에 있다.
브라우저는 스크립트의 출처에 관계없이 페이지에서 요청하는 스크립트라면 다운로드하여 실행한다.

CSP는 신뢰할 수 있는 콘텐츠 허용 목록을 생성할 수 있게 한다.
`Content-Security-Policy` HTTP 헤더를 정의하고 브라우저에는 이런 소스에서 받은 리소스만 실행하거나 렌더링할 것을 선언한다. 

```
Content-Security-Policy: script-src 'self' https://apis.google.com
```

처럼 선언한다면 스크립트의 출처가 현재 페이지거나 `https://apis.google.com` 일 경우에만 스크립트의 실행을 허용할 것이다.

스크립트가 보안에 가장 큰 위협을 주지만 스크립트가 아닌 다양한 리소스에 대하여 신뢰할 출처를 선언할 수 있다.

```
- `base-uri` 는 페이지의 <base> 요소에 나타날 수 있는 URL을 제한한다.
- `child-src` 는 frame 내에서 실행될 수 있는 컨텐츠의 출처 URL을 제한한다. 예를들어 `child-src https://youtube.com`을 선언하면 유튜브의 컨텐츠는 frame 내에서 실행할 수 있지만, 다른 origin의 리소스는 실행되지 않는다. `frame-src`는 deprecated 되었으며 `child-src` 를 사용할 것을 권장한다.
- `connect-src` 는 (`XHR`, `WebSockets` 및 `EventSource`를 통해) 연결할 수 있는 출처를 제한한다.
- `font-src` 는 웹 글꼴을 제공할 수 있는 출처를 제한한다.
- `form-action` 은 <form> 태그에서의 제출을 위해 유효한 엔드포인트를 제한한다.
- `frame-ancestors` 는 현재 페이지를 삽입할 수 있는 소스를 제한한다. 이 지시문은 <frame>, <iframe>, <embed> 및 <applet> 태그에 적용된다. 이 지시문은 <meta> 태그에서 사용할 수 없고 HTML 이외의 리소스에만 적용된다.
- `img-src` 는 이미지를 로드할 수 있는 출처를 정의한다.
- `media-src` 는 동영상과 오디오를 제공하도록 허용되는 출처를 제한한다.
- `object-src` 는 플래시와 기타 플러그인에 대한 제어를 허용한다.
- `plugin-types` 는 페이지가 호출할 수 있는 플러그인의 종류를 제한한다.
- `report-uri` 은 콘텐츠 보안 정책 위반 시 브라우저가 보고서를 보낼 URL을 지정한다. <meta> 태그에서는 이 지시문을 사용할 수 없다.
- `style-src` 는 script-src에서 스타일시트에 해당한다.
- `upgrade-insecure-requests` 는 사용자 에이전트에 URL 스킴을 HTTP를 HTTPS로 변경하도록 지시한다. 다시 작성해야할 많은 오래된 URL을 가지고 있는 웹사이트를 위한것이다.
```
  
세미콜론으로 구분하여 작성하면 된다.
  
```
Content-Security-Policy: default-src https://cdn.example.net; child-src 'none'; object-src 'none'
```

스크립트의 출처를 제한하더라도 인라인스크립트를 삽입하여 공격하는 경우에는 막을 수 없다.
그래서 CSP를 설정하면 모든 인라인 스크립트는 유해한것으로 간주된다.
  
인라인스크립트를 꼭 사용해야 한다면 허용할 출처에 `unsafe-inline` 를 추가하면 된다.
  
```
Content-Security-Policy: script-src 'self' `unsafe-inline` https://apis.google.com
```  

하지만 인라인 스크립트는 가장 큰 보안 위협이고, 이를 제한하는건 CSP가 제공하는 가장 큰 강점이다. 모든 인라인 코드를 걷어내는 것은 쉽지 않은 작업이지만 고려해볼 가치가 있다.
그래도 꼭꼭 사용해야겠다면 스크립트 태그에 nonce 속성을 사용할것.
  
Eval도 동일한 맥락에서 유해한것으로 간주된다.
  

사용사례

세가지 외부 서비스의 위젯을 사용한다고 가정한다.

구글+ 버튼

```
Google의 +1 버튼은 https://apis.google.com에서 받은 스크립트를 포함하고 https://plusone.google.com에서 받은 <iframe>을 삽입합니다. 버튼을 삽입하려면 이런 두 가지 출처를 모두 포함하는 정책이 필요합니다. 최소 정책은 다음과 같습니다. script-src https://apis.google.com; child-src https://plusone.google.com. Google이 제공하는 자바스크립트의 스니펫이 외부 자바스크립트 파일로 추출되는지도 확인해야 합니다. child-src를 사용하는 기존 정책이 있는 경우 이를 child-src로 변경해야 합니다.
```

페이스북 Like 버튼

```
여러 가지 구현 옵션이 있습니다. <iframe> 버전이 사이트 나머지 부분으로부터 안전하게 샌드박싱되어 있으므로 이 버전을 고수하는 것이 좋습니다. child-src https://facebook.com 지시문이 올바른 기능을 수행해야 합니다. 참고로, Facebook이 제공하는 <iframe> 코드는 기본적으로 상대 URL인 //facebook.com을 로드합니다. 명시적으로 HTTPS를 지정하도록 https://facebook.com으로 변경하세요. 꼭 그럴 필요가 없는데도 HTTP를 사용할 이유는 없습니다.
```  

witter의 Tweet 버튼

```
Twitter의 Tweet 버튼은 스크립트와 프레임에 대한 액세스에 의존하며, 둘 다 https://platform.twitter.com에서 호스팅됩니다. (Twitter 역시 기본적으로 상대 URL을 제공합니다. URL을 로컬에서 복사해 붙여넣을 때는 HTTPS를 지정하도록 코드를 편집하세요.) Twitter가 제공하는 자바스크립트 스니펫을 외부 자바스크립트 파일로 이동해 넣는 한, 전부 script-src https://platform.twitter.com; child-src https://platform.twitter.com으로 설정될 것입니다.

다른 플랫폼에도 비슷한 요구사항이 있으며, 역시 비슷한 방법으로 해결할 수 있습니다. 그냥 default-src를 'none'으로 설정하고 콘솔을 살펴보면서 위젯이 작동하도록 하려면 어떤 리소스를 사용해야 할지 확인하는 방법을 추천합니다.
```

위 세가지 위젯을 사용하려면 다음과 같이 CSP를 구성하면 된다.

```
script-src https://apis.google.com https://platform.twitter.com; child-src https://plusone.google.com https://facebook.com https://platform.twitter.com
```

  
`child-src` 와 `frame-ancestors`의 차이는 뭐야
`upgrade-insecure-requests`는 뭐야

---
https://ko.wikipedia.org/wiki/%EC%BD%98%ED%85%90%EC%B8%A0_%EB%B3%B4%EC%95%88_%EC%A0%95%EC%B1%85#cite_note-Stamm_2009-1
https://developers.google.com/web/fundamentals/security/csp?hl=ko
