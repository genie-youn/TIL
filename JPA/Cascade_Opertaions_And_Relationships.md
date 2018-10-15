# Cascade Operaions and Relationships



1:N 의 관계를 갖는 도메인의 부모를 바꿔주는 로직을 구현하다가 정상적으로 변경했음에도 불구하고 트랜잭션이 끝나는 시점에 해당 객체가 DB에서 삭제되어 버리는 황당한 경험을 했다. 공부도 안하고 JPA를 쓰는 댓가를 톡톡히 치르는 요즘 JPA 의 `Cascade`  에 대하여 정리하려고 한다.



## Cascade Operaions and Relationships



관계를 사용하는 엔티티는 종종 다른 엔티티의 존재에 의존한다. 예를들어, 주문의 일부인 상품은 만약 주문이 삭제되면 주문에 속해있던 상품 또한 함께 삭제되어야 한다. 이것은 삭제 종속 관계에 의하여 호출된다.



`javax.persistence.CascadeType` enum 타입은 관계 어노테이션들의 `cascade` 속성으로 주입될 종속 연산자를 정의한다.



| Cascade 연산자 | 설명                                                         |
| -------------- | ------------------------------------------------------------ |
| ALL            | 모든 종속 연산자들이 부모 엔티티와 연관된 엔티티에 적용된다. <br />`All` 은 `cascade={DETACH, MERGE, PERSIST, REFRESH, REMOVE}` 와 일치한다. |
| DETACH         | 만약 부모 엔티티가 영속성 컨텍스트에서 분리되면, 연관된 엔티티 또한 분리된다. |
| MERGE          | 만약 부모 엔티티가 영속성 컨텍스트에 합쳐지면, 연관된 엔티티 또한 합쳐진다. |
| PERSIST        | 만약 부모 엔티티가 영속성 컨텍스트에 영속화되면, 연관된 엔티티 또한 영속화 된다. |
| REFRESH        | 만약 부모 엔티티가 현재 영속성 컨텍스트에서 갱신되면, 연관된 엔티티 또한 갱신된다. |
| REMOVE         | 만약 부모 엔티티가 현재 영속성 컨텍스트에서 삭제되면, 연관된 엔티티 또한 삭제된다. |



삭제 종속 관계는 `cascade=REMOVE` 속성을 `@OneToOne` 또는 `@OneToMany` 어노테이션에 명시함으로써 정의할 수 있다.

```java 
@OneToMany(cascade=REMOVE, mappedBy="customer")
public Set<Order> getOrders() { return orders; }
```



### Reference

https://docs.oracle.com/javaee/6/tutorial/doc/bnbqa.html#bnbqm



[^ 1]: DETACH, MERGE, PERSIST... 흠..

