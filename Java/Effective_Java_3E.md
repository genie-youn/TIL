## 객체생성과 파괴

### ITEM 1. 생성자대신 정적 팩터리 메서드를 고려하라

클라이언트가 클래스의 인스턴스를 생성하는 전통적인 방법은 public 생성자를 이용하는것.
이것 말고도 인스턴스를 생성해 반환하는 팩터리 메서드를 만드는 방법도 있다.

팩터리 메서드를 만들면 생성자를 만드는것 보다 다섯가지 장점을 가진다.

1. 이름을 가질 수 있다. (생성자는 하나의 시그니처로 하나만 만들 수 있다.)
2. 호출할 때마다 인스턴스를 새로 생성하지 않아도 된다. (불변 객체를 미리 생성해두거나, 캐싱해 둘 수 있다. -> 인스턴스를 통제할 수 있다. 싱글턴이라던지)
3. 반환타입의 하위타입도 반환 할 수 있다. (`Collections`를 생각해볼 것)
4. 입력 매개변수에 따라 매번 다른 클래스를 반환할 수 있다. (EnumSet. 클라이언트는 팩터리 메서드가 건네주는 객체가 어떤 클래스의 인스턴지인지 알 수도, 알 필요도 없다. 그저 EnumSet의 하위클래스이면 된다.)
5. 정적 팩터리 메서드를 작성하는 시점에 반환할 객체의 클래스가 존재하지 않아도 된다. (서비스 제공자 프레임워크의 서비스 접근 API e.g. JDBC 서비스 제공자 프레임워크는 서비스 인터페이스 (`Connection`) 제공자 등록 API (`DriverManager.registerDriver`) 서비스 접근 API (`DriverManager.getConnection`) 으로 이루어진다.)

단점은 다음과 같다.

1. 상속을 하면 public 이나 package-protected 생성자가 필요하니 정적 팩터리 메서드만 제공하는 클래스는 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 찾기가 어렵다. (요즘은 IDEA가..)

흔히 사용하는 컨벤션은 다음과 같다.

from : 하나의 매개변수를 받아 해당 타입의 인스턴스를 반환하는 형변환 메서드

of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드

valueOf : from + of

instance / getInstance : (매개변수를 받으면) 매개변수로 명시한 인스턴스를 반환하지만 같은 인스턴스임을 보장하진 않는다.

create / newInstance : 매번 새로운 인스턴스임을 보장한다.

get*Type* : getInstance 와 같으나, 반환할 클래스가 아닌 다른 클래스에 팩터리 메서드를 만들때 사용

```java
e.g. FileStorage fs = Files.getFileStorage(~~);
```

new*Type* : newInstance 와 같으나, 반환할 클래스가 아닌 다른 클래스에 펙터리 메서드를 만들때 사용

```java
BufferReader br = Files.newBufferReader(~~);
```

##### 결론
상황에 따라 생성자와 정적 팩터리 메서드를 사용하되, 일반적인 경우 정적 팩터리 메서드를 사용하는 것이 유리할 때가 많다.

### ITEM 2. 생성자에 매개변수가 많으면 빌더를 고려해라

점층적 생성자 패턴 :
  생성자에 variation 을 주어 다양하게 만든다.
  읽기가 힘듬

자바빈즈 패턴 :
  파라미터가 없는 생성자로 객체를 만들고 `setter` 를 호출하여 값을 설정한다.
  필요한 모든 `setter`가 호추링 끝나기 전까지 객체는 일관성 [^consistency] 이 무너진 상태에 놓인다.
  불변객체를 만들수 없어 스레드 안전성을 확보하기가 번거롭다.

그래서 빌더패턴을 써야한다.
계층적으로 설계된 클래스와 사용하면 더 진가를 발휘한다.

```java
public abstract class Pizza {
  abstract static class Builder<T extends Builder<T>> {
    public T addTopping(Topping topping) {
      toppings.add(Objects.requireNotNull(topping));
      return self();
    }

    abstract Pizza build();
    // 하위클래스는 이 메소드를 재정의하여
    // this를 반환하게 해야함
    protected abstract T self();
  }

  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone();
  }
}
```
여기서 `self()` 는 self 타입이 없는 자바에서 하위타입에서도 형변환을 하지 않고 메서드 체인을 만들기 위한 방법으로 `Simulated Self-Type` 이라고 부른다.

