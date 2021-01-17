---

layout: post
title: 210117 Effective Java 8장 정리
tags:
  - Effective Java
---

## 8. 메서드

### [item49] 매개변수가 유효한지 검사하라

- 매개변수 제약은 반드시 문서화해야 하며, 메서드 몸체가 시작되기 전 검사해야 한다
  - 예외: 유효성 검사 비용이 지나치게 높거나 실용적이지 않을때, 계산 과정에서 암묵적 검사가 수행될때
    - 예시: Collections.sort(List)
    - 상호 비교될 수 없는 타입의 객체가 들어있다면 그 객체와 비교할 때 ClassCastException
    - 하지만 암묵적 유효성 검사에 너무 의존했다가는 실패 원자성(failure atomicity, item76)을 어긴다
- '오류는 가능한 한 빨리 (발생한 곳에서) 잡아야 한다'
  - 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워진다
  - 감지하더라도 오류의 발생 지점을 찾기 어려워진다

#### 매개변수 검사를 제대로 하지 않았을 때

- 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다
  - 메서드가 잘 수행되지만 잘못된 결과를 반환한다
  - 메서드는 문제없이 수행되지만, 어떤 객체를 이상하게 만들어 놓아 미래의 알 수 없는 시점에 이 메서드와 관련없는 오류를 낸다
- 매개변수 검사에 실패하면 실패 원자성(failure atomicity, item76)을 어긴다

#### public과 protected 메서드는 매개변수 값이 잘못됐을때 던지는 예외를 문서화해야 한다

- @throws 자바독 태그(item74) 사용

- IllegalArgumentException, IndexOutOfBoundsException, NullPointerException

- 매개변수의 제약을 문서화하면 그 제약을 어겼을 때 발생하는 예외도 함께 기술

- 클래스 수준 주석: 그 클래스의 모든 public 메서드에 적용됨

- java.util.Objects.requireNonNull

  - 자바 7에서 추가

  - 유연하며 사용하기 편하다

  - null검사를 수동으로 하지 않아도 되며, 원하는 예외 메서드 지정 가능

  - 입력을 그대로 반환하여 값을 사용하는 동시에 null 검사를 수행할 수 있다

  - ```java
    this.strategy = Objects.requireNonNull(strategy, "전략");
    ```

- checkFromIndexSize, checkFromToIndex, checkIndex

  - 자바 9에서 Objects에 범위 검사 기능이 추가됨
  - null 검사 메서드만큼 유연하지 못하다
    - 예외메세지 지정 불가
    - 리스트와 배열 전용 설계
    - 닫힌 범위(양끝단 값을 포함)는 다루지 못한다
#### 공개되지 않은 메서드(public이 아닌 메서드)

- 패키지 제작자만이 메서드가 호출되는 상황을 통제
  - 오직 유효한 값만이 메서드에 넘겨지리라는 것을 보증할 수 있고, 그렇게 해야 한다
  - public이 아닌 메서드라면 단언문(assert)을 사용해 매개변수 유효성을 검증

#### 단언문 assert

```java
/* 재귀 정렬용 private 도우미 함수 */
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ...
}
```

- 단언문들은 자신이 단언한 조건이 무조건 참이라고 선언한다
  - 이 메서드가 포함된 패키지를 클라이언트가 어떤 식으로 사용하든 상관없다
- 단언문과 일반적인 유효성 검사가 다른 점
  - 실패하면 AssertionError를 던진다
  - 런타임에 아무런 효과도, 아무런 성능 저하도 없다
    - 단, java를 실행할때 명령줄에서 -ea 또는 --enableassertions플래그를 설정하면 런타임에 영향을 준다

#### 메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수

- 생성자: 이 원칙의 특수한 사례
  - 생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는데 필요

-----

### [item50] 적시에 방어적 복사본을 만들라

- 자바는 네이티브 메서드를 사용하지 않는다
  - C,C++등에서 흔히 보이는 버퍼 오버런, 배열 오버런, 와일드 포인터 같은 메모리 충돌 오류에서 안전하다
  - 자바로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 그 불변식이 지켜진다

