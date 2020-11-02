# iOS 모바일웹에서 키보드 처리

## 들어가며
각 항목의 수정이 가능한 리스트가 있고, 특정 버튼을 누르면 해당 리스트에 새로운 항목을 추가한 뒤 자동으로 포커스를 주어 새로운 항목을 입력하게 하는 UI를 구현중, iOS 모바일 브라우저에서만 의도한 대로 동작하지 않는 이슈를 만났다.

포커스를 아무리 주어도 한번 더 클릭하기 전에는 키보드가 올라오지 않는게 문제였는데, 어렴풋이 iOS 에선 키보드를 임의로 올리는게 불가능하단 얘기를 들은적은 있지만 정확히 무엇이 원인인지는 이해하고 있지 않아 이 기회에 정리해두려 한다.

동작가능한 예제는 [다음](https://codepen.io/genie-youn/pen/abZbyzq) 을 참고한다.

## 구성
코드를 간략하게 살펴보면 다음과 같다.

```html
<div id="app">
  <button @click="addItem">add item</button>
  <ul ref="itemContainer">
    <li v-for="(item, index) in items" :key=index>
      <textarea :id="`item-${index}`" :value="item"></textarea>
    <li>
  </ul>
</div>
```

UI는 하나의 버튼과 하나의 리스트로 구성되어 있고, 리스트는 `items` 에 반응하여 `item` 을 값으로 갖는 `textarea` 를 렌더링한다.

```javascript
const Demo = {
  data() {
    return {
      items: ["item1", "item2", "item3"]
    }
  },
  methods: {
    addItem() {
      const length = this.items.push("");
      this.$nextTick(() => document.querySelector(`#item-${length - 1}`).focus());
    }
  }
}

Vue.createApp(Demo).mount('#app')
```

버튼을 클릭하면 새로운 항목을 추가하고 추가된 항목에 포커스를 주는 `addItem` 을 호출하도록 되어 있다.

위 코드를 실행시켜보면 모든 브라우저에서 의도한대로 정상적으로 동작하는것을 확인할 수 있다.

만약 `addItem` 내에서 서버와 API 요청을 처리한 후에 항목을 추가한 뒤 포커스를 준다면 어떻게 될까?

```javascript
// 서버에 항목을 추가할 수 있는지 체크하는 API를 호출한다고 가정
const checkAppendable = () => new Promise(resolve => {
  setTimeout(() => {
    resolve();
  }, 300)
});

const Demo = {
  data() {
    return {
      items: ["item1", "item2", "item3"]
    }
  },
  methods: {
    addItem() {
      // 변경된 부분. API를 통해 항목을 더 추가 가능한지 확인 후 새로운 항목을 추가하고 렌더링되면 포커스를 준다.
      checkAppendable().then(res => {
        const length = this.items.push("");
        this.$nextTick(() => document.querySelector(`#item-${length - 1}`).focus());
      });
    }
  }
}
Vue.createApp(Demo).mount('#app')
```

첫번째 예제와 달리 두번째 예제를 모바일 iOS 기기의 브라우저들에서 실행해보면 포커스는 정상적으로 주어지나 시스템 키보드가 올라가지 않는 것을 확인할 수 있다.

## 이유
첫번째 예제에선 키보드가 정상적으로 올라가지만 두번째 예제에선 정상적으로 올라가지 않는 이유는 중간에 API를 호출하는 비동기 동작이 포함되었기 때문이다.

iOS webkit은 [포커스가 사용자 제스처에 대한 응답이 아닌 경우엔 시스템 키보드가 올라가지 않도록 설게](https://bugs.webkit.org/show_bug.cgi?id=195884#c4)되었다. 키보드를 불러오는게 화면중 많은 공간을 차지하고 잘 입력할 수 있도록 페이지를 확대/축소 하거나 스크롤 하는데 이러한 동작이 유저의 제스처가 아닌 프로그래밍되어 자동으로 주어지는 포커스에 의해 실행된다면 사용자에게 짜증나고, 방해가 되는 사용자 경험을 만든다고 판단했기 때문이다.

그렇다면 '사용자 제스처에 대한 응답' 이란 무엇일까? 코드를 살펴보자.

[/Source/WebKit/UIProcess/ios/WKContentViewInteraction.mm](https://github.com/WebKit/webkit/blob/bc26aef617f2cbcbf4ed47cc6d76f6f76a7acfdc/Source/WebKit/UIProcess/ios/WKContentViewInteraction.mm#L5906-L5941)
```c++
BOOL shouldShowInputView = [&] {
        switch (startInputSessionPolicy) {
        case _WKFocusStartsInputSessionPolicyAuto:
            // The default behavior is to allow node assistance if the user is interacting.
            // We also allow node assistance if the keyboard already is showing, unless we're in extra zoom mode.
            if (userIsInteracting)
                return YES;

            if (self.isFirstResponder || _becomingFirstResponder) {
                // When the software keyboard is being used to enter an url, only the focus activity state is changing.
                // In this case, auto focus on the page being navigated to should be disabled, unless a hardware
                // keyboard is attached.
                if (activityStateChanges && activityStateChanges != WebCore::ActivityState::IsFocused)
                    return YES;

#if PLATFORM(WATCHOS)
                if (_isChangingFocus && ![_focusedFormControlView isHidden])
                    return YES;
#else
                if (_isChangingFocus)
                    return YES;

                if ([UIKeyboard isInHardwareKeyboardMode])
                    return YES;
#endif
            }
            return NO;
        case _WKFocusStartsInputSessionPolicyAllow:
            return YES;
        case _WKFocusStartsInputSessionPolicyDisallow:
            return NO;
        default:
            ASSERT_NOT_REACHED();
            return NO;
        }
    }();
