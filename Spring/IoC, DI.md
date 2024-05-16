IoC(Inversion of Control) 는 객체의 생성과 의존성 관리를 객체 코드가 아닌 외부의 컨테이너에서 제어하도록 하는 설계 원칙 -> 객체는 자신이 직접 다른 객체를 생성하거나 의존 관계를 설정하지 않고, 외부 컨테이너가 객체 간의 관계를 설정하고 관리함

DI(Dependency Injection)
- DI 는 객체가 필요한 의존 객체를 외부에서 주입하는 방식
- 생성자 주입, 세터 주입, 필드 주입이 있음


1. **생성자 주입**
```java
public class OrderService {
    private final DiscountPolicy discountPolicy;

    public OrderService(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```
- 의존성을 생성자의 매개변수로 전달해 주입하는 방식
- 의존성 주입 후 객체의 불변 상태가 유지됨. 객체가 생성된 후에는 의존성이 변경되지 않도록 보장함
- 객체가 생성될 때 필수 의존성이 모두 주입되도록 강제할 수 있음. 필수 의존성 보장
- 의존성이 많아질 경우 생성자의 매개변수가 많아져 코드가 복잡해질 수 있음

2. **세터 주입**
```java
public class OrderService {
    private DiscountPolicy discountPolicy;

    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```
- 세터 메서드를 통해 의존성을 주입하는 방식
- 의존성을 선택적으로 주입할 수 있어, 일부 의존성만 주입할 수 있음
- 객체가 생성된 후에도 의존성을 변경할 수 있어 유연함
- 객체가 완전히 초기화되기 전에 메서드가 호출되면 문제 생길수도 있음
- 필수 의존성이 보장되지 않음. 모든 의존성이 주입되지 않은 채로 객체가 사용될 수도 있음

3. **필드 주입**
```java
public class OrderService {
    @Autowired
    private DiscountPolicy discountPolicy;
}
```
- 객체의 필드에 직접 주입하는 방식
- 코드가 간결해지고, 주입이 편해짐
- 스프링 컨테이너가 객체를 생성하고 주입해야 하기 때문에, DI 컨테이너에 강하게 의존