사용하는 클라이언트의 코드는 다음과 같다.

```java
public class NyPizza extends Pizza {
  public static Builder extends Pizza.Builder<Builder> {
    @Override
    public NyPizza build() {
      return new NyPizza(this);
    }
    @Override
    public protected Builder self() {
      return this;
    }
  }
}
```

#### 결론
생성자나 정적 팩터리 메서드가 처리해야할 인자가 많다면 빌더패턴이 좋은 해답이다.
매개변수중 다수가 필수가 아니거나 타입이 같다면 더욱 진가를 발휘한다.
빌더는 점층적 생성자보다 코드를 읽고 쓰기가 훨씬 수월하고 자바 빈즈보다 훨씬 안전하다.

### ITEM 3. private 생성자나 열거타입으로 싱글턴을 보장하라

싱글턴을 만드는 방법은 세가지다

```java
public static final Elvis INSTANCE = new Elvis();
private Elvis() {...}
```

API상 싱글턴임이 명확하고 간결하다. 리플렉션을 방어해야 한다.

```java
private static final Elvis INSTANCE = new Elvis();
private Elvis() {...}
public static Elvis getInstance() {
  return INSTANCE;
}
```

유연하다. API를 바꾸지 않아도 싱글턴이 아니게 할 수 있다. 정적 펙터리 메서드를 `Supplier`로 사용할 수 있다.
리플렉션을 방어해야한다.

어쨋든 둘자 직렬화 하려면 인스턴스의 모든 필드를 `transient` 로 선언하고 `readResolve` 메소드를 제공해야한다. 그렇지 않으면 역직렬화 할 때 새로운 인스턴스를 생성하게 된다.

위 단점을 모두 해결할 수 있는 방법이 원소가 하나인 Enum 타입의 객체를 싱글턴으로 사용하는 것이다.

대부분 상황에서 제일 좋은 해결책이 될 수 있지만, Enum 외의 클래스를 상속받을 수 없다.

### ITEM 4. 인스턴스화를 막으려면 private 생성자를 사용해라

### ITEM 5. 자원을 직접 명시하지 말고 의존객체 주입을 사용해라.

클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴이나 정적 유틸 클래스는 적합하지 않다.
이 자원들을 클래스가 직접 만들게 해서도 않된다.
대신 클래스 자원을 (혹은 그 자원을 생성하는 팩터리를) 생성자에 (혹은 정적 팩터리 메서드나 빌더에) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재활용성, 테스트 용이성을 기가 막히게 향상시켜준다.

### ITEM 6. 불필요한 객체 생성을 피해라

단순히 임시 객체를 만들지 말라는 소리가 아니라 객체의 재사용성을 적극 고려해라
생성자는 호출할 때 마다 새로운 객체를 반환하지만, 정적 팩터리 메서드는 그렇지 않다.
비싼 객체는 캐싱을 고려해라.
박싱된 기본타입 보다는 기본타입을 활용하고, 의도치 않은 오토박싱이 개입하지 않도록 유의해라.
이 아이템을 객체생성은 비싸니 피해라로 오해하진 말것. 기본적으로 요즘 JVM은 별다른 일을 하지 않는 객체를 만들고 회수하는 일은 크게 부담되지 않는다.
> 개인적으로는 읽기 좋은 코드를 해치지 않는 선에서, 성능은 필요할때 튜닝하는게 좋은것 같다.

### ITEM 7. 다 쓴 객체 참조를 해제하라

다 쓴 참조는 null 로 해제할것. 단, 자기 메모리를 직접 관리하는 예외적인 경우에만. 기본적으로는 스코프 밖으로 밀어내는게 맞다.
예를 들면 스택이나 캐시에 넣어놓고 까먹는다던지, 리스터나 콜백을 등록하고 해제를 안한다던지.. 이런 경우에는 `WeakHashMap` 이 해결책이 될 수 있다.


--- 봐야해
어쨋든 둘자 직렬화 하려면 인스턴스의 모든 필드를 `transient` 로 선언하고 `readResolve` 메소드를 제공해야한다. 그렇지 않으면 역직렬화 할 때 새로운 인스턴스를 생성하게 된다.
