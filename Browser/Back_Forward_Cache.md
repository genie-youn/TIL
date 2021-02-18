# 들어가며

 매번 헷갈리는 개념인 Back/Forward Cache 에 대해서 정리하려 한다.

# Back/Forward Cache



https://docs.google.com/document/d/1mrgp7XzR16rd1xqFYOJgC1IP0NPLZFaRU5Ukj3-TlLw/edit#heading=h.d9wqdzopmdcf
https://docs.google.com/document/d/19u1jg934oNwE6IzZQ-vkeZLVqQiQWdZSkgTWkzp0CmU/edit
https://webkit.org/blog/427/webkit-page-cache-i-the-basics/
https://web.dev/bfcache/#optimize-your-pages-for-bfcache
https://developers.google.com/web/updates/2019/02/back-forward-cache
https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/1.5/Using_Firefox_1.5_caching

# 파이어폭스 (MDN)
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

캐시되어있는 페이지에 진입하면 인라인 스크립트와 `onload` 핸들러는 동작안함.

대신 `pageshow` 이벤트 잡아서 쓰면됨 (`onload`가 동작하지 않으므로)

반대로 `unload` 도 동작하지 않으므로 `pagehide` 잡아서 쓰면 됨

위 두 이벤트는 캐시된 페이지의 진입이 아닌 최초 진입이라면 `persisted` 의 값이 `true` 로 설정되어 있음

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
   "http://www.w3.org/TR/html4/loose.dtd">
<HTML>
<head>
<title>Order query Firefox 1.5 Example</title>
<style type="text/css">
body, p {
	font-family: Verdana, sans-serif;
	font-size: 12px;
   	}
</style>
<script type="text/javascript">
function onLoad() {
	loadOnlyFirst();
	onPageShow();
}

function onPageShow() {
//calculate current time
	var currentTime= new Date();
	var year=currentTime.getFullYear();
	var month=currentTime.getMonth()+1;
	var day=currentTime.getDate();
	var hour=currentTime.getHours();
	var min=currentTime.getMinutes();
	var sec=currentTime.getSeconds();
	var mil=currentTime.getMilliseconds();
	var displayTime = (month + "/" + day + "/" + year + " " +
		hour + ":" + min + ":" + sec + ":" + mil);
	document.getElementById("timefield").value=displayTime;
}

function loadOnlyFirst() {
	document.zipForm.name.focus();
}
</script>
</head>
<body onload="onLoad();" onpageshow="if (event.persisted) onPageShow();">
<h2>Order query</h2>

<form name="zipForm" action="http://www.example.com/formresult.html" method="get">
<label for="timefield">Date and time:</label>
<input type="text" id="timefield"><br>
<label for="name">Name:</label>
<input type="text" id="name"><br>
<label for="address">Email address:</label>
<input type="text" id="address"><br>
<label for="order">Order number:</label>
<input type="text" id="order"><br>
<input type="submit" name="submit" value="Submit Query">
</form>
</body>
</html>
```

위 예제는 사용자가 페이지를 벗어낫다 다시 돌아오면 현재 시간을 다시 계산해서 보여주게 됨

```html
<script>
function onLoad() {
	loadOnlyFirst();

//calculate current time
	var currentTime= new Date();
	var year = currentTime.getFullYear();
	var month = currentTime.getMonth()+1;
	var day = currentTime.getDate();
	var hour=currentTime.getHours();
	var min=currentTime.getMinutes();
	var sec=currentTime.getSeconds();
	var mil=currentTime.getMilliseconds();
	var displayTime = (month + "/" + day + "/" + year + " " +
		hour + ":" + min + ":" + sec + ":" + mil);
	document.getElementById("timefield").value=displayTime;
}

function loadOnlyFirst() {
	document.zipForm.name.focus();
}
</script>
</head>