#### 클라이언트가 불변식을 깨뜨리려 혈안이 되어있다 가정하고 방어적으로 프로그래밍하라

- 악의적인 보안 침해 시도
- 평범한 프로그래머의 실수로 인한 클래스 오작동

#### 예시) Period 클래스: 객체의 허락 없이 외부에서 내부를 수정했다

```java
/* 기간을 표현하는 클래스: 불변식을 지키지 못했다 */
public final class Period {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end){
        if(start.compareTo(end) > 0){
            throw new IllegalArgumentException(start+" after "+end);
        }
        this.start = start;
        this.end = end;
    }
    
    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }
    ...
}
```

- 이 클래스는 불변처럼 보이며, 시작 시간이 종료 시간보다 늦을 수 없다는 불변식이 무리 없이 지켜질 것 같다
- Date가 가변이라는 사실을 이용하면 어렵지 않게 불변식을 깰 수 있다

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부 수정
```

- 자바 8 이후로는 이 문제를 쉽게 해결 가능하다
  - Date 대신 불변인 Instant 사용
  - LocalDateTime, ZonedDateTime 사용
  - Date는 낡은 API이므로 새로운 코드를 작성할 때는 더이상 사용하면 안 된다
- 외부 공격으로부터 period 인스턴스의 내부를 보호하려면...
  - 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy) 해야 한다
  - period 인스턴스 안에서는 원본이 아닌 복사본을 이용한다

```java
/* 생성자 수정 */
    private Period(Date start, Date end){
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(start.compareTo(end) > 0){
            throw new IllegalArgumentException(start+" after "+end);
        }
    }
```

- 매개변수의 유효성을 검사하기 전, 방어적 복사본을 만들고 이 복사본으로 유효성을 검사했다

  - 멀티스레딩 환경에서 원본 객체 유효성을 검사하고 복사본을 만드는 찰나에 다른 스레드가 원본 객체를 수정할 위험이 있다

  - TOCTOU(time-of-check/time-of-use) 공격

- 방어적 복사시 Date의 clone을 사용하지 않았다

  - Date는 final이 아니므로 clone이 Date가 정의한 것이 아닐 수 있다
  - clone이 악의를 가진 하위 클래스의 인스턴스를 반환할 수 있다
  - 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들때 clone을 사용해서는 안 된다
  
```java
/* 두 번째 공격 */
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부 수정
```

- 접근자가 가변 필드의 방어적 복사본을 반환하도록 수정한다

```java
/* 접근자 수정 */
    public Date start() {
        return new Date(start.getTime());
    }
    public Date end() {
        return new Date(end.getTime());
    }
