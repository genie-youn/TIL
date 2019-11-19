# Vue와 Event

### 들어가며
Vue를 처음 접하다보면 많이 헷갈려하는 부분이 Event와 관련된 부분이 아닐까 싶다.

기본적으로 DOM에서 이벤트가 동작하는 방식과는 사뭇 다르게 동작하기 때문인데, 왜 기존에 DOM의 Event가 동작하는 방식과는 다르게 설계되었는지 간략히 정리하려 한다.

### DOM과 Event
DOM의 Event의 특징은 크게 두가지, 버블링 <sub>Bubbling</sub> 과 캡쳐 <sub>Capture</sub> 라고 할 수 있겠다. 각각 이벤트가 상위 요소들로 전파되거나, 하위 요소를 따라 탐색하는 걸 일컫는다.

### Vue와 Event
Vue의 Event는 위 두가지, 즉 이벤트의 전파가 존재하지 않는다. 자식 컴포넌트에서 발생한 이벤트가 처리되지 않았다고 할 지라도 이 이벤트가 더 상위의 부모 컴포넌트, 그 부모의 부모 컴포넌트까지 전파되지 않는다. 그래서 처음 Vue 컴포넌트를 구현하다 보면 특정 컴포넌트에서 이벤트를 발생시키고 증조 할아버지쯤 컴포넌트에서 이벤트를 잡으려 노력하며 '왜 이벤트가 안잡히지' 하고 삽질을 하게 되는 것이다.

### 예제
> 예제코드는 다음 [저장소](https://github.com/genie-youn/til-vue-event)를 참고한다.

다음과 같은 컴포넌트가 있다고 하자.

```vue
<template>
  <div class="container">
    <div class="columns">
      <div class="column">
        <Card/>
      </div>
      <div class="column">
        <Card/>
      </div>
      <div class="column">
        <Card/>
      </div>
    </div>
  </div>
</template>

<script>
import Card from '@/components/Card.vue';

export default {
  name: 'HelloWorld',
  components: { Card },
  props: {
    msg: String,
  },
};
</script>
```
