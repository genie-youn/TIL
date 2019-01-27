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

### ITEM 8. finalizer 와 cleaner 사용을 피하라

자바는 두가지 객체 소멸자를 제공한다. `finalizer` 는 예측할 수 없고 상황에 따라 위험할 수 있어 일반적으로 불필요하다. 자바 9부터는 deprecated 되었다. `cleaner` 는 `finalizer` 보다는 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로는 불필요하다.

파일이나 스레드등 종료해야 할 자원을 갖고있는 객체는 `finalizer` 나 `cleaner` 대신 `AutoClosable` 을 구현해주고 클라이언트에서 인스턴스를 다 쓰고 `close` 를 호추랗여 닫게 해야한다. 이때 각 인스턴스는 자신이 닫겼는지를 추적하는게 좋다. 즉, 이 객체가 닫겼는지를 필드에 기록하고 닫긴 후에 사용된다면 `IllegalStateExpcetion` 을 뱉게 해라

다음 두가지 상황에서 충분히 심사숙고 한 뒤 사용할 것

1. 자원의 소유자가 close 메소드를 호출하지 않는것을 방어함
2. 네이티브 피어와 연결된 객체 (GC 되지 않음) -> 이것도 웬만하면 close 를..

### ITEM 9. try-finally 보다는 try-with-resources 를

꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고 try-with-resources를 사용해라. 예외는 없다. 코드는 더 짧고 분명해지고 만들어지는 예외정보도 훨씬 유용하다.

## 모든 객체의 공통 메서드들

`Object` 의 final이 아닌 메서드 (`equals`, `hashCode`, `toString`, `clone`, `finalize`) 는 재정의를 염두해두고 설계된 것이라 재정의시 지켜줘야 하는 규약이 확실하다. 규약에 맞게 재정의 하지 않으면 규약에 맞게 정의되 있으리라 기대하고 만들어진 클래스 (`HashMap` 등) 이 오작동하게 만든다.

### ITEM 10. equals 는 일반 규약을 지켜 재정의 해라

재정의 하지 않으면 오로지 자기 자신과만 같다. 다음과 같은 경우에는 재정의 하지 말것

1. 각 인스턴스가 본질적으로 고유하다.
2. 인스턴스의 논리적 동치성을 검사할 필요가 없다.
3. 상위 클래스에서 재정의한 equals가 유효하다.
4. 클래스가 private 하거나 package-private 하고 equals 를 호출할 일이 없다.

즉 재정의 해야하는 경우는 객체 식별성이 아니라 논리적 동치성을 비교해야 할 때, 상위 클래스에서 적절히 equals를 재정의 하지 않은 value Object 이다.

---
어쨋든 둘다 직렬화 하려면 인스턴스의 모든 필드를 `transient` 로 선언하고 `readResolve` 메소드를 제공해야한다. 그렇지 않으면 역직렬화 할 때 새로운 인스턴스를 생성하게 된다.

### ITEM 13 clone 재 정의는 주의해서 진행해라.

`Cloneable` 은 복제해도 되는 클래스를 명시하는 Mixin Interface 이지만 의도한 목적을 제대로 이루지 못했음.
`clone` 메서드가 선언된것이 `Object` 재 정의를 강요할 수 없음
그마저도 `protected` 로 정의되어 있기 때문에 외부 객체에서 호출할 수 없음.
*!리플렉션을 사용하면 가능하지만 이 마저도 100%가 아니다.*

그럼에도 불구하고 널리 쓰이니까 알아두면 좋음
`clone` 메서드가 잘 동작하게 하는 구현 방법과 언제 그렇게 해야하는지, 가능한 다른 선택지에 대해 논한다.

`Cloneable` 인터페이스는 놀랍게도 `Object` 의 `protected` 메서드인  `clone()` 의 동작방식을 결정함.
`Cloneable` 을 구현한 클래스에서는 `clone()` 을 호출하면 그 객체의 필드들을 하나한 복사한 클래스를 반환하고 구현하지 않았을 경우에는 `CloneNotSupportException` 을 뱉는다. 이는 인터페이스를 굉장히 이례적으로 사용한 케이스니 따라하지 말것. 인터페이스를 구현한다는 것은 일반적으로 그 클래스가 인터페이스에서 정의한 기능을 제공한다는 의미지만 `Cloneable` 은 상위 클래스에 정의된 메서드의 동작방식을 변경한다. *!왜 이따위로 만들었지*

