<template>
  <div>
    <div>
      <ul>
        <li @click="click">추천된 태그1</li>
        <li @click="click">추천된 태그2</li>
      </ul>
    </div>
    <input @focusout="focuseout" />
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      insertedTag: "",
    }
  },
  methods: {
    focuseout() {
      // focuseout 됐을때 처리해야할 로직들..
      console.log("focusout");
    },
    click() {
      // click 됐을때 처리해야할 로직들
      console.log("click");
    }
  },
}
</script>
