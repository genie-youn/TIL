# Flutter에 Firebase 연동시 크래시가 난다면?

최근 진행중인 토이프로젝트에서 플루터앱에 푸쉬를 구현하기 위해 Firebase를 연동하다 앱이 자꾸 크래시를 내며 죽어버리는 현상이 있었다.

다음과 같은 스택트레이스를 찍으며 자꾸 죽어나갔는데,

```
0   CoreFoundation                      0x00007fff23c7127e __exceptionPreprocess + 350
1   libobjc.A.dylib                     0x00007fff513fbb20 objc_exception_throw + 48
2   CoreFoundation                      0x00007fff23c710bc +[NSException raise:format:] + 188
3   Runner                              0x00000001070ad0ba +[FIRApp configure] + 138
4   Runner                              0x00000001071d7d3c -[FLTFirebaseMessagingPlugin initWithChannel:] + 268
5   Runner                              0x00000001071d7b24 +[FLTFirebaseMessagingPlugin registerWithRegistrar:] + 196
6   Runner                              0x00000001070a82f1 +[GeneratedPluginRegistrant registerWithRegistry:] + 209
```

플루터 이슈를 찾아보니 다음과 같은 이슈가 있었다.

https://github.com/flutter/flutter/issues/16871

주된 내용은 프로퍼티 파일인 `GoogleService-Info.plist`을 바로 넣지말고 Xcode를 통해서 넣어야 한다는 내용인데, 따라해봐도 잘 안됐다.

쭉쭉 내리다 [순서를 바꿔보라는 코멘트](https://github.com/flutter/flutter/issues/16871#issuecomment-554782611) 를 발견해서 `FirebaseApp.configure()`를 `GeneratedPluginRegistrant.register(with: self)` 호출 전으로 옮겨봤더니 정상동작하기 시작했다.

굳이 원인까지 찾아볼 필요는 없을것 같아서 여기까지~

## 마치며
플루터앱에 Firebase 플러그인을 설치했는데 앱이 크래시를 내며 죽어버린다면, `FirebaseApp.configure()`를 `GeneratedPluginRegistrant.register(with: self)` 호출 전으로 옮겨보자.

> 레퍼런스엔 단순히 `FirebaseApp.configure()` 를 추가하세요 라고만 적혀있다.