`Object` 의 명세에서 가져온 `clone` 의 규약은 굉장히 허술함

```
객체의 복사본을 생성해 반환한다. 일반적으로 다음과 같은 규약을 따른다
x.clone() != x
x.clone().getClass() == x.getClass()
x.clone().equals(x)

관습적으로 반환되는 객체는 super.clone() 을 호출하여 얻어야 한다. 만약 Object 를 제외한 모든 클래스와 슈퍼클래스가 이 규약을 지킨다면 x.clone().getClass() == x.getClass() 는 참이다.
```

*!super 로 호출하면 왜 처음 호출한 하위클래스가 반환되는가?*

관례상 반환된 객체와 원본 객체는 독립적이여야 한다. 이를 만족하려면 `super.clone()` 으로 얻은 객체의 필드중 하나 이상을 반환전에 수정해야 함. 강제성이 없다는 것을 제외하면 생성자 체인과 같다.
`super.clone()` 이 아닌 자기 자신의 생성자를 호출하면 하위 객체에서 `clone()` 을 호출했을 때 원하는 타입을 받지 못한다. 사실 말도 안되는게 `super.clone()` 으로 `Object.clone()` 의 동작방식에 기대지 않을 꺼면 `Cloneable` 을 구현할 필요 자체가 없다.

> 공변 반환 타이핑 covariant return type : 재정의한 메서드의 반환타입은 상위클래스가 반환하는 타입의 하위타입일 수 있다. super.~ 로 호출한 아이는 하위타입으로 캐스팅이 가능하다.

`clone` 의 구현이 가변 객체를 참조하면 재앙이 벌어짐
`clone` 메소드는 사실상 생성자와 같은 효과를 내므로 원본 객체에 아무런 영향을 끼치지 않는 동시에 객체의 불변식을 보장해야한다.

```Java
@Override public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements = elements.clone();
    return result;
  } catch (CloneNotSupportException e) {
    log.error(e);
    // re-throwing
  }
}
```

배열의 `clone()` 은 형변환이 필요없다. 런타임타입/컴파일타임타입 모두 원본의 타입을 반환하기 때문 (`Object.clone()` 이 아닌 __Array__ 의 `clone()` 을 사용한다.)
*!얘도 진작 얘처럼 만들지? 자바에서 Native한 코드인가*

여기서 `elements` 가 `final` 이면 이 방법은 동작하지 않기 때문에 `Cloneable` 의 아키텍처는 '가변 객체를 참조하는 필드는 `final` 로 선언하라' 는 일반적인 용법과 충돌한다.

깊은 복사를 할때, `clone` 을 재귀적으로 호출하는 것만으로는 부족한 경우가 있음 (HashTable 의 `clone` 메소드)
`Entry // LinkedList` 의 길이가 너무 길어지면 재귀호출 중 스택오버플로우가 발생, 이런 경우 순회로 깊은 복사를 구현할 것

또다른 방법은 `super.clone()` 으로 초기상태를 결정한 다음 원본 객체와 똑같은 상태로 만드는 고수준 API들을 만드는것이다. (같은 값으로 set 하는것)

생성자에서는 재정의 될 수 있는 메소드를 호출하지 말아야 하는데, `clone` 메서드에서도 마찬가지.
따라서 앞서 언급한 메서드는 `final` 이거나 `private` 일 것이다.

재정의한 값을 `throw`을 없앨것

상속용 클래스는 `Cloneable`을 구현하지 말것 *!복사하는게 의미가 없어서? 하위클래스에게 위임하는게 맞아서?*

근데 이런거 복잡하니까 복사 생성자 (변환 생성자) / 복사 팩터리 (변환 팩터리) 를 만드는게 더 낫다.

```Java
public Yum (Yum yum) {...}
public Yum newInstance(Yum yum) {...}
```

왜냐하면
1. 언어모순적이고 위험천만한 (생성자를 사용하지 않는) 객체 생성메커니즘이 없고
2. 엉성하게 문서화된 규약에 기대지도 않고
3. 정상적인 `final` 용법과도 충돌하지 않으며
4. 불필요한 `checked-exception` 을 뱉지도 않고
5. 형변환도 필요없다.