```

- 모든 필드가 객체 안에 캡슐화되었다

#### 객체가 잠재적으로 변경될 수 있는가?

- 메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 한다
  - 변경될 수 있는 객체라면 그 객체가 클래스에 넘겨진 뒤 임의로 변경되어도 그 클래스가 문제없이 동작할지를 따져보아야 한다
  - 확신할 수 없다면 복사본을 만들어 저장
- 클래스가 불변이든 가변이든, 가변인 내부 객체를 클라이언트에 반환할 때는 반드시 심사숙고해야 한다.
  - 안심할 수 없다면 원본을 노출하지 말고 복사본을 반환해야 한다
  - 예시: 길이가 1 이상인 배열은 무조건 가변이다 - 항상 방어적 복사 수행
- 복사 비용이 너무 크거나, 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을때의 책임이 클라이언트에 있음을 문서에 명시하자

-----

### [item51] 메서드 시그니처를 신중히 설계하라

#### API 설계 요령들

- 배우기 쉽고, 쓰기 쉬우며, 오류 가능성이 적은 API를 만들기 위한 설계 요령이다.

##### 메서드 이름을 신중히 짓자

- 항상 표준 명명 규칙(item68)을 따라야 한다
- 이해할 수 있고, 같은 패키지에 속한 다른 이름들과 일관되게 짓는다
- 개발자 커뮤니티에서 널리 받아들여지는 이름을 사용한다
- 긴 이름을 피한다
- 애매하다면 자바 라이브러리의 API가이드를 참조하라

##### 편의 메서드를 너무 많이 만들지 말자

- 모든 메서드는 각각 자신의 소임을 다해야 한다
- 메서드가 너무 많은 클래스는 어렵다
- 인터페이스도 마찬가지로 메서드가 너무 많으면 이를 구현하는 사람과 사용하는 사람 모두 고통스럽게 한다
- 확신이 서지 않는다면 만들지 말자

##### 매개변수 목록은 짧게 유지하자

- 4개 이하가 좋다
- 같은 타입 매개변수 여러개가 연달아 나오는 경우가 특히 해롭다
  - 사용자가 매개변수 순서를 기억하기 어렵고, 실수로 순서를 바꿔 입력해도 그대로 컴파일되고 실행된다

##### 매개변수 목록을 짧게 줄이는 기술 세 가지

- 여러 메서드로 쪼갠다
  - 쪼개진 메서드 각각은 원래 매개변수 목록의 부분집합을 받는다
  - 직교성이 높아지며 총 메서드 수가 줄어든다
    - 직교성(orthogonality): 공통점이 없는 기능들이 잘 분리되어 있다 / 기능을 원자적으로 쪼개 제공한다
  - 예시) java.util.List 인터페이스의 subList, indexOf 메서드
- 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다
  - 정적 멤버 클래스로 둔다
  - 매개변수 몇 개를 독립된 하나의 개념으로 볼 때(예시: 카드게임 클래스)
- 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용한다
  - 매개변수가 많으며, 일부는 생략해도 괜찮을 때 도움이 된다
  - 모든 매개변수를 추상화한 객체를 정의, 클라이언트에서 이 객체의 세터 메서드를 호출해 필요한 값을 설정하게 한다
    - 각 세터 메서드는 매개변수 하나 혹은 서로 연관된 몇 개만 설정하게 한다
  - 먼저 필요한 매개변수를 다 설정한 다음, execute 메서드를 호출해 앞서 설정한 매개변수들의 유효성을 검사하고 설정이 완료된 객체를 넘겨 원하는 계산을 수행한다

##### 매개변수의 타입으로는 클래스보다 인터페이스가 낫다

- 매개변수로 적합한 인터페이스가 있다면 이를 구현한 클래스가 아닌 그 인터페이스를 직접 사용한다
- 예시: 메서드 매개변수로 HashMap 대신 Map 을 사용한다
  - 그러면 HashMap 외 TreeMap, ConcurrentHashMap 등 어떤 Map 구현체도 인수로 건넬 수 있다
- 인터페이스 대신 클래스를 사용한다면?
  - 클라이언트에세 특정 구현체만 사용하도록 제한
  - 입력 데이터가 다른 형태로 존재한다면 명시한 특정 구현체의 객체로 옮겨 담느라 비싼 복사 비용을 들여야 한다

##### boolean보다는 원소 2개짜리 열거 타입이 낫다

- 열거 타입을 사용하면 코드를 읽고 쓰기가 더 쉬워진다
- 열거 타입 상수 각각에 메서드를 정의해둘 수도 있다

-----

### [item52] 다중정의는 신중히 사용하라

- 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택된다
  - 메서드를 재정의했다면 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준이 된다
  - 메서드 재정의: 상위 클래스가 정의한 것과 똑같은 시그니처의 메서드를 하위 클래스에서 다시 정의
  - 메서드를 재정의한 후 하위 클래스의 인스턴스에서 그 메서드를 호출하면 재정의한 메서드가 실행된다
    - 컴파일타임에 그 인스턴스의 타입이 무엇이었는지는 상관없다

#### 예시) 컬렉션을 집합, 리스트, 그 외로 구분하고자 만든 프로그램

```java
public class CollectionClassifier {
    public static String classify(Set<?> s){
        return "집합";
    }
    public static String classify(List<?> s){
        return "리스트";
    }
    public static String classify(Collection<?> s){
        return "그 외";
    }
    public static void main(String[] args){
        Collection<?>[] collections = {
            new HashSet<String>(), 
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values)()
        };
        
