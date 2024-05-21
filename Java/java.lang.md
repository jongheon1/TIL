
### Long class

Long class 는 원시 타입 long 의 값을 object 로 감싼다. Long type object 는 long type 인 하나의 필드를 포함한다.
long 을 String 으로, String 을 long 으로 바꾸는 메서드 뿐만 아니라, long 을 다루는 여러 유용한 메서드를 포함한다.

MIN_VALUE = 0x8000000000000000L
MAX_VALUE = 0x7fffffffffffffffL


#### `public static String toString(long i, int radix)`

두 번째 인자가 지정한 기수로 첫 번째 인자의 문자열 표현을 반환한다.
radix 가 Character.Min_RADIX(2) 보다 작거나, Character.MAX_RADIX(36) 보다 크면, radix 는 10으로 대체 된다.

#### `public static long parseLong(String s, int radix)`

두 번째 인자가 지정한 기수로 string argument 를 signed long 으로 파싱한다. 즉, radix 가 10 이면, s 를 10진수로 인식해 long 으로 표현하겠다는 뜻
string 의 characters 는 첫번째 character 가 - 인 경우를 제외 하고, 모두 radix 가 지정한 digits 이어야한다. (1~z)
type indicator 로서 string 의 끝에 L 혹은 l 이 나타나는 것은 허락되지 않는다.
NumberFormatExcaption 은 다음과 같은 상황에 발생한다.
- s 가 null 이거나 혹은 string length 가 0일 때
- radix 가 Character.Min_RADIX(2) 보다 작거나, Character.MAX_RADIX(36) 보다 클 때
- string 의 characters 가 radix 가 지정한 digits 이 아닐 때
- string 으로 나타나는 값이 long type value 가 아닐 때

Examples:
  - parseLong("0", 10) returns 0L
  - parseLong("473", 10) returns 473L
  - parseLong("+42", 10) returns 42L
  - parseLong("-0", 10) returns 0L
  - parseLong("-FF", 16) returns -255L
  - parseLong("1100110", 2) returns 102L
  - parseLong("99", 8) throws a NumberFormatException
  - parseLong("Hazelnut", 10) throws a NumberFormatException
  - parseLong("Hazelnut", 36) returns 1356099454469L
```java
public static long parseLong(String s) throws NumberFormatException {  
    return parseLong(s, 10);  
}
```
`parseLong(String s)` 은 위와 같이 `parseLong(s, 10)` 을 호출함. 즉 s 를 10진수 long 으로 인식 해 표현하겠단 뜻.


#### `public static Long valueOf(long l)`
특정한 long value 를 나타내는 Long instance 를 반환함
새로운 Long instance 가 필요하지 않다면, 이 메소드가 생성자 Long(long) 보다 일반적으로 쓰여야함. 자주 요청되는 값을 캐싱함으로써, 공간 및 시간적 성능이 훨씬 향상될 가능성이 높기 때문.

```java
public static Long valueOf(long l) {  
    final int offset = 128;  
    if (l >= -128 && l <= 127) { // will cache  
        return LongCache.cache[(int)l + offset];  
    }  
    return new Long(l);  
}```
valueOf 함수의 상세 구현
-128 ~ 127 사이의 값이라면 LongCache 라는 캐시에서 값을 가져오고, 범위 밖이라면 새로 만듦.
엥 근데 고작 이 정도만 캐시한다고? 이것만 할거면 걍 다 새로 만들면 안 되나? 싶긴 한데
-128 ~ 127 사이의 값이 젤 많이 사용된다고 한다.  예를 들어 반복문에서 인덱스로 쓴다거나,,, 기본적인 산술 연산에 쓴다거나,,, 하는 것 말이다. 또한 Integer 클래스에서도 같은 범위는 캐시하기 때문에 확실히 빈도로 따지면,,, 젤 많고,, 이것만 캐시해도 오버헤드를 많이 줄일 수 있다고 한다.


```java
private static class LongCache {  
    private LongCache() {}  
  
    static final Long[] cache;  
    static Long[] archivedCache;  
  
    static {  
        int size = -(-128) + 127 + 1;  
  
        // Load and use the archived cache if it exists  
        CDS.initializeFromArchive(LongCache.class);  
        if (archivedCache == null || archivedCache.length != size) {  
            Long[] c = new Long[size];  
            long value = -128;  
            for(int i = 0; i < size; i++) {  
                c[i] = new Long(value++);  
            }  
            archivedCache = c;  
        }  
        cache = archivedCache;  
    }  
}
```
LongCache 의 내부..
cache 와 archivedCache 가 있고,,, 정적 초기화 블록이 있다.
LongCache 클래스는 private 에다가,, 생성자도 private 으로 막혀있다... 즉 Long 내부에서만 쓸 수 있고,, 생성자가 필요 없다는 뜻
cache 는 Long 객체들을 저장할 최종 배열이고, archivedCache 기존에 있던 캐시 배열을 불러오기 위한 것,,
정적 초기화 내부에선 캐시를 초기화 한다.
size 는 배열 크기인데,,, 굳이`-(-128) + 127 + 1;` 이렇게 해 둔 건,,, 범위가 -128 ~ 127 까지라는 걸 보여주기 위한 것인 듯..
`CDS.initializeFromArchive(LongCache.class)` 부분은 아카이브된 캐시를 로드하는 역할을 한다. CDS(Class Data Sharing) 은 JVM 에서 클래스 데이터를 공유하고 메모리 사용을 최적화 하는 기술이란다.. 내부는 좀 어려운 듯.. 걍 캐시의 캐시,, 정도로 이해하면 될 듯
그래서 아카이브된 캐시를 로드하려 하는데, 딱 맞는게 있다면 archivedCache 로 로드
근데 이제,,, null 이거나,, 크기가 맞지 않으면,, 새로 Long 배열 생성한다.
그래서 -128 ~127 까지 Long 생성해서 archivedCache 에 할당하고,,,
모두 끝나면 cache 에 할당 !

정적 초기화 블록이 완료되면, LongCache.cache 배열이 초기화되어서 사용 준비가 완료된다. 이제 valueOf 메서드는 캐시된 값을 반환할 수 있다....

참고로 LongCache 클래스와 정적 초기화 블록은 valueOf 에서와 같이 LongCache 클래스를 처음 참조할 때 초기화 되는데,, 이를 지연 초기화(lazy initialization) 이라고 한다... 메모리에서 지연 할당과 비슷한 듯


#### `public static Long valueOf(String s, int radix)`
string 에서 값을 추출한 값을 가지는 Long object 를 반환함
```java
public static Long valueOf(String s, int radix) throws NumberFormatException {  
    return Long.valueOf(parseLong(s, radix));  
}
```
string 을 parseLong 으로 long 으로 바꾼 뒤,,, valueOf(long l) 을 호출해서,,, 그대로 Long 으로 감쌈...
같은 String 에 대해서 parseLong 은 걍 long 으로 바꾸는 것이고,,, valueOf 는 parseLong 한 값을,,, Long 으로 감싸기만 하는 것....