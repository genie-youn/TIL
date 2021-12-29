# ref와 reactive의 차이에 대해서 알아보자

[Vue Composition API](https://github.com/vuejs/composition-api)와 Vue3에서 반응형을 설정하는 방법으로는 두가지가 존재한다.

`ref`와 `reactive`를 사용하는것인데, 이 둘 간의 어떤 차이가 있는지 정리하려한다.

우선 어떤 변수든 `ref` 함수를 통해 반응형을 설정할 수 있다.

```javascript
import { ref } from 'vue'

const counter = ref(0)
```

`ref` 함수는 인자로 받은 변수를 `value`로 갖는 랩핑객체를 반환한다. 이 객체는 반응형 변수의 값에 접근하거나 상태를 변경할 수 있다.

```javascript
import { ref } from 'vue'

const counter = ref(0)

console.log(counter) // { value: 0 }
console.log(counter.value) // 0

counter.value++
console.log(counter.value) // 1
```

어떤 값을 객체로 감싸는 일이 불필요해 보일 수 있지만 이는 자바스크립트의 각기 다른 데이터타입들이 일관성있게 동작하도록 하기 위해 꼭 필요하다.
자바스크립트의 `Number`나 `String`같은 원시 타입은 값으로 전달되지만 그 외에는 참조로 전달되기 때문이다.

어떠한 값이든 래퍼 객체로 한번 감싸는것은 애플리케이션 전체에서 반응형을 잃어버릴것을 걱정하지 않고 공유하여 사용할 수 있도록 해준다.

> 원시 타입은 값으로 전달되니 이 변수에 반응형을 걸어봤자 파라미터로 전달되면 없어지겟네

다르게 설명하면 `ref`는 값에 대한 반응형 참조를 생성하는것이다. 참조와 함께 동작하는 컨셉은 Composition API에서 자주 사용된다.

[Reactive API](https://v3.vuejs.org/api/basic-reactivity.html)

Reactive
객체의 반응형 복사본을 반환함.

```javascript
const obj = reactive({ count: 0 })
```

깊게 복사된다. 즉 중첩된 모든 프로퍼티에 반응형을 건다. ES2015 Proxy에 기반을 둔 구현으로 반환된 프록시 객체는 원본 객체와 동일하지 않다.
반환된 반응형 프록시 객체만을 사용하고 원본 객체는 사용하지 않는것이 좋다.

`reactive`는 모든 중첩된 refs 를 벗겨내지만 ref 반응형은 그대로 유지된다.

```javascript
const count = ref(1)
const obj = reactive({ count })

// ref will be unwrapped
console.log(obj.count === count.value) // true

// it will update `obj.count`
count.value++
console.log(count.value) // 2
console.log(obj.count) // 2

// it will also update `count` ref
obj.count++
console.log(obj.count) // 3
console.log(count.value) // 3
```

마찬가지로 ref를 `reactive`로 반환된 반응형 프록시 객체의 프로퍼티로 할당해도 ref가 자동으로 벗겨진다.

```javascript
const count = ref(1)
const obj = reactive({})

obj.count = count

console.log(obj.count) // 1
console.log(obj.count === count.value) // true
```

### `readonly`
`readonly`는 파라미터로 주어진 객체 (반응형 객체든 평범한 자바스크립트 객체이든) 읽기 전용인 프록시 객체를 반환하며 이는 deep 하게 적용된다.

```javascript
const original = reactive({ count: 0 })

const copy = readonly(original)

watchEffect(() => {
  // works for reactivity tracking
  console.log(copy.count)
})

// mutating original will trigger watchers relying on the copy
original.count++

// mutating the copy will fail and result in a warning
copy.count++ // warning!
```

마찬가지로 파라미터로 주어진 객체가 `ref`로 반환된 반응형 프록시 객체라면 unwrapped 됨.

```javascript
const raw = {
  count: ref(123)
}

const copy = readonly(raw)

console.log(raw.count.value) // 123
console.log(copy.count) // 123
```

`isProxy`

`isReactive`

`isReadonly`

`toRaw`
`reactive`나 `readonly` 프록시의 원본 객체를 반환.

`markRaw`
찍어놓으면 `reactive`를 걸어도 반응형이 안걸림

```javascript
const foo = markRaw({})
console.log(isReactive(reactive(foo))) // false

// also works when nested inside other reactive objects
const bar = reactive({ foo })
console.log(isReactive(bar.foo)) // false
```

`shallowReactive`
`reactive`와 다르게 얕은 반응형 프록시 객체를 반환함

```javascript
const state = shallowReactive({
  foo: 1,
  nested: {
    bar: 2
  }
})

// mutating state's own properties is reactive
state.foo++
// ...but does not convert nested objects
isReactive(state.nested) // false
state.nested.bar++ // non-reactive
```

`ref`는 벗겨지지 않음

`shallowReadonly`
마찬가지로 `readonly`와 다르게 얕은 읽기 전용 프록시 객체를 반환함.

```javascript
const state = shallowReadonly({
  foo: 1,
  nested: {
    bar: 2
  }
})

// mutating state's own properties will fail
state.foo++
// ...but works on nested objects
isReadonly(state.nested) // false
state.nested.bar++ // works
```

마찬가지로 `ref`는 벗겨지지 않음


그럼 ref는 값이 대상이고 reactive는 객체가 대상이다.. 정도인가?

```vue
<template>
  <div>hi</div>
  <div>{{ a.a }}</div>
  <div>{{ a.b }}</div>
  <div>{{ a.c.d }}</div>
  <div>{{ a.c.e.f }}</div>
  <div>{{ a.c.e.g }}</div>
  <div>---------------------</div>
  <div>{{ b.a }}</div>
  <div>{{ b.b }}</div>
  <div>{{ b.c.d }}</div>
  <div>{{ b.c.e.f }}</div>
  <div>{{ b.c.e.g }}</div>
</template>

<script>
import { reactive, ref } from "vue";

export default {
  name: "PageAContainer",
  setup() {
    const a = reactive({
      a: 0,
      b: 0,
      c: {
        d: 0,
        e: {
          f: 0,
          g: 0,
        },
      },
    });
    const b = ref({
      a: 0,
      b: 0,
      c: {
        d: 0,
        e: {
          f: 0,
          g: 0,
        },
      },
    });

    setInterval(() => {
      a.a++;
      a.b++;
      a.c.d++;
      a.c.e.f++;
      a.c.e.g++;

      b.value.a++;
      b.value.b++;
      b.value.c.d++;
      b.value.c.e.f++;
      b.value.c.e.g++;
    }, 1000);
    return {
      a,
      b,
    };
  },
};
</script>

<style scoped></style>
```

로 테스트 해보면 동일하게 동작한다. 즉 value를 접근해야만 한다는것 말곤 차이가 없음 둘다 deep 하게 걸리고..
reactive가 반응형 프록시를 생성하는 기본이고, ref는 프리미티브 타입은 파라미터로 전달될때 pass by value로 전달되기 때문에 이를 회피하기 위해서 값을 한번 감싸는 wrapper 객체를 생성한다. 맞나? 그래서 참조를 생성한다는 의미로 ref로 정의한거고.?

맞네 ref내부에서 reactive를 호출한다음 이를 value로 갖는 객체를 반환한다.

```typescript
export function ref(raw?: unknown) {
  if (isRef(raw)) {
    return raw
  }

  const value = reactive({ [RefKey]: raw })
  return createRef({
    get: () => value[RefKey] as any,
    set: (v) => ((value[RefKey] as any) = v),
  })
}

export function createRef<T>(
  options: RefOption<T>,
  isReadonly = false,
  isComputed = false
): RefImpl<T> {
  const r = new RefImpl<T>(options)

  // add effect to differentiate refs from computed
  if (isComputed) (r as ComputedRef<T>).effect = true

  // seal the ref, this could prevent ref from being observed
  // It's safe to seal the ref, since we really shouldn't extend it.
  // related issues: #79
  const sealed = Object.seal(r)

  if (isReadonly) readonlySet.set(sealed, true)

  return sealed
}

export class RefImpl<T> implements Ref<T> {
  readonly [_refBrand]!: true
  public value!: T
  constructor({ get, set }: RefOption<T>) {
    proxy(this, 'value', {
      get,
      set,
    })
  }
}
```