        for(Collection<?> c: collections)
            System.out.println(classify(c));
    }
}
```

- 예상 결과: 집합/ 리스트/ 그 외
- 실제 결과: 그 외/ 그 외/ 그 외
- 다중정의(overloading, 오버로딩) 된 세 classify중 어느 메서드를 호출할지는 컴파일타임에 정해진다
  - for문 안의 c는 항상 Collection<?> 타입이다
  - 런타임에는 타입이 매번 달라지나, 호출할 메서드를 선택하는 데는 영향을 주지 못한다
  - 따라서 컴파일타임의 매개변수 타입을 기준으로 항상 세 번째 메서드`classify(Collection<?> s)`가 호출된다

```java
public static String classify(Collection<?> c) {
    return c instanceof Set ? "집합" :
    		c instanceof List ? "리스트" : "그 외";
}
```

- 다중정의가 혼동을 일으키는 상황은 피해야 한다
  - 특히 공개 API라면 더욱 신경써야 한다
  - API사용자가 매개변수를 넘기면서 어떤 다중정의를 호출할지 모른다면 프로그램이 오동작하기 쉽다.
  - 런타임에 이상하게 행동하기 때문에 API사용자들이 문제를 진단하느라 긴 시간을 허비하게 된다

#### 예시) 재정의(override)된 메서드 호출 메커니즘

```java
class Wine{
    String name(){ return "포도주"; }
}

