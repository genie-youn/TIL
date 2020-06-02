# javascript focusout과 click에 대하여

## 들어가며
얼마 전 게시글에 태그를 입력하는 기능을 만들던 중이었다. 태그를 입력하면 그와 관련있는 태그들을 레이어로 노출해주고 이걸 선택하면 태그가 입력되게 된다. 또한 태그를 작성하다 다른곳을 클릭하면 작성중인 태그는 박싱되고 태그입력이 완료되어야 한다. 간략히 코드로 보면 다음과 같다.

```vue
<template>
  <div>
    <!-- 작성중인 태그에 따라 노출, 간단히 하기 위해 입력중인게 있으면 노출 -->
    <div v-if="insertedTag">
      <ul>
        <li @click="click">추천된 태그1</li>
        <li @click="click">추천된 태그2</li>
      </ul>
    </div>
    <input @focusout="focusout" v-model="insertedTag" />
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      // 현재 작성중인 태그
      insertedTag: "",
    }
  },
  methods: {
    focusout() {
      // focusout 되면 input 을 비워주고 작성중이던 태그를 등록해야함
      this.insertedTag = "";
      // ...작성중인 태그 저장 로직 생략
    },
    click() {
      // click 됐을때 처리해야할 로직들 생략
      console.log("click");
    }
  },
}
</script>
```
태그를 입력하다 다른곳을 클릭하게 되면 입력중이던 텍스트를 새로운 태그로 등록하고 입력중이던 텍스트는 빈 문자열로 초기화 해주기 위하여 `focusout` 이벤트에 관련 로직들을 주었다.

추천된 태그들은 클릭하면 해당 텍스트를 입력하는 로직을 처리하기 위해 `click` 이벤트를 핸들링한다.

여기서 두 이벤트간 충돌이 발생한다. 텍스트를 입력하면 관련된 태그가 나타나고, 나타난 태그를 클릭하는 순간 인풋에서 `focuseout` 이 발생하여
`insertedTag` 는 빈문자열로 초기화, 입력된 텍스트를 기반하여 추천되어 나타난 태그는 함께 사라지게 되고 DOM이 사라졌으니 `click` 이벤트도 발생하지 않게 된다.

즉, 추천 태그의 `click` 보다 인풋의 `focusout` 이 먼저 트리거되서 발생하는 이슈이다. `Performance` 탭을 보면 `mousedown` > `focusout` > `mouseup` 순으로 이벤트가 트리거 되는것을 확인할 수 있다.

![](https://user-images.githubusercontent.com/16642635/83576804-8505f580-a56d-11ea-9e65-2e2af4c72ae0.png)
