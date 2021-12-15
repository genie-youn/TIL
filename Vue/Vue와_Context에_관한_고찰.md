# Vue와 Context에 관한 고찰

UI를 구현하다보면 현재 페이지에서만 사용되는 정보이지만, 페이지 전체에 영향을 주는 즉 페이지에 포함된 컴포넌트에서 전역적으로 접근이 가능해야 하는 상태를 다루는 일이 빈번하게 발생한다.
당연하게도 현재 페이지에서만 사용되는 정보이기 때문에 이 상태들은 페이지와 동일한 라이프사이클을 갖게 된다.

전체적인 구조는 다음과 같다. `Container`에 포함된 모든 컴포넌트들이 전부 API를 통해 받아온 `stateA` 라는 상태에 접근이 필요하다고 가정해보자. 
이 `stateA`는 오로지 `PageA` 페이지에서만 사용되는 상태다.

```
PageAContainer
ㄴ ComponentA
  ㄴ ComponentAA
  ㄴ ComponentAB
    ㄴ ComponentABA
ㄴ ComponentB
ㄴ ComponentC
  ㄴ ComponentCA
```

가장 기본적인 구현은 `PageA`에 필요한 상태를 관리하는 책임을 `Container`에게 부여하고 각 컴포넌트에게 `props` 를 통해 `stateA`를 전달하는 것이다.
이 과정에서 `CompoenntA`는 `props`를 통해 주입받은 `stateA`를 자신의 자식 컴포넌트인 `ComponentAA`, `ComponentAB`에게, `ComponentAA`는 `ComponentABA`에게, `ComponentC`는 `CompoentCA`에게 다시 `props`로 주입해주어야 하는 `Prop drilling`이 발생한다.



container.vue
```vue
<template>
  <div>
    <ComponentA />
    <ComponentB />
    <ComponentC />
  </div>
</template>

<script>
import ComponentA from "./components/ComponentA";
import ComponentB from "./components/ComponentB";
import ComponentC from "./components/ComponentC";
import { fetchStateA } from "@/api/awsome-api";

export default {
  name: "PageAContainer",
  components: {
    ComponentA,
    ComponentB,
    ComponentC,
  },
  data() {
    return {
      stateA: {},
    };
  },
  created() {
    fetchStateA().then((res) => {
      this.stateA = res;
    });
  },
};
</script>
```
