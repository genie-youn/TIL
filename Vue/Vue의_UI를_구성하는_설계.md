# Vue의 UI를 구성하는 설계

vue는 부모 컴포넌트로부터 주입받은 `props`와 자신의 상태인 `state`에 반응형으로 UI를 렌더링하는 `component`들을 조합하여 전체 UI를 구성한다.
이 컴포넌트들은 재사용성을 확보하고, 요구사항이 변경되었을 때 기능을 확장하고 변경하기 쉽도록 책임에 따라 최소한의 단위로 작게 나누어진다.
때로는 하나의 컴포넌트가 여러 자식 컴포넌트를 갖기도 하고, 그 자식 컴포넌트가 또 다시 여러 자식컴포넌트를 갖기도 한다.

이에 따라 많은 정보를 표현해야 하는 웹 페이지는 많은 단계의 깊이를 가진 복잡한 계층구조의 컴포넌트 트리를 갖게되며, 
이 많은 컴포넌트들이 정보를 표현하는데 필요로 하는 정보는 대개 페이지와 동일한 라이프사이클을 갖는다.

이러한 상황에서 선택할 수 있는 컴포넌트들의 협력을 설계하는 몇가지 방법을 소개한다.

### Basic
가장 기본적인 설계로 컴포넌트의 기본 정의에 충실히 각 컴포넌트들은 자신이 필요한 상태를 직접 관리한다. 

컴포넌트가 생성될때 API를 통해 정보를 요청하고 이를 자신의 `data`로 관리한다.

동일한 계층에 존재하는 컴포넌트들이 동일한 정보를 필요로 하다면, 이를 부모 컴포넌트에 위임하고 `props`로 전달받도록 한다.

pros: 
- 간단하다.
cons: 
- 상태에 관련된 로직이 복잡할 경우, 컴포넌트에 상태에 관련된 로직과 이를 가지고 UI를 렌더링하는 로직, 사용자와의 상호작용을 처리하는 로직과 섞여 복잡도가 높은 컴포넌트를 만든다.

### Presentational and Container Components
기본적인 설계의 단점을 해결하기 위해 컴포넌트로부터 상태에 관련된 로직을 분리해낸 `Container` 컴포넌트 (이하 컨테이너)를 정의하고 상태에 관련된 로직을 가지고 있지 않은 `Presentational` 컴포넌트(이하 컴포넌트)로 분류한다.

상태 관리에 대한 책임은 컨테이너에게 부여하고 컴포넌트는 컨테이너로부터 `props`를 주입받아 상태에 따라 UI를 렌더링하고 사용자와의 상호작용을 처리하는 책임만을 갖는다.

컨테이너는 기본적으로 페이지 (라우팅의 대상이 되는) 단위로 구성된다.

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

`PageA`에 필요한 상태를 관리하는 책임을 `PageAContainer`에게 부여한다. 
`PageAContainer`는 API를 통해 `PageA`에 필요한 정보 `StateA`를 fetch한 뒤 이를 각 컴포넌트에 `props`로 전달할것이다.

이 과정에서 컴포넌트 트리 가장 마지막 계층에 있는 `ComponentABA`가 `StateA`의 `B`라는 상태를 필요로 한다면 `ComponentA`에게 `StateA.B`를 `props`로 주입하고,
`ComponentA`는 이를 자신의 자식 컴포넌트인 `ComponentAB`에게 주입하고, `ComponentAB`는 이를 다시 자신의 자식 컴포넌트인 `ComponentABA`에게 주입해야 하는 `Prop drilling`이 발생한다.

ComponentABA와 같은 컴포넌트 트리 말단에 위치한 컴포넌트가 사용자와의 상호작용에 따라 `StateA.B`에 변경이 필요하다면 위 과정을 역으로 거쳐 이벤트를 발생시켜 `PageAContainer` 에게 메세지를 전달하거나, 이에 대한 콜백을 함께 `props`로 주입받아야 할 것이다.

<img width="710" alt="스크린샷 2021-12-16 오전 9 05 37" src="https://user-images.githubusercontent.com/16642635/146284032-6a6f94a4-3c99-4256-9f45-f667993f2bff.png">


