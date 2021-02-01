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
