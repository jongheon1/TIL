# 한정적 와일드카드 타입


### 제네릭 타입과 한정적 와일드카드 타입

```java
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```
위 메서드는 컴파일은 잘 되지만 완벽하진 않다. Iterable src 의 원소 타입이 스택의 원소 타입과 일치해야만 잘 작동한다.

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);
numberStack.pushAll(integers);
```

위 코드는 컴파일에 실패한다. 왜냐하면 매개변수화 타입이 불공변이기 때문이다. 다시말해 Iterable<Integer> 는 Iterable<Number> 의 하위 타입이 아니라는 것이다.

이 문제를 해결하기 위해 한정적 와일드카드 타입을 사용할 수 있다.

pushAll 의 입력 매개변수 타입은 'E 의 Iterable' 이 아니라, 'E 의 하위 타입의 Iterable' 이어야 한다. Iterable<? extends E> 가 정확히 이런 뜻이다.

```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

위와 같이 하면 Stack 은 물론이고 이를 사용하는 클라이언트 코드도 잘 컴파일된다.

pushAll 과 짝을 이루는 popAll 도 마찬가지다.

```java
public void popAll(Collection<> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
```
이번에도 역시나 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치해야만 잘 작동한다.

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = new ArrayList<>();
numberStack.popAll(objects);
```

위 코드는 컴파일에 실패한다. 왜냐하면 Collection<Object> 는 Collection<Number> 의 하위 타입이 아니기 때문이다.

이번에도 한정적 와일드카드 타입을 사용해 문제를 해결할 수 있다.

popAll 의 입력 매개변수 타입은 'E 의 Collection' 이 아니라, 'E 의 상위 타입의 Collection' 이어야 한다. Collection<? super E> 가 정확히 이런 뜻이다.

```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
```

이제 Stack 과 클라이언트 코드 모두 잘 컴파일된다.



### 어떨 때 어떤 와일드카드 타입을 써야할까

| PECS: producer-extends, consumer-super

즉 매개변수화 타입 T 가 생산자라면 <? extends T> 를 사용하고, 소비자라면 <? super T> 를 사용하면 된다. 

Stack 에서 pushAll 의 src 의 매개변수는 Stack 이 사용할 E 인스턴스를 생산한다. src 에서 생산하는 인스턴스를 Stack 이 E 타입으로 받아들여 사용한다는 것이다. 그러므로 src 는 E 의 하위 타입을 생산해야 하고, src 의 적절한 타입은 Iterable<? extends E> 이다.

한편 popAll 의 dst 매개변수는 Stack 으로부터 E 인스턴스를 소비한다. dst 에서 E 인스턴스를 받아들여 사용할 수 있어야 한다는 것이다. 그러므로 dst 는 E 의 상위 타입을 소비해야 하고, dst 의 적절한 타입은 Collection<? super E> 이다.

PECS 공식은 와일드카드 타입을 사용하는 기본 원칙이다.


### 추가 예시

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

결과로 Set<E> 를 반환하는데, s1 과 s2 는 모두 이 Set 에서 사용할 E 인스턴스를 생산한다. s1, s2 가 E 의 하위 타입을 생산해야 하는 것이다. 그러므로 다음처럼 선언해야 한다.

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

수정한 선언을 사용하면 다음 코드도 잘 컴파일된다.

```java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers, doubles);
```

다음과 같은 메서드 선언도 있다.

```java
public static <E extends Comparable<E>> E max(List<E> list)
```

와일드카드 타입을 사용해 다듬는다면 이렇게 된다.

```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

먼저 입력 매개변수인 list 는 E 인스턴스를 생산한다. 그러므로 List<? extends E> 가 되어야 한다.

타입 매개변수 E 는 Comparable<E> 를 확장하는데, 이때 Comparable<E> 는 E 인스턴스를 소비한다. 그러므로 Comparable<? super E> 가 되어야 한다.


정리하면 다음과 같다.

먼저 list 는 비교 대상이 되는 E 인스턴스를 생산한다. E 의 하위 타입의 List 가 되어야 E 를 기준으로 비교할 수 있다.

E 는 비교 대상이기 때문에 Comparable 를 구현해야 한다. 그런데 이때 Comparable 은 결국 비교하기 위해 E 를 소비하는 곳이다. Comparable 에선 E 를 받야들일 수 있어야 하기 때문에 E 의 상위 타입의 Comparable 이 되어야 하는 것이다. 그러므로 Comparable<? super E> 가 되어야 한다.
