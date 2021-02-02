# 들어가며

 매번 헷갈리는 개념인 Back/Forward Cache 에 대해서 정리하려 한다.

# Back/Forward Cache



https://docs.google.com/document/d/1mrgp7XzR16rd1xqFYOJgC1IP0NPLZFaRU5Ukj3-TlLw/edit#heading=h.d9wqdzopmdcf
https://docs.google.com/document/d/19u1jg934oNwE6IzZQ-vkeZLVqQiQWdZSkgTWkzp0CmU/edit
https://webkit.org/blog/427/webkit-page-cache-i-the-basics/
https://web.dev/bfcache/#optimize-your-pages-for-bfcache
https://developers.google.com/web/updates/2019/02/back-forward-cache
https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/1.5/Using_Firefox_1.5_caching

# 파이어폭스
페이지 네비게이션을 더 빠르게 하기 위해 자바스크립트의 상태를 포함한 페이지 전체를 메모리에 캐싱해둠. 그래서 앞으로가기 혹은 뒤로가기시에 네트워크에서 페이지를 로딩하지 않고 이 캐시를 사용하여 빠르게 복원함. 이 캐시는 단일 브라우저 세션에서 유효하며 브라우저를 종료하면 삭제됨.

다음 상황에선 캐싱하지 않음
- 페이지에 `unload` 혹은 `beforeunload` 핸들러가 있다면
- 페이지가 `cache-control: no-store` 로 설정되어 있을 경우
- HTTPS로 서빙되고 다음 중 하나의 설정이 되어있는 경우
  + "Cache-Control: no-cache"
  + "Pragma: no-cache"
  + "Expires: 0" 또는 "Expires" 가 header의 `Date` 보다 이전의 날짜로 설정되어 있을 경우 ("Cache-Control: max-age=" 이 지정되지 않는 한)
- 사용자가 페이지를 벗어났거나 어떠한 이유로 네트워크 요청이 펜딩되어 페이지가 정상적으로 로드되지 않았을 때
- 페이지가 IndexedDB 트랜잭션을 포함하는 경우
- 최상위 페이지가 프레임 (<iframe> 등) 을 포함하고 있는 경우

새로운 두개 브라우저 이벤트가 추가됨

파이어폭스 1.5에 추가된듯. 이 이벤트가 있어도 레거시 브라우저를 포함한 다른 브라우저에도 정상적으로 동작함.

사파리에는 아래 버전에서 추가된것 같다.
Note: as of 10-2009 development versions of Safari added support for these new events (see the [webkit bug](https://bugs.webkit.org/show_bug.cgi?id=28758)).

웹 페이지 동작 표준은
1. 유저가 페이지로 네비게이션 하면
2. 페이지가 로드되면서 나서 인라인 스크립트가 실행되고
3. 페이지가 전부 로드되고 나면 `onload` 핸들러가 실행됨
