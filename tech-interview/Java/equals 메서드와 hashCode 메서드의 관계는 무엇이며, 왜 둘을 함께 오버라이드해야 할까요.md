# equals 메서드와 hashCode 메서드의 관계는 무엇이며, 왜 둘을 함께 오버라이드해야 할까요?

equals 는 두 객체가 논리적으로 같음을 판단할 때 사용합니다. Object 클래스에서 기본 구현을 제공하는데, 이는 두 객체의 참조 값이 같은지만 확인합니다. 대부분의 경우 이 기본 동작은 충분하지 않기 때문에 equals 를 오버라이드 해서 사용합니다.
hashCode 는 객체의 hashcode 를 반환합니다. hashcode 는 자바 어플리케이션이 실행하는 동안 같은 객체에 대해 hashCode 를 호출 할 때 마다 일관되게 같은 정수 값을 가집니다. 또한 equals 에 따라 두 객체가 같으면, 두 객체에 대해 각각 hashCode 를 호출 했을 때 같은 정수 결과가 생겨야합니다.

만약 equals 만 오버라이드 하고 hashCode 는 오버라이드 하지 않는다면 논리적으로 같은 두 객체가 다른 hashcode 값을 가질 수 있습니다. 이 경우 정의상으로 문제될 뿐만 아니라, 해시를 기반으로 하는 컬렉션을 사용할 때 성능상의 문제가 생길 수도 있습니다.

1. "이해를 돕기 위해, 자바에서 equals와 hashCode를 어떻게 오버라이드 하는지 간단한 예를 들어 설명해 줄 수 있나요?"

```
class Person {
    public String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person p = (Person) o;
        return Objects.equals(name, p.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```


2. "해시 충돌이 무엇이며, hashCode에서 충돌을 줄이기 위한 방법은 무엇이 있을까요?"

해시 충돌이란 서로 다른 두 객체가 동일한 해시코드를 반환하는 경우입니다. 이는 해시 기반 자료구조에서 성능 저하를 초래할 수 있습니다. 이를 막기 위해 

3. "equals와 hashCode를 잘못 구현했을 때 발생할 수 있는 성능 문제에 대해 자세히
설명해 줄 수 있나요?"
4. "equals와 hashCode 메소드의 중요성은 HashMap이나 HashSet 외에 다른 컬렉션에서는 어떻게 나타날까요?"
5. "자바 7 이상에서 Objects 클래스에 추가된 hash와 equals 메소드를 사용하는 것의 장단점은 무엇일까요?"

