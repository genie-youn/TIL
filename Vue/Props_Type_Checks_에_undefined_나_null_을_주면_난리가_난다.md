# 들어가며
얼마 전, "스토어에 데이터가 정상적으로 저장되지 않아요" 라는 제보를 받고 디버깅을 시작했다. 놀랍게도 스토어의 문제가 아닌 스토어의 상태가 변하게 되면서 새로 마운트된 컴포넌트의 `Props Type Check`에 `undefined`를 줬었는데, 이게 에러를 뱉으면서 뷰가 뻗어버린것.

해당 컴포넌트는 `A` 타입의 프로퍼티를 받게 되어 있었는데, 이 값이 게으르게 초기화 되는 값이라 `undefined` 도 받아야 하는 `nullable` 한 값이라. `type check` 에서 "이거 A여야 하는데 `undefined`가 들어오네?" 하는 경고 메세지가 보기 싫어서 `undefined` 를 넣었다가 대참사가 일어난것.

왜 그런일이 발생했는지 정리해두려한다.

# Case A. props 로 `undefined` 와 다른 타입을 주고, 타입에 명시한 값으로 초기화를 진행했을 경우.

```vue
<template>
  <h1>{{ msg }}</h1>
</template>

<script>
export default {
  props: {
    msg: {
      type: [String, undefined],
      required: true,
    }
  },
}
</script>
```

위와 같이 `props`가 받을수 있는 타입을 `String`과 `undefined` 로 선언한 컴포넌트를 다음처럼 "test" 라는 문자열로 초기화를 하게 되면

```vue
<template>
  <img alt="Vue logo" src="src/assets/logo.png" />
  <caseA :msg="msg" />
  <button @click="setUndefined">set undefined</button>
  <button @click="setObject">set Object</button>
</template>

<script>
import caseA from './components/caseA.vue'

export default {
  name: 'App',
  components: {
    caseA
  },
  data() {
    return {
      msg: "test",
    }
  },
  methods: {
    setUndefined() {
      this.msg = undefined;
      console.log("undefined");
    },
    setObject() {
      this.msg = {
        test: "abc",
      };
      console.log("object");
    },
  },
}
</script>
```

컴포넌트와 아래 버튼들도 정상적으로 생성된다. 이후 이 값을 `undefined` 나 `Object`로 변경하게 되면 각각 'Right-hand side of 'instanceof' is not an object' 를 뱉으며 죽어버리게 된다.

> 정상적인 props type checker는 경고만 내보내지 강제하진 않는다.

# Case A. props 로 `undefined` 와 다른 타입을 주고, 타입에 명시하지 않은 값으로 초기화를 진행했을 경우.
이 경우 문제는 조금 더 심각해지는데 컴포넌트를 생성하는 과정에서 'Right-hand side of 'instanceof' is not an object' 에러가 발생하게 되고 **이후 초기화 과정이 중단되어 버려 전혀 상관이 없는 하단 버튼까지 렌더링이 되지 않는다.**

# 원인

```javascript
function assertType (value: any, type: Function): {
  valid: boolean;
  expectedType: string;
} {
  let valid
  const expectedType = getType(type)
  if (simpleCheckRE.test(expectedType)) {
    const t = typeof value
    valid = t === expectedType.toLowerCase()
    // for primitive wrapper objects
    if (!valid && t === 'object') {
      valid = value instanceof type
    }
  } else if (expectedType === 'Object') {
    valid = isPlainObject(value)
  } else if (expectedType === 'Array') {
    valid = Array.isArray(value)
  } else {
    valid = value instanceof type
  }
  return {
    valid,
    expectedType
  }
}
```


# 관련 이슈
이미 누군가 [리포팅](https://github.com/vuejs/vue/issues/1961)을 했고, [구현 PR](https://github.com/vuejs/vue/pull/9358) 이 올라와 있지만 1년반째 머지되지 않고있다.

그마저 `null` 만 포함되고 `undefined` 는 포함 안되어있는듯..
