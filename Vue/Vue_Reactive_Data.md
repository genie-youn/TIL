# Vue 와 반응형 (Data)

얼마 전 특정 유저의 상태에 따라 알림 레이어를 노출해야 하는 스펙을 구현중에 의도한 대로 동작하지 않아 한참을 해맸다. Vue 의 반응형이 어떻게 동작하는지 제대로 이해하고 있지 않아 생긴 문제였고 이 기회에 Vue와 반응형에 대하여 정리하려 한다.

### 반응형?

Vue에서 반응형 ^Reactivity^ 이란 Vue 인스턴스에 `data` 프로퍼티로 자바스크립트 객체를 전달하면 이 객체의 변경을 감지하여 전파하는것을 일컫는다.

```html
<template>
  <div>
    {{ message }}
  </div>
</template>
<script>
export default {
  data() {
    return {
      message: 'hello world'
    }
  }
}
</script>
```

위와 같은 컴포넌트가 존재할 때 `message` 의 값이 바뀌면 `div` 내부의 문자열도 이에 **반응** 하여 변경되게 된다.

### 반응형이 적용되는 과정

Vue 인스턴스에서 반응형은 이벤트와 라이프사이클이 초기화 된 이후 설정된다. (`beforeCreate` 훅과 `created` 훅 사이)

core/instance/init.js
```javascript
callHook(vm, 'beforeCreate')
initInjections(vm)
initState(vm) // 여기서 반응형 설정이 이루어진다.
initProvide(vm)
callHook(vm, 'created')
```

해당 시점에 `props`, `methods`, `data`, `computed`, `watch` 에 대한 초기화가 이루어 진다.

`data` 객체에 `__ob__` 라는 이름의 옵저버 객체를 프로퍼티로 등록한 후 각 `key` 들을 순회하면서 해당 프로퍼티가 수정 가능하다면 (`configurable`) `reactiveGetter`, `reactiveSetter` 를 각각 프로퍼티의 getter 와 setter 로 등록한다. 또한 각각의 프로퍼티는 클로저로 `Dep` 이라는 다수의 Subscriber 를 가질 수 있는 객체를 가지게 되고 템플릿의 디렉티브는 이 `Dep` 객체를 구독하게 된다.

core/observer/index.js defineReactive()
```javascript
const dep = new Dep()

Object.defineProperty(obj, key, {
  enumerable: true,
  configurable: true,
  get: function reactiveGetter () {
    // ...
  },
  set: function reactiveSetter (newVal) {
    // ...
  }
})
```

이후 `mount` 하는 과정에서 새로운 `Watcher` (Subscriber) 객체를 생성하고 뷰 인스턴스를 업데이트 하는 `vm._update()` 함수를 변경이 생겼을 때 실행할 함수로 넣어둔다. `vm._update()` 는 `vm._render()` 함수를 실행한 결과인 `vnode` 객체를 받아서 DOM 을 업데이트한다.

```javascript

updateComponent = () => {
  vm._update(vm._render(), hydrating)
}

new Watcher(vm, updateComponent, noop, {
  before () {
    if (vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'beforeUpdate')
    }
  }
}, true /* isRenderWatcher */)

// Watcher constructor
constructor (
  vm: Component,
  expOrFn: string | Function,
  cb: Function,
  options?: ?Object,
  isRenderWatcher?: boolean
) {
  this.vm = vm
  if (isRenderWatcher) {
    vm._watcher = this
  }
}
```

이 객체를 `vm._watcher` 프로퍼티에 등록하고 `Watcher.get()` 을 통해 뷰 인스턴스를 한번 업데이트를 한다. 렌더링 함수를 호출해 `template` 내의 구문을 컴파일하고 템플릿의 디렉티브에서 참조하고 있는 `data` 프로퍼티의 `getter`를 호출하게 된다. `getter` 로 등록된 `reactiveGetter`를 자세히 보면

```javascript
get: function reactiveGetter () {
  const value = getter ? getter.call(obj) : val
  if (Dep.target) {
    dep.depend()
    if (childOb) {
      childOb.dep.depend()
      if (Array.isArray(value)) {
        dependArray(value)
      }
    }
  }
  return value
},
```

`Dep.target` 에는 앞서 뷰 인스턴스를 업데이트 하는 함수를 담고있는 `Watcher` 객체가 들어있다. 그래서 프로퍼티가 참조하고 있는 `Dep` 객체에 `depend()` 함수를 호출하게 된다.

```javascript
// Dep (Observable) 객체의 함수다.
depend () {
  if (Dep.target) {
    Dep.target.addDep(this)
  }
}

// 앞서 등록된 `Watcher` (Subscriber) 객체의 함수다
addDep (dep: Dep) {
  const id = dep.id
  if (!this.newDepIds.has(id)) {
    this.newDepIds.add(id)
    this.newDeps.push(dep)
    if (!this.depIds.has(id)) {
      dep.addSub(this)
    }
  }
}
```