class SparklingWine extends Wine{
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}
public class Overriding{
    public static void main(String[] args){
        List<Wine> wineList = List.of(new Wine(), new SparklingWine(), new Champagne());
        for (Wine wine : wineList){
            System.out.println(wine.name());
        }
    }
}
```

- for문의 컴파일타임 타입이 모두 Wine인 것에 무관하게 항상 가장 하위에서 정의한 재정의 메서드가 실행된다
- 하지만 다중정의된 메서드 사이에선 객체의 런타임 타입은 전혀 중요하지 않다
  - 선택은 컴파일타임에, 오직 매개변수의 컴파일타임 타입에 의해 이루어진다

#### 매개변수 수가 같은 다중정의는 만들지 말자

- 가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다
- 다중정의하는 대신 메서드 이름을 다르게 지어주는 방법도 있다
  - 예시) [ObjectOutputStream](https://docs.oracle.com/javase/7/docs/api/java/io/ObjectOutputStream.html) 의 write 메서드는 모든 기본 타입과 일부 참조 타입용 변경을 가짐
    - 모든 메서드에 다른 이름을 지어주었다(writeBoolean(boolean),writeInt(int),writeLong(long)....)
    - read 메서드와 이름 짝을 맞추게 된다
- 생성자는 이름을 다르게 지을 수 없으므로, 두번째 생성자부터는 무조건 다중정의가 된다
  - 정적 팩터리 사용
  - 생성자는 재정의 불가능하니 다중정의와 재정의가 혼용될 걱정은 없다
- 매개변수 수가 같은 다중정의 메서드가 많더라도, 그중 어느 것이 주어진 매개변수 집합을 처리할지가 명확히 구분된다면 헷갈릴 일은 없다
  - 매개변수중 하나 이상이 근본적으로 다르다면 헷갈릴 일이 없다
    - '근본적으로 다르다' - 두 타입의 값을 서로 어느 쪽으로든 형변환할 수 없다
  - 이 조건을 충족한다면 어느 다중정의 메서드를 호출할지가 매개변수들의 런타임 타입만으로 결정된다
  - 따라서 컴파일타임 타입에는 영향을 받지 않게 되고, 혼란을 주는 주된 원인이 사라진다
  - 예시) ArrayList의 int를 받는 생성자와 Collection을 받는 생성자
    - 어떤 상황에서든 두 생성자중 어느 것이 호출될지 헷갈릴 일이 없다

#### (자바 5 이후) 오토박싱으로 인한 기본타입과 참조 타입 구분의 모호함 예시) SetList

```java
public class SetList {
    public static void main(String[] args){
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();
        
        for(int i= -3; i<3; i++){
            set.add(i);
            list.add(i);
        }
        
        for(int i=0; i<3; i++){
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set+ " " +list);
    }
}
```

- -3~2까지의 정수를 정렬된 집합과 리스트에 각각 추가한 다음, 양쪽에 똑같이 remove 메서드 세번 호출

  - 0,1,2를 제거한 후 [-3.-2,-1] [-3.-2,-1] 출력 예상
  - 집합에서는 음이 아닌 값을 제거, 리스트에서는 홀수를 제거한 후 [-3.-2,-1] [-2 0 2]를 출력

- set.remove(i)의 시그니처는 remove(Object)

  - 다중정의된 다른 메서드가 없으니, 기대한 대로 동작하여 집합에서 0 이상의 수들을 제거

- list.remove(i)는 remove(int index)를 수행

  - '지정한 위치'의 원소를 제거

- 따라서 list.remove의 인수를 Integer로 형변환하여 올바른 다중정의 메서드를 선택하게 해준다

  - ```java
    for(int i=0; i<3; i++){
                set.remove(i);
                list.remove((Integer) i);
        		// list.remove(Integer.valueOf(i));
            }
    ```

- `List<E>` 인터페이스가 remove(Object) 와 remove(int)를 다중정의했기 때문

  - 자바 4까지는 Object와 int가 근본적으로 달랐기 때문에 문제가 없었다
  - 제네릭과 오토박싱이 등장하면서 두 메서드의 매개변수 타입이 더는 근본적으로 다르지 않게 되었다
  - = 자바 언어에 제네릭과 오토박싱이 더해지면서 List 인터페이스가 취약해졌다

#### (자바 8 이후) 람다와 메서드 참조로 인한 다중정의 모호함 예시

```java
// 1. Thread 생성자 호출
new Thread(System.out::println).start();

// 2. ExecutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

- 2는 컴파일 오류 발생
  - 넘겨진 인수는 모두 같고, 양쪽 모두 Runnable을 받는 형제 메서드 다중정의
  - submit 다중정의 메서드 중에는 `Callable<T>`를 받는 메서드도 있다
  - 모든 println은 void를 반환하므로 반환값이 있는 Callable과 헷갈릴 일은 없다?
    - 다중정의 해소(resolution: 적절한 다중정의 메서드를 찾는 알고리즘)는 이렇게 동작하지 않음
  - println이 다중정의 없이 단 하나만 존재했다면 이 submit은 제대로 컴파일 됐을것
    - println, submit 양쪽 다 다중정의되면서 다중정의 해소 알고리즘이 우리의 기대처럼 동작하지 않는다
- System.out::println은 부정확한 메서드 참조이다(inexact method reference)
  - 암시적 타입 람다식이나 부정확한 메서드 참조 같은 인수 표현식은 목표 타입이 선택되기 전에는 그 의미가 정해지지 않아 적용성 테스트때 무시된다
  - 다중정의된 메서드(혹은 생성자)들이 함수형 인터페이스를 인수로 받을때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다
- 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안 된다
  - 서로 다른 함수형 인터페이스라도, 서로 근본적으로 다르지는 않다
  - 컴파일할때 명령줄 스위치로 -Xlint:overloads를 지정하면 이런 종류의 다중정의를 경고해준다

#### 근본적으로 다르다(radically different)

