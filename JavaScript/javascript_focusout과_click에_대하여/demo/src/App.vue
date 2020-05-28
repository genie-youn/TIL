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
