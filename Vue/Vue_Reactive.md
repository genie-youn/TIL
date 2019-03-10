# Vue 와 반응형

얼마 전 특정 유저의 상태에 따라 알림 레이어를 노출해야 하는 스펙을 구현중에 의도한 대로 동작하지 않아 한참을 해맸다. Vue 의 반응형이 어떻게 동작하는지, 제대로 이해하고 있지 않아 생긴 문제였고 이 기회에 Vue와 반응형에 대하여 정리하려 한다.

### 반응형?

Vue에서 반응형 ^Reactivity^ 이란 Vue 인스턴스에 `data` 옵션으로 자바스크립트 객체를 전달하면 이 객체의 변경을 감지하여 전파하는것을 일컫는다.

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

Vue 인스턴스에서 반응형은 이벤트와 라이프사이클이 초기화 된 이후 설정된다.

```javascript
// expose real self
vm._self = vm
initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```
