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
    <input @focusout="focusout" />
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