- 두 타입의 값을 (null이 아닌) 서로 어느 쪽으로든 형변환할 수 없다
- Object외의 클래스 타입과 배열 타입은 근본적으로 다르다
- Serializable과 Cloneable 외의 인터페이스 타입과 배열 타입도 근본적으로 다르다
- String, Throwable처럼 상위/ 하위 관계가 아닌 두 클래스는 '관련 없다(unrelated)'
- 어떤 객체도 관련없는 두 클래스의 공통 인스턴스가 될 수 없으므로, 관련 없는 클래스들끼리도 근본적으로 다르다

-----

### [item53] 가변인수는 신중히 사용하라

- 가변인수(varargs) 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다
- 가변인수 메서드 호출->인수의 개수와 길이가 같은 배열을 만듦->인수를 이 배열에 저장, 가변인수 메서드에 건네준다

#### 인수가 1개 이상이어야 하는 경우

- 인수 개수는 런타임에(자동 생성된) 배열의 길이로 알 수 있다
- 인수를 0개만 넣어 호출한다면 컴파일타임이 아닌 런타임에 실패한다
- 매개변수를 2개 받는다
  - 첫 번째: 평범한 매개변수
  - 두 번째: 가변인수

- 가변인수를 사용할 때는 성능 문제까지 고려한다
  - 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다
  - 인수를 적게 사용한다면, 메서드를 다중 정의한다

-----

### [item54] null이 아닌 빈 컬렉션이나 배열을 반환하라

- null이 아닌 빈 배열이나 컬렉션을 반환하라
  - null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어나며, 성능이 뛰어나게 좋은 것도 아니다

#### 예시) 컬렉션이 비었을 때 null을 반환

```java
/* 컬렉션이 비었을 때 null을 반환한다 */
private final List<Cheese> cheesesInStock=...;

public List<Cheese> getCheeses(){
    return cheesesInStock.isEmpty()? null: new ArrayList<>(cheesesInStock);
}

...
List<Cheese> cheeses = shop.getCheeses();
// null 상황을 처리하는 코드를 추가로 작성해야 한다
if(cheeses != null && ....)
    ...
```

- 컬렉션, 배열 같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용할때면 위와 같은 방어 코드를 작성해야 한다
  - 실제 객체가 0개인 가능성이 거의 없는 상황에서는 수년 뒤에야 오류를 발견하게 된다
- 빈 컨테이너를 할당하는 데에도 비용이 드니 null을 반환하는 것이 낫다?
  - 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는다면 이정도의 성능 차이는 신경 쓸 수준이 되지 않는다
  - 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다

```java
/* 빈 컬렉션을 반환 */
public List<Cheese> getCheeses(){
    return new ArrayList<>(cheesesInStock);
}
/* 최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 */
public List<Cheese> getCheeses(){
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
/* 길이가 0일수도 있는 배열 반환 */
public Cheese[] getCheeses(){
    return cheesesInStock.toArray(new Cheese[0]);
}
/* 최적화 - 빈 배열을 매번 새로 할당하지 않도록 */
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
/* 나쁜 예시 - 배열을 미리 할당하면 성능이 나빠진다 */
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[cheesesInStoke.size()]);
}
```

-----

### [item55] 옵셔널 반환은 신중히 하라

- 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 하는 메서드라면 옵셔널을 반환해야 하는 상황
- 옵셔널 반환에는 성능 저하가 뒤따른다
  - 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 낫다
  - 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다

#### 메서드가 특정 조건에서 값을 반환할 수 없을 때

- 예외를 던진다
  - 진짜 예외적인 상황에서만 사용
  - 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용도 만만치 않다
- (반환 타입이 객체 참조라면) null을 반환한다
  - 별도의 null처리 코드를 추가해야 함
- 자바 8 이후 `Optional<T>` 
  - null이 아닌 T타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다

#### 예시) 주어진 컬렉션에서 최댓값 구하기

