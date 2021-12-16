# Vue의 UI를 구성하는 설계

vue는 부모 컴포넌트로부터 주입받은 `props`와 자신의 상태인 `state`에 반응형으로 UI를 렌더링하는 `component`들을 조합하여 전체 UI를 구성한다.
이 컴포넌트들은 재사용성을 확보하고, 요구사항이 변경되었을 때 기능을 확장하고 변경하기 쉽도록 책임에 따라 최소한의 단위로 작게 나누어진다.
때로는 하나의 컴포넌트가 여러 자식 컴포넌트를 갖기도 하고, 그 자식 컴포넌트가 또 다시 여러 자식컴포넌트를 갖기도 한다.

이에 따라 많은 정보를 표현해야 하는 웹 페이지는 많은 단계의 깊이를 가진 복잡한 계층구조의 컴포넌트 트리를 갖게되며,
이 많은 컴포넌트들이 정보를 표현하는데 필요로 하는 정보는 대개 페이지와 동일한 라이프사이클을 갖게되고 컴포넌트 트리 전역에서 접근할 수 있어야 한다.

이러한 상황에서 선택할 수 있는 컴포넌트들의 협력을 설계하는 몇가지 방법을 소개한다.

> 일반적인 상황이 아닌, 페이지와 동일한 라이프사이클을 가진 복잡한 정보를 표현하는 페이지를 구현하기 위하여 페이지 단위의 상태관리가 필요하고 깊은 계층의 컴포넌트 트리를 가진 페이지를 구현해야하는 상황이다.

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
      this.stateA = res;
    });
  },
  provide() {
    return {
      B: computed(() => this.stateA.B),
    };
  },
};
</script>
```

ComponentABA.vue
```vue
<template>
  <span>{{ B }}</span>
</template>

<script>
export default {
  name: "ComponentABA",
  inject: ["B"],
};
</script>
```

<img width="838" alt="스크린샷 2021-12-16 오전 9 07 56" src="https://user-images.githubusercontent.com/16642635/146284227-50681d62-36cf-4e6e-b0e5-3c8694d231de.png">

pros:
- 컴포넌트로부터 상태를 관리하는 로직을 제거했기 때문에 컴포넌트는 `UI = f(state)`의 철학에 맞게 주어진 `state`로 UI를 구성하는 순수한 상태를 유지할 수 있다.
- 컴포넌트를 순수한 상태로 유지할 수 있기 때문에 재사용하기 쉽고 테스트하기 쉬운 컴포넌트를 작성할 수 있다.
- 컴포넌트의 복잡도를 많이 낮출 수 있다.
cons:
- provide-inject간 관계가 명시적으로 드러나지는 않기 때문에 코드 가독성이 저하될 수 있다.
- 상태 관리 로직이 복잡할경우 컨테이너가 가진 다른 관점의 책임들 (레이아웃을 구성하고, 자신의 자식 컴포넌트간의 관계를 설정하고, 자신의 자식 컴포넌트가 참여하는 flow 로직을 처리하는 등) 과 섞여 복잡해진다.

### Page Store
컴포넌트-컨테이너 구조에서 prop drilling과 컨테이너의 복잡함을 해결하기 위해 페이지의 상태를 관리하는 책임을 페이지 스토어를 정의하고 이에 위임한다.

컨테이너는 더 이상 상태관리의 책임을 지지 않으며, 생성시 스토어에게 메세지를 전달하고 레이아웃을 구성하고, 자식 컴포넌트들의 관계를 설정하며 이들이 참여하는 flow 로직을 처리하는 책임을 갖는다.

`PageA`에 필요한 상태는 전부 `PageAStore`에서 관리되며 각 컴포넌트들은 필요한 상태를 `PageAStore`로부터 `getters`를 통해 접근하고 `actions`를 통하여 side-effect를 발생시킨다.

이때 컴포넌트가 스토어에 대한 의존성을 갖게 되어 더 이상 순수하지 않은 상태가 된다.

따라서 재사용성을 확보해야만 하기 때문에 순수하게 유지할 컴포넌트와 페이지 외의 다른 페이지에서 사용할 가능성이 없어 순수하게 유지할 필요는 없지만, 유지보수성을 확보하기 위해 컴포넌트화한 컴포넌트간의 구분이 필요하며 전자의 경우 `props`와 `event`를 통해 협력에 참여하게 하고, 후자의 경우 스토어를 통해 협력에 참여하게 한다.

ContainerA.vue
```vue
<template>
  <div>
    <ComponentA />
    <ComponentB />
    <ComponentC />
  </div>
</template>

<script>
import { mapActions } from "vuex";
import ComponentA from "./components/ComponentA";
import ComponentB from "./components/ComponentB";
import ComponentC from "./components/ComponentC";

export default {
  name: "PageAContainer",
  components: {
    ComponentA,
    ComponentB,
    ComponentC,
  },
  created() {
    this.initialize();
  },
  methods: {
    ...mapActions("PageAStore", ["initialize"]),
  },
};
</script>
```

ComponentABA.vue
```vue
<template>
  <span>{{ B }}</span>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  name: "ComponentABA",
  computed: {
    ...mapGetters("PageAStore", ["B"]),
  },
};
</script>
```

pros:
- 컨테이너의 복잡성을 낮출 수 있다.
- 컨테이너와 컴포넌트 모두에서 상태를 관리하기 위한 로직이 분리된다.
- 페이지에 필요한 상태에 대한 전역적인 접근 방법을 제공한다.
cons:
- 스토어는 기본적으로 특정 페이지를 위한 저장소가 아니고 애플리케이션 전체에서 전역적으로 접근이 필요한 상태들을 관리하는 저장소이다.
- 그렇기 때문에 페이지 단위로 라이프사이클을 관리할 수 없으며 글로벌 스토어가 이러한 페이지를 위한 스토어들이 모두 등록될 경우 스토어의 네임스페이스가 오염될 가능성이 높다.
- 이러한 단점은 컨테이너의 인스턴스에 페이지 스토어를 직접 import하여 별개의 접근자로 접근하도록 하면 해결할 수 있다. (`this.$pageStore = PageAStore`)
- 