```

__

기본값인 `_WKFocusStartsInputSessionPolicyAuto`일 경우에 다음과 같은 조건에서 inputView에 진입하게 된다.
1. 파라미터로 주입받은 `userIsInteracting` 가 `true` 일 때
2. `(self.isFirstResponder || _becomingFirstResponder) && (activityStateChanges && activityStateChanges != WebCore::ActivityState::IsFocused)` 가 `true` 일 때
3. `(self.isFirstResponder || _becomingFirstResponder) && _isChangingFocus` 가 `true` 일때
4. `(self.isFirstResponder || _becomingFirstResponder) && [UIKeyboard isInHardwareKeyboardMode]` 가 `true` 일 때

하나씩 살펴보도록 하자.

우선 1의 경우는 파라미터로 전달받은 호출은 `userIsInteracting` 을 찾아야 한다. 호출은 여기서 한다.
[/Source/WebKit/UIProcess/ios/PageClientImplIOS.mm](https://github.com/WebKit/webkit/blob/3c7f9853695401da0181d257702d735615222a3e/Source/WebKit/UIProcess/ios/PageClientImplIOS.mm#L580-L598)

```c++
void PageClientImpl::elementDidFocus(const FocusedElementInformation& nodeInformation, bool userIsInteracting, bool blurPreviousNode, OptionSet<WebCore::ActivityState::Flag> activityStateChanges, API::Object* userData)
{
    MESSAGE_CHECK(!userData || userData->type() == API::Object::Type::Data);

    NSObject <NSSecureCoding> *userObject = nil;
    if (API::Data* data = static_cast<API::Data*>(userData)) {
        auto nsData = adoptNS([[NSData alloc] initWithBytesNoCopy:const_cast<unsigned char*>(data->bytes()) length:data->size() freeWhenDone:NO]);
        auto unarchiver = adoptNS([[NSKeyedUnarchiver alloc] initForReadingFromData:nsData.get() error:nullptr]);
        unarchiver.get().decodingFailurePolicy = NSDecodingFailurePolicyRaiseException;
        @try {
            auto* allowedClasses = m_webView.get()->_page->process().processPool().allowedClassesForParameterCoding();
            userObject = [unarchiver decodeObjectOfClasses:allowedClasses forKey:@"userObject"];
        } @catch (NSException *exception) {
            LOG_ERROR("Failed to decode user data: %@", exception);
        }
    }

    [m_contentView _elementDidFocus:nodeInformation userIsInteracting:userIsInteracting blurPreviousNode:blurPreviousNode activityStateChanges:activityStateChanges userObject:userObject];
}
```

**

여기도 전달받은 파라미터를 그대로 토스한다.



이거 두개 보다가 말았다..
https://trac.webkit.org/changeset/230902/webkit
https://bugs.webkit.org/show_bug.cgi?id=184844


https://bugs.webkit.org/show_bug.cgi?id=195884#c4
https://bugs.webkit.org/show_bug.cgi?id=190017
https://bugs.webkit.org/show_bug.cgi?id=214434
https://bugs.webkit.org/show_bug.cgi?id=213149
https://bugs.webkit.org/show_bug.cgi?id=207272
https://bugs.webkit.org/show_bug.cgi?id=195856
https://bugs.webkit.org/show_bug.cgi?id=193811
https://bugs.webkit.org/show_bug.cgi?id=142757
