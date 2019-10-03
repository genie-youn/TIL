# 크롬 개발자도구에서 Remote Device를 못 찾을 때

인앱 브라우저에서 동작하는 스크립트를 디버깅하기 위해 늘 하던대로 개발자도구를 켜고, USB 디버깅을 켠다음, USB를 연결해서 크롬의 Remote Device 탭을 열었는데 왠걸 디바이스를 찾을 수 없는 일이 발생했다.
> 자세한 방법은 [여기](https://developers.google.com/web/tools/chrome-devtools/remote-debugging?utm_campaign=2016q3&utm_medium=redirect&utm_source=dcc)

### 삽질
승인된 기기를 전부 지워보기도 하고, 옆자리 동료 맥에도 꽂아보고 케이블도 바꿔봤지만 읽힐 생각은 전혀 없던 찰나 스택오버플로우에서 Android 디버그 브리지(adb)를 받아서 돌려보라는 글을 하나 보았다.

이 [링크](https://developer.android.com/studio/releases/platform-tools) 에 들어가서 `./adb devices` 를 돌려보면 콘솔에 연결되있던 디바이스를 찾을 수 있는것을 확인할 수 있다. 그러면 크롬에도 연결된 디바이스가 떠 있는 것을 볼 수 있다.

> adb에 대한 자세한 설명은 [여기](https://developer.android.com/studio/command-line/adb?gclid=Cj0KCQjwuNbsBRC-ARIsAAzITuedSIz0vnnWTeIax-fiI4dQw6x0HU7ZgZeBB-Po51oSD-LUNVCk7a8aAvV4EALw_wcB) 를 참고한다.

### 결론
원인은 모르겠지만.. 늘 하던게 안되서 두시간은 해맸던 것 같다. ㅠ.ㅠ