만약 `PageA`의 상태관련된 로직이 너무 복잡하여 `PageAContainer`가 너무 복잡해지고 많은 책임을 지게 되었다면 이를 표현하는 상태를 기준으로 더 작은 컨테이너로 나눌 수 있다.

```
PageAContainer
ㄴ ContainerA
  ㄴ ComponentA
    ㄴ ComponentAA
    ㄴ ComponentAB
      ㄴ ComponentABA
ㄴ ComponentB
ㄴ ContainerC
  ㄴ ComponentC
    ㄴ ComponentCA
```

각각의 컨테이너는 독립적인 동작을 보장하는 컨텍스트의 단위가 된다. 즉 다음과 같이 사용이 가능해야 한다.

```
PageBContainer
ㄴ ContainerA
  ㄴ ComponentA
    ㄴ ComponentAA
    ㄴ ComponentAB
      ㄴ ComponentABA
```

pros:
- 컴포넌트로부터 상태를 관리하는 로직을 제거했기 때문에 컴포넌트는 `UI = f(state)`의 철학에 맞게 주어진 `state`로 UI를 구성하는 순수한 상태를 유지할 수 있다.
- 컴포넌트를 순수한 상태로 유지할 수 있기 때문에 재사용하기 쉽고 테스트하기 쉬운 컴포넌트를 작성할 수 있다.
- 컴포넌트의 복잡도를 많이 낮출 수 있다.
cons:
- prop drilling
- 컴포넌트가 순수하게 유지될 수 있도록 하기위해  side-effect를 일으킬 수 있는 함수를 `props`로 전달하거나 prop drilling의 역으로 이벤트를 전달받아야 한다.
- 상태 관리 로직이 복잡할경우 컨테이너가 가진 다른 관점의 책임들 (레이아웃을 구성하고, 자신의 자식 컴포넌트간의 관계를 설정하고, 자신의 자식 컴포넌트가 참여하는 flow 로직을 처리하는 등) 과 섞여 복잡해진다.

> Presentational and Container Component의 자세한 내용은 Dan Abramov의 [아티클](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)을 참고.
> 페이지 컨테이너는 해당 패턴을 사용하면서 저자가 개인적으로 정의한 개념임.
> Dan Abramov는 현재는 컨테이너보단 hooks를 사용하여 상태 관리 로직을 분리할것을 권장하고 있음.

### Provide/Inject
컨테이너-컴포넌트 구조를 유지하면서 prop drilling을 해결하기 위해 Vue의 Provide/Inject를 사용할 수 있다. 필요한 상태와 side-effect를 일으키는 콜백 provide를 통해 주입받으면 컨테이너-컴포넌트의 단점을 해결할 수 있다.

PageAContainer.vue
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
import { computed } from "vue";

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
      console.log(res);
      this.stateA = res;
    });
  },
  provide() {
    return {
      stateA: computed(() => this.stateA),
    };
  },
};
</script>
```

ComponentABA.vue
```vue
<template>
  <span>{{ stateA }}</span>
</template>

<script>
export default {
  name: "ComponentABA",
  inject: ["stateA"],
};
</script>

```<img width="838" alt="스크린샷 2021-12-16 오전 9 07 56" src="https://user-images.githubusercontent.com/16642635/146284227-50681d62-36cf-4e6e-b0e5-3c8694d231de.png">

pros:
- 컴포넌트로부터 상태를 관리하는 로직을 제거했기 때문에 컴포넌트는 `UI = f(state)`의 철학에 맞게 주어진 `state`로 UI를 구성하는 순수한 상태를 유지할 수 있다.
- 컴포넌트를 순수한 상태로 유지할 수 있기 때문에 재사용하기 쉽고 테스트하기 쉬운 컴포넌트를 작성할 수 있다.
- 컴포넌트의 복잡도를 많이 낮출 수 있다.
cons:
- provide-inject간 관계가 명시적으로 드러나지는 않기 때문에 코드 가독성이 저하될 수 있다.
- 상태 관리 로직이 복잡할경우 컨테이너가 가진 다른 관점의 책임들 (레이아웃을 구성하고, 자신의 자식 컴포넌트간의 관계를 설정하고, 자신의 자식 컴포넌트가 참여하는 flow 로직을 처리하는 등) 과 섞여 복잡해진다.
