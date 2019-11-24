# Vue와 Event

### 들어가며
Vue를 처음 접했을때 많이 헷갈렸던 부분이 Event와 관련된 부분이었다.

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
  <div class="card">
    <CardHeader/>
    <CardImage/>
    <CardContent/>
  </div>
</template>
<script>
import CardImage from '@/components/CardImage.vue';
import CardContent from '@/components/CardContent.vue';
import CardHeader from '@/components/CardHeader.vue';

export default {
  name: 'Card',
  components: {
    CardContent,
    CardImage,
    CardHeader,
  },
};
</script>
```

`CardHeader` 컴포넌트 안에는 다음과 같이 두가지 버튼이 포함되어 있다.

```vue
<template>
  <div class="icon-container">
    <AlertButton/>
    <CheckButton/>
  </div>
</template>

<script>
import AlertButton from '@/components/AlertButton.vue';
import CheckButton from '@/components/CheckButton.vue';

export default {
  name: 'CardHeader',
  components: {
    CheckButton,
    AlertButton,
  },
};
</script>
```

그중 체크유무를 표시하는 컴포넌트는 다음과 같다.
```vue
<template>
<span class="icon" :class="{checked: checked}" @click="check">
      <i class="fas fa-check"></i>
    </span>
</template>
<script>
export default {
  name: 'CheckButton',
  data() {
    return {
      checked: false,
    };
  },
  methods: {
    check() {
      this.checked = !this.checked;
    },
  },
};
</script>
```

정리하면 `Card` > `CardHeader` > `CheckButton` 으로 컴포넌트가 중첩되어 있는 구조이다.
`CheckButton` 에서 클릭 이벤트가 발생하면 `Card`의 `border` 색상을 변경해주고 싶다고 했을 때, 단순히 구현한다면 다음과 같이 `Card` 컴포넌트에 클릭 이벤트 리스너를 등록하여 구현할 수 있을 것이다.

Card.vue
```vue
...
mounted() {
    this.$el.addEventListener('click', (e) => {
      if (e.target.classList.contains('btn-check')) {
        this.$el.style.border = '1px solid #2f9e4d';
      }
    });
  },
...
```

`CheckButton` 에서 발생한 클릭 이벤트가 버블링되어 `Card`까지 전파되고 이를 잡아서 처리하는. 자바스크립트에서 흔한 이벤트 처리방식이다. 그래서 뷰를 처음 접했을 때, 이와 똑같이 이벤트를 구현했었다.