더욱이 인터페이스를 인자로 받음으로써 유연한 변환도 제공할 수 있게 된다.
범용 컬렉션들이 `Collection` 이나 `Map` 을 인자로 받는 생성자를 사용하는 이유

##### 결론

배열 외에 `clone` 은 쓰지마라

### ITEM 14. Comparable 을 구현할 지 고려해라.

`equals` 와 비슷하지만 동치성 비교외에 순서까지 비교할 수 있다.
`Comparable` 을 구현했다는 것은 자연적인 순서가 있다는 것을 의미한다.

`Comparable` 을 구현하면 이 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있다. 좁쌀만한 노력으로 코끼리만한 효과를 누릴수 있는것이다. 자연적인 순서를 갖는 값 클래스라면 꼭 구현하도록 하자.

다음과 같은 규약을 지켜야 한다.
1. 역 성립 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
2-1. 추이성 성립 x.compareTo(y) > 0 && y.compareTo(z) -> x.compareTo(z) > 0
2-2. x.compareTo(y) == 0 && y.compareTo(z) == 0 -> x.compareTo(z) == 0
3. x.compareTo(y) == 0 -> x.equals(y) = true

3번은 필수는 아니지만 만약 `Comparable`을 구현하는데 이게 성립되지 않을 경우 꼭 **클래스의 순서는 equals 메서드와 일치하지 않는다** 를 명시해야한다.

마지막 규약은 되도록 지키는게 좋은게 정렬된 컬렉션들은 `equals` 메서드의 규약을 따른다고 되어 있지만 실제로는 동치성을 비교할 때 `equals` 대신 `compareTo` 를 사용하는 경우가 많다.

TreeMap.java
```Java
if (cpr != null) {
  do {
    parent = t;
    cmp = cpr.compare(key, t.key);
    if (cmp < 0)
    t = t.left;
    else if (cmp > 0)
    t = t.right;
    else
    return t.setValue(value);
  } while (t != null);
}
```

Collections.java
```Java
int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
  int low = 0;
  int high = list.size()-1;

  while (low <= high) {
    int mid = (low + high) >>> 1;
    Comparable<? super T> midVal = list.get(mid);
    int cmp = midVal.compareTo(key);

    if (cmp < 0)
    low = mid + 1;
    else if (cmp > 0)
    high = mid - 1;
    else
    return mid; // key found
  }
  return -(low + 1);  // key not found
}
```

구현은 다음처럼 자바에서 제공하는 비교자나 직접 비교자를 구현하면 된다.

```Java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
  public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.comapre(s, cis.s);
  }
}
```

```Java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(areaCode, pn.areaCode);
  if (result == 0) {
    result = Short.comapre(prefix, pn.prefix);
    if (result == 0) {
      result = Short.comapre(lineNum, pn.lineNum)
    }
  }
  return result;
}
```

아니면 함수형 인터페이스 `Comparator` 를 구현하면 좀 더 선언적으로 가능
```Java
private static final Comparator<PhoneNumber> COMPARATOR =
  comparingInt((PhoneNumber pn) -> pn.areaCode)
    .thenComparingInt(pn -> pn.prefix)
    .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
  return COMPARATOR.compare(this, pn);
}

```

관계 연산자를 (> , <, ==) 를 직접 쓰는것보다 (거추장스럽고 오류를 유발한다. 맨날 헷갈림) 될 수 있으면 박싱 클래스의 `compare` 정적 메서드를 활용해라.

비교자를 구현할 때는 성능을 위해 핵심적인 요소부터 비교할것

값의 차 (-) 를 기준으로 `Comparator` 를 구현하지 말것.
정수 오버플로우와 부동소수점 계산 바식에 따른 오류를 낼 수 있다.

다음 두가지 방법을 써라
```Java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return Integer.compare(o1.hashCode(), o2.hashCode());
  }
};

static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

##### 결론

순서를 고려해야 하는 값 클래스를 작성한다면 꼭 `Comparable` 인터페이스를 구현하여, 그 인스턴스를 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다. `compareTo` 메서드에서 필드의 값을 비교할 때 <와 >연산자는 쓰지 말아야한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 `compare` 메서드나 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
