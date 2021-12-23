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