```java
/* 컬렉션에서 최댓값 구하기 (컬렉션이 비었다면 예외 던지기) */
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}

/* 컬렉션에서 최댓값을 구해 Optional<E>로 반환 */
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);
}

/* 컬렉션에서 최댓값을 구해 Optional<E>로 반환 - 스트림 버전 */
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

- 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 않는다

#### 옵셔널 활용

- null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야 하는 기준
  - 검사 예외와 취지가 비슷하다
  - 반환값이 없을수도 있음을 API 사용자에게 명확하게 전달한다
- 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환한다
- 메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 한다
  - 기본값 설정
    - Supplier<T>를 인수로 받는 orElseGet을 사용하여 값이 처음 필요할 때 생성, 초기 설정 비용을 낮출 수 있다
  - 원하는 예외를 던짐
  - 항상 값이 채워져 있다고 가정한다
    - NoSuchElementException이 발생할 위험이 있다
- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다
  - 빈 Optional<List<T>> 반환보다 빈 List<T> 반환이 좋다
- 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자
  - 덜 중요한 기본타입 용인 Boolean, Byte, Character, Short, Float등은 예외일 수 있다
- 옵셔널은 컬렉션의 키,값,원소나 배열의 원소로 사용하는 것에 적절하지 않다

----

### [item56] 공개된 API 요소에는 항상 문서화 주석을 작성하라

- [문서화 주석 작성법](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)
  - 자바4 이후로는 갱신되지 않았으나 아주 유용하다....

#### API를 올바로 문서화 하려면

##### 공개된 모든 클래스,인터페이스,메서드,필드 선언에 문서화 주석을 달아야 한다

- 직렬화 가능하다면 직렬화 형태에 관해서도 적어야 한다
- 유지보수까지 고려한다면 대다수의 공개되지 않은 요소들에도 문서화 주석을 다는 것이 좋다

##### 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다

- 상속용으로 설계된 것이 아니라면 그 메서드가 무엇(what)을 하는지를 기술해야 한다
- 문서화 주석에는 클라이언트가 해당 메서드를 호출하기 위한 전제조건을 모두 나열해야 한다
- 전제조건은 @throws 태그로 비검사 예외를 선언, 암시적으로 기술한다
- @param 태그를 이용해 그 조건에 영향받는 매개변수에 대해 기술할 수 있다
- 문서화 주석에 HTML태그를 이용, 자바독 유틸리티는 문서화 주석을 HTML로 변환하므로 문서화 주석 안의 HTML요소들이 최종 HTML 문서에 반영된다
- @문자에는 무조건 이스케이프 문자를 붙여야 하므로 문서화 주석 안의 코드에서 애너테이션을 사용한다면 주의하기 바란다

##### 한 클래스(혹은 인터페이스) 안에서 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안 된다

##### 요약 설명에서는 마침표(.) 에 유의하라

- 요약 설명이 끝나는 판단 기준: 마침표
  - 의도치 않은 마침표를 포함한 텍스트를 {@literal}로 감싸주는 것이다
  - 자바10 부터는 {@summary}라는 요약 설명 전용 태그가 추가되어 더 깔끔하게 처리 가능하다

##### 메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는, 주어가 없는 동사구여야 한다

##### 자바 9부터는 자바독이 생성한 HTML 문서에 검색(인덱스) 기능이 추가되었다

- {@index} 태그로 API에서 중요한 용어를 추가로 색인화한다

##### 제네릭 타입이나 제네릭 메서드를 문서화할때는 모든 타입 매개변수에 주석을 달아야 한다

##### 열거 타입을 문서화할 때는 사웃들에도 주석을 달아야 한다

- 열거 타입 자체와 그 열거 타입의 public 메서드도 물론이다

##### 애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다

- 애너테이션 타입 자체도 물론이다
- 필드 설명은 명사구로 한다
- 요약 설명은 프로그램 요소에 이 애너테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구로 한다

##### 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다

- 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다
- 가장 자주 누락되는 설명이다(스레드 안전성과 직렬화 가능성)

##### 자바독은 메서드 주석을 상속시킬 수 있다

- {@inheritDoc} 태그를 사용, 상위 타입의 문서화 주석 일부를 상속