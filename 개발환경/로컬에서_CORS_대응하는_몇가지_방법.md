# 로컬에서 CORS를 대응하는 몇가지 방법

## 들어가며
프론트 엔드 개발을 하다보면 번거로운 일중 하나가 CORS에 관련된 것이다.

특히 `80`이나 `443` 포트 외에 로컬에서 주로 실행되는 `8080` 과 같은 포트들에 대해서는 열려있지 않는 경우가 많은데, 만약 기존 서버군이 많게 분산되어 있다면 발견될 때마다 요청하는 것도 번거로운 일일 것이다.

그래서 로컬에서 CORS를 대응할 수 있는 방법엔 무엇이 있을지 몇가지 정리해두려 한다.

> [CORS](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)에 대한 자세한 내용은 다음을 참고.

## charles

https://www.charlesproxy.com/

charles 는 HTTP 요청을 프록시하거나 모니터링 할 수 있는 툴이다. SSL 인증서를 넣어서 HTTPS 요청을 모니터링 하거나 요청,응답의 헤더, 바디등 다양한 정보를 변조하며 테스트 해볼 수 있다.

charles 로 로컬에서 CORS를 대응하려면 `rewrite` 를 통해 서버 응답의 헤더를 변조하면 된다.

```
Access-Control-Allow-Origin: https://local.myapp.com:8080
Access-Control-Allow-Methods: POST, GET, DELETE, PUT, OPTIONS
Access-Control-Allow-Headers: X-MY-CUSTOM-HEADER, Content-Type
```

charles 는 웹 개발할때 많은 유용한 기능이 있기에 꼭 CORS를 위해서가 아니더라도 디버깅이나 테스트하는데 유용하므로 친해지도록 하자.

한가지 흠이 있다면, charles 의 라이센스가 유료라는 것인데, 회사에서 사주지 않는 다면 두번째 선택지가 있다.

## Moesif Orign & CORS Changer

두번째는 Moesif Orign & CORS Changer 라는 크롬 익스텐션이다. 동일하기 Response의 CORS 관련 헤더를 조작할 수 있다.

다만, HTTPS 환경에서 적용하려면 `ADVANCED SETTINGS` 에서 도메인을 직접 주어야 하는데 (기본은 whildcard로 적용되지만 https는 제외되는것 같다.) 회사 도메인의 메일을 적어주어야 `ADVANCED SETTINGS` 에 진입할수 있다..

다행히 스팸메일 같은게 오거나 하지 않으니 편하게 추가해주도록 하자.

> 설치는 다음 [링크](https://chrome.google.com/webstore/detail/moesif-orign-cors-changer/digfbfaphojjndkpccljibejjbppifbc) 를 참고