<body onload="onLoad();">
```

하지만 이 예제는 처음 페이지에 진입했을때 시간이 캐시되고 사용자가 페이지를 벗어났다 다시 돌아와도 이전의 시간이 캐시되어 보여지게 됨

# 사파리 (Webkit 레퍼런스)

웹킷 페이지 캐시

웹킷의 페이지 캐시 = 파이어폭스의 Back-Forward Cache = 오페라의 Fast History Navigation

이에 대한 웹킷의 구현을 "Page Cache" 라고 지칭함으로써 웹킷의 "Back/Forward List" 와 혼란을 줄이고자 함.

페이지 캐시는 최종사용자가 웹페이지를 더 부드럽게 네비게이션하기 위한 기능임

엄밀히 말하면 [HTTP Sense](https://www.ietf.org/rfc/rfc2616.txt) 에서 얘기하는 캐시와는 다름

원본 리소스가 디스크에 저장되는 "disk cache" 와는 결이 다름 ??

그리고 웹킷이 여러 웹 페이지에서 공유하기 위해 디코딩된 리소스를 메모리에 가지고 있는 관습적인 의미의 "memory cache" 와도 차이가 있다 ??

간단히 얘기하면 사용자가 페이지를 벗어날때 페이지를 "pause" 하고 다시 돌아오면 "play" 하는것과 같음

사용자가 링크를 클릭하여 새로운 페이지로 네비게이션하면 이전 페이지가 완전히 제거되는 경우가 많음

돔이 제거되면 자바스크립트 객체는 가비지 컬렉터의 수집 대상이 되고 플러그인은 제거되며 디코딩된 이미지 데이터가 삭제되고 기타 다른 정리를 위한 일들이 일어남

위의 일들이 일어나면 사용자가 뒤로가기를 클릭했을때 고통스러워짐. 웹킷은 아마 리소스를 네트워크를 통해 다시 내려받고 메인 HTML 파일을 다시 파싱하고 스크립트를 다시 실행시키고 이미지를 다시 디코딩하고 페이지를 다시 레이아웃하고 적절한 위치로 스크롤을 다시 옮겨주고 스크린을 다시 그려줘야 함. 이 모든 작업은 시간을 소비하고 CPU를 사용하며 배터리를 소모시킨다.

이상적으로 이전 페이지는 페이지 캐시로 대체할 수 있다.

화면에 표시되지 않더라도 페이지 전체를 메모리에 저장함. 전부 파괴하는 대신 일시정지 시키고 뒤로가기 버튼을 눌렀을 경우 다시 재생시키는것

뒤로가기시 이전에 보던 페이지를 거의 즉시 볼 수 있어 더 나은 사용자 경험을 제공함

이렇게 좋은 페이지 캐시가 동작하지 않을때 그 이유는?

#### 몇몇 페이지는 흥미롭지 않음?

페이지가 정확히 동일한 상태를 반환하지 않는다면 이걸 캐싱하는건 의미가 없음

예를 들어 페이지가 로딩이 끝나지 않았을 경우나, 페이지가 로딩중에 에러가 났거나, 사용자를 새로운 URL로 이동시키기 위해서만 존재하는 리다이렉션 페이지이거나..

#### 몇몇 페이지는 너무 복잡함

어떻게 "pause" 할지 찾아내기 어려운 페이지는 페이지 캐시 대상으로 고려되지 않음

예를들어 플러그인은 Webkit이 "pause" 버튼을 누르지 않도록 원하는 모든것을 할 수 있는 네이티브 코드를 포함하고 있다...???

또다른 예로는 Webkit이 역사적으로 캐시하지 않은 멀티 프레임을 가지고 있는 페이지가 있음

#### 보안이 필요한 페이지

HTTPS 사이트의 서버 관리자는 종종 보안문제를 겪으며 브라우저의 동작에 민감하게 반응한다.

예를들어, 금융기관의 경우 고객이 허용하기 전 각 브라우저의 동작을 철저히 확인한다.

그중 Back/Forward 동작은 특히나 주의가 필요함. 이런 기관들은 당연히 사용자가 탐색할 때 브라우저에 남겨진 데이터의 유형에 대해 매우 까다로움.

그 결과, 웹킷은 처음부터 페이지 캐시에서 모든 HTTPS 사이트를 허용하지 않도록 하여 보수적으로 접근하였다.

보다 세밀한 접근 방식은 사용자 경험을 개선하는 데 큰 도움이 될 수 있다.

#### 계획된 개선
현재 스펙으로 처리할 수 없는 몇가지 중요한 케이스가 있고, 개선의 여지가 있다.

웹킷의 페이지 캐시는 첫번째 사파리 베타 릴리즈 전인 2002년에 작성되었음. 이 기능은 그 당시 웹킷의 아키텍처와 2002년 Web의 풍경을 모두 반영했다.

2009년도의 웹은 많은 변화가 있었고 (...) 페이지 캐시도 이에 맞게 끌어올려야만 했다. 다행히 이 작업은 순조롭게 진행중이다.

예를들어 [revision 48306](https://trac.webkit.org/changeset/48036/webkit) 에서 주요한 제약사항이 해결되어 프레임을 포함하는 페이지 또한 페이지 캐시될 수 있다. 최신 nigthly 버전의 웹킷을 사용하여 브라우징을 한다면 더 빨라졌단걸 느낄 수 있었을것..

하지만 아직 개선되어야 할 점이 더 많음

플러그인이 개선해야할 리스트중 다음으로 큰 부분. 앞서 얘기했듯 플러그인은 원하는 네이티브 코드를 실행시킬 수 있으므로 "pause" 버튼을 안정적으로 누를 수 없다.

초기 버전의 웹킷은 몇가지 타입의 플러그인과 함께 단일 프레임 페이지들을 다루었음.
웹킷은 페이지를 떠날때 플러그인을 해제하고 사용자가 돌아왔을 때 플러그인을 복원함.

그러나 WebCore 에 대한 작업이 계속되면서 더 빠르고 쉽게 포팅하여 이러한 기능은 사라졌다?

[Bug #13634](https://bugs.webkit.org/show_bug.cgi?id=13634) 는 모든 페이지의 모든 플러그인이 동작하도록 지원한다.

그 다음으로는 HTTPS 페이지인데, 현재는 전부다 캐시를 태우지 않지만 선택적으로 접근하여 이러한 기관들의 요구를 만족시키면서도 유저들에게 캐시의 이점을 제공할 것임

[Bug #26777](http://webkit.org/b/26777) 은 응답 헤더에 `cache-control: no-store` 또는 `cache-control: no-cache` 가 포함되지 않는 한 HTTPS 페이지를 캐시하도록 허용했음. 이는 기관들이 선택적으로 컨텐츠를 보호할 수 있는 수단을 제공함.

#### Unload Handlers

unload 이벤트는 사용자가 페이지를 닫을 때 정리하는 작업을 하기 위해 설계되었음

그렇기 때문에 페이지가 터미널 상태에 있다고 가정하고 페이지의 주요한 부분을 파괴할 수 도 있는 정리 작업을 진행할 수 있기 때문에 브라우저는 페이지가 페이지 캐시에 캐싱되기 전에 unload 이벤트를 발생시킬 수 없다. unload 이벤트 핸들러를 통해 페이지의 주요한 로직을 정리해버리면 페이지 캐싱하는 의미가 없으니까.

그러나 만약 브라우저가 unload 핸들러 없이 페이지를 페이지 캐시에 캐싱한다면, 그 페이지는 브라우저가 "pauesd" 된 동안 제거될 가능성이 있지만 중요한 작업이 포함되어 있을지도 모르는 정리 작업은 동작하지 않게 됨

unload 이벤트의 목적은 '페이지를 닫을 때 중요한 작업을 허용하는 것' 이기 때문에 모든 주요 브라우저가 unload 이벤트 핸들러를 가진 페이지를 페이지 캐싱에 캐싱하지 않으며 이는 사용자 경험에 직접적인 부정적인 영향을 끼치게 된다.

아직 해결하지 못한 숙제이니 계속 지켜봐줘~~