위와 같은 핑퐁을 거쳐 해당 프로퍼티의 `Dep(Observable)` 을 뷰 인스턴스를 업데이트 하는 함수를 가진 `Wathcer(Subscriber)` 의 구독이 등록되게 된다.

> 이런식으로 Dep 과 Wathcer 가 서로 핑퐁을 하면서 구독 등록의 과정이 이루어 지는데 중간에 this 가 계속 인자로 주어지다 보니 코드가 읽기가 많이 헷갈렸던것 같다..

이후 `message` 의 값을 변경하게 되면 setter 로 등록해 두었던 `reactiveSetter` 함수가 호출되게 된다.

```javascript
set: function reactiveSetter (newVal) {
  const value = getter ? getter.call(obj) : val
  /* eslint-disable no-self-compare */
  if (newVal === value || (newVal !== newVal && value !== value)) {
    return
  }
  /* eslint-enable no-self-compare */
  if (process.env.NODE_ENV !== 'production' && customSetter) {
    customSetter()
  }
  // #7981: for accessor properties without setter
  if (getter && !setter) return
  if (setter) {
    setter.call(obj, newVal)
  } else {
    val = newVal
  }
  childOb = !shallow && observe(newVal)
  dep.notify()
}
```

새로 받은 값을 set 하고 나면 프로퍼티의 `dep (Observable)` 객체에 `notify` 를 호출하여 상태의 전파를 일으킨다.

이 객체를 구독하고 있던 `Watcher` 객체가 `updateComponent` 함수를 실행시켜 변경된 `message` 의 값을 읽어가고 virtual dom 을 계산하고 변경된 부분을 다시 렌더링 함으로써 `message` 의 값에 템플릿 내 디렉티브가 반응형으로 동작하게 된다.

객체를 초기화 하는 과정에서 `data` 객체에 있는 프로퍼티를 순회하며 반응형을 설정하기 때문에 이 시점에 존재하지 않았던 프로퍼티에 대해서는 반응형으로 동작하지 않게 된다. 초기화 이후 `data` 객체에 프로퍼티를 아무리 추가해봤자 변경을 감지하지 못한다는 이야기다. 그렇기 때문에 빈 값으로라도 초기에 선언하여 인스턴스를 초기화 해야 하는 것이다.

### 놓치기 쉬운것들

#### 중첩된 객체

만약 반응형으로 동작해야할 값이 중첩된 객체라면 `data` 객체의 key 를 순회하며 반응형을 설정하는 과정에서 내부 객체를 재귀적으로 훑으며 반응형을 설정하므로 **중첩된 객체의 프로퍼티 또한 빈값으로라도 선언되어 있어야 한다**

코드를 보다가 이부분이 좀 헷갈렸는데 분명 `reactiveSetter` 내부에도 새로 받은 값에 대해서 재귀적으로 반응형하는 부분이 있었기 때문에 그렇다면 루트 수준에만 존재하면 자동으로 반응형이 되지 않나? 하고 생각했었다. 예를들어

```javascript
var vm = new Vue({
  data: {
    a: {
      b: 1,
    }
  }
})
```

다음과 같은 컴포넌트가 있을 때 `a` 에는 이미 반응형이 설정되어 있으니, `vm.a.c = {d : 1}` 라고 주어졌을때 `a` 의 `reactiveSetter` 가 호출될것이고, 그러면 `{d : 1}` 객체에 대해서도 반응형이 잡히겠지. 하고 생각한 것인데 이는 기존에 알고있던 중첩된 객체의 프로퍼티 또한 빈값으로라도 선언이 되어 있어야 한다는 사실과 어긋나는 것이였다.

그 이유는 단순하게 자바스크립트에서 `a.c = 4` 를 주었을 때 a 의 setter 가 호출되지 않는다. c 에 대한 setter 를 호출할뿐..

위 상황에서는 a 와 b 모두 재귀적으로 `reactiveSetter` 가 등록되어 있으니 `data.a.b = 3` 은 반응형을 타겠지만 `data.a.c = 3` 은 c 에 대한 `reactiveSetter` 가 등록되어 있지 않으니 반응형을 타지 않는다.

루트 수준 (`data` 객체의 직접적인 프로퍼티) 에 반응형을 추가하는 방법은 존재하지 않지만, 초기화시 존재했던 중첩된 프로퍼티 객체에 새로운 프로퍼티를 추가할 수 있는 API 는 제공한다.

```javascript
var vm = new Vue({
  data: {
    a: {
      b: 1,
    }
  }
})

Vue.set(vm.a, 'c', 2);
vm.$set(vm.a, 'c', 2); // 또는
```

마찬가지로 다음과 같은 케이스도 반응형으로 동작하지 않는다.

```javascript
Object.assign(this.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

setter 를 명시적으로 호출해주어야 한다.

```javascript
this.userProfile = Object.assign({}, this.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
