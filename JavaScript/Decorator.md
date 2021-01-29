## 들어가며
[State of Javascript 2020](https://2020.stateofjs.com/en-US/)을 살펴보다 Draft 상태의 ECMAScript 스펙인 `Decorator` 를 발견했다. 그간 여러 기술 아티클들에서 스쳐지나가며 몇 번 접하였으나 차일피일 미루다 이제는 더 이상 미룰 수 없을것 같아 어떤 내용을 담고 있는지, 어떤 문제를 해결하고자 하는 스펙인지 알아보려한다.

## Decorator
우선 Decorator 라는 워딩을 듣고 떠오른건 GOF의 디자인패턴에 기술된 Decorator 패턴이었다.

Decorator 패턴은 객체에 동적으로 새로운 책임을 부여하기 위한 패턴으로 이를 상속으로 풀어내는 것보다 유연하다.
투명성을 만족하게 되면 객체에 새로운 책임을 부여하고 제거하는것이 자유로워진다.

이와 동일한 의미의 데코레이터 인지를 염두해두고 [스펙](https://github.com/tc39/proposal-decorators)을 읽어보았다.

---

데코레이터 함수는 객체 생성시에 데코레이팅할 대상과 부가적인 컨텍스트를 인자로 호출되고 데코레이팅 된 함수를 반환하게 됨.
데코레이팅할 대상이 메소드라면 메소드가 호출될때, 게터라면 게터가 호출될때, 세터라면 세터가 호출될 때 데코레이터 함수가 반환한 본래의 로직을 데코레이팅 한 함수를 대신 호출하여 부가적인 로직을 구현할 수 있도록 함.

```javascript
export function logged(f) {
  const name = f.name;
  function wrapped(...args) {
    console.log(`starting ${name} with arguments ${args.join(", ")}`);
    const ret = f.call(this, ...args);
    console.log(`ending ${name}`);
    return ret;
  }
  Object.defineProperty(wrapped, 'name', { value: name, configurable: true })
  return wrapped;
}
```

이러한 데코레이터가 있다고 하면, 이렇게 쓰게 되는데

```javascript
import { logged } from "./logged.mjs";

class C {
  @logged
  m(arg) {
    this.#x = arg;
  }

  @logged
  set #x(value) { }
}

new C().m(1);
// starting m with arguments 1
// starting set #x with arguments 1
// ending set #x
// ending m
```

만약 이를 바벨등으로 변환하여 현재 자바스크립트로 동작하게 만들어 본다면 우선 생성자에 `@logged` 달려 있으므로

```javascript
C.prototype.m = logged(C.prototype.m, { kind: "method", name: "m", isStatic: false });
```

이런식으로 데코레이터 함수로 생성자를 wrapping 함.
첫번째 인자로는 데코레이팅할 대상을 두번째 인자로는 추가적인 context 를부여

`logged`는 기존의 생성자를 데코레이팅한 함수를 반환해야 함.

생성자를 데코레이팅 할땐 다음과 같이 할 수 있음

`@defineElement`

```javascript
import { defineElement } from "./defineElement.mjs";

@defineElement('my-class')
class MyClass extends HTMLElement { }
```

```Javascript
// defineElement.mjs
export function defineElement(name, options) {
  return klass => { customElements.define(name, klass, options); return klass; }
}
```

## Proposal

자바스크립트의 클래스를 확장하기 위한 스펙이다.

**Decorators** 는 클래스 요소 혹은 자바스크립트의 문법이 객체를 정의하는 동안 호출되어 데코레이터가 반환하는 새로운 값으로 감싸거나 대체하는 함수이다.

Decorator는 어노테이션으로 관리될 수 있어 간편하고, 객체의 프로퍼티를 제약하지 않는다. 어노테이션으로 추가된 데코레이터는 `[Symbol.metadata]` 프로퍼티에 추가된다.

어떻게 변환을 수행할지 보다는 무엇을 데코레이팅할지 간단하게 감쌀수 있는 데코레이터를 만들기 위해 이 제안은 다음과 같은 요구조건에 초점을 맞췄다.

- 클래스의 외관에서는 데코레이터의 실행에 관련된 코드가 노출되지 않도록 하여 엔진에 의해 좀 더 최적화 될 수 있도록 한다.
- cross-file 에 대한 지식이 없이도 per-file 을 기반으로 동작할 수 있도록 구현되어야 한다.
- 추가되는 새로운 네임스페이스나 second-class value 의 타입이 없다. 데코레이터는 함수여야 한다.
