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

https://bugs.webkit.org/show_bug.cgi?id=195884#c2
https://bugs.webkit.org/show_bug.cgi?id=190017
https://bugs.webkit.org/show_bug.cgi?id=214434
https://bugs.webkit.org/attachment.cgi?id=404507&action=prettypatch
https://bugs.webkit.org/show_bug.cgi?id=213149
https://bugs.webkit.org/attachment.cgi?id=401807&action=prettypatch
https://bugs.webkit.org/show_bug.cgi?id=207272
https://bugs.webkit.org/attachment.cgi?id=389873&action=prettypatch
https://bugs.webkit.org/show_bug.cgi?id=195856
https://bugs.webkit.org/attachment.cgi?id=367298&action=prettypatch
https://bugs.webkit.org/show_bug.cgi?id=142757
