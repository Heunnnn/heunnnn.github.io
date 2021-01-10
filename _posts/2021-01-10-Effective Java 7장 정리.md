---

layout: post
title: 210110 Effective Java 7장 정리
tags:
  - Effective Java
---

## 6. 람다와 스트림

### [item42] 익명 클래스보다는 람다를 사용하라

#### 익명 클래스

```java
/* 익명 클래스의 인스턴스를 함수 객체로 사용 */
Collections.sort(words, new Comparator<String>(){
    public int compare(String s1,String s2){
        return Integer.compare(s1.length(), s2.length());
    }
});
```

- 전략 패턴과 유사
  - Comparator 인터페이스: 정렬을 담당하는 추상 전략
  - 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현
  - 코드가 길어지게 되므로 사용X

#### 람다 

```java
/* 람다식을 함수 객체로 사용, 익명 클래스를 대체한다 */
Collections.sort(words, (s1,s2) -> Integer.compare(s1.length(), s2.length()));
// 람다 자리에 비교자 생성 메서드를 사용하면 코드가 더 간결해진다
Collections.sort(words, comparingInt(String::length));
// List 인터페이스에 추가된 Sort를 사용하면 더 짧아진다
words.sort(comparingInt(String::length));
```

- 작은 함수 객체들을 아주 쉽게 표현한다
- 람다(`Comparator<String>`), 매개변수 s1/s2(`String`), 반환값(`int`)의 타입에 대해 코드상 언급이 없다
  - 컴파일러가 문맥을 살펴 타입을 추론한다
  - 컴파일러가 타입을 추론하는 데 필요한 타입 정보 대부분은 제네릭에서 얻는다
    - 따라서 item26 '제네릭의 로 타입을 사용하지 말라', item29 '제네릭을 사용하라', item30 '제네릭 메서드를 사용하라' 를 지키는 것이 좋다
- 타입을 명시해야 코드가 더 명확할 때를 제외하고는, 람다의 모든 매개변수 타입을 생략한다

#### 열거 타입에서 람다 사용

```java
/* 람다를 인스턴스 필드에 저장, 상수별 동작을 구현한다 */
public enum Operation {
    PLUS ("+", (x, y) -> x + y;),
    MINUS("-", (x, y) -> x - y;),
    TIMES("*", (x, y) -> x * y;),,
    DIVIDE("/", (x, y) -> x / y;),;
    
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op){
        this.symbol = symbol;
        this.op = op;
    }

}
```

- item 34의 '상수별 클래스 몸체 구현보다 열거타입에 인스턴스 필드를 두는 편이 낫다'
- 람다를 이용한다
  - 각 열거타입 상수의 동작을 람다로 구현해 생성자에 넘긴다
  - 생성자는 이 람다를 인스턴스 필드로 저장해둔다
  - apply 메서드에서 필드에 저장된 람다를 호출한다
- 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다
  - 열거 타입 생성자에 넘겨지는 인수들의 타입은 컴파일 타임에 추론
  - 인스턴스는 런타임에 만들어진다
- 상수별 클래스 몸체를 사용해야 하는 경우
  - 코드 자체로 동작이 명확하지 않거나 코드 줄 수가 길어지는 경우
    - 람다는 한 줄일때 가장 좋고 길어야 세 줄 안에 끝내는게 좋다
  - 인스턴스 필드나 메서드를 사용해야만 하는 경우

#### 익명 클래스를 사용해야 하는 경우

- 람다는 함수형 인터페이스에서만 쓰인다
- 추상 클래스의 인스턴스를 만들 때
- 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때
- 함수 객체가 this 키워드를 이용, 자기 자신을 참조해야 하는 경우
  - 람다는 자신을 참조할 수 없다
  - 람다의 this 키워드: 바깥의 인스턴스를 가리킨다
  - 익명 클래스의 this 키워드: 익명 클래스 인스턴스 자신을 가리킨다
- 익명 클래스와 람다 모두, 직렬화하는 일은 극히 삼가야 한다
  - 직렬화 형태가 구현(가상머신)별로 다를 수 있다
  - 직렬화해야 하는 함수 객체가 있다면(예시: Comparator)
    - item 24 'private 정적 중첩 클래스' 의 인스턴스를 사용한다

-----

### [item43] 람다보다는 메서드 참조를 사용하라

- 대부분의 상황에서는 메서드 참조가 더 짧고 명확하다
  - 람다로 작성할 코드를 새로운 메서드에 담고, 람다 대신 그 메서드 참조를 사용하라
  - 메서드 참조에는 기능을 잘 드러내는 이름을 지을 수 있고, 설명을 문서로 남길 수 있다
- 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다
  - 예외: [제네릭 함수 타입의 구현](https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.9) 예제 9.9-2
- IDE들도 람다를 메서드 참조로 대체하라고 권고한다

#### 예시) Map의 merge 메서드

```java
map.merge(key, 1, (count, incr) -> count + incr)
```

- 키가 맵 안에 없다면 키와 1 매핑, 이미 있다면 기존 매핑값 증가
- 매개 변수인 count와 incr은 큰 쓰임새 없이 공간을 꽤 차지한다
  - 람다에서는 두 인수의 합을 단순히 반환하고 있다
- 자바 8부터 모든 기본 타입의 박싱 타입 클래스에서, 이 람다와 기능이 같은 정적 메서드 sum을 제공한다

```java
map.merge(key, 1, Integer:: sum);
```

- 매개변수 수가 늘어날수록 메서드 참조로 제거 가능한 코드도 늘어난다
  - 어떤 람다에서는 매개변수의 이름 자체가 좋은 가이드가 되어, 길이는 더 길지만 읽기 쉽고 유지보수도 쉬울 수 있다

#### 람다가 메서드 참조보다 간결한 경우

```java
/* 아래 코드는 GoshThisClassNameIsHumongous 클래스 내부의 코드이다 */
service.execute(GoshThisClassNameIsHumongous::action);
// 람다로 대체
service.execute(() -> action());
```

- 주로 메서드와 람다가 같은 클래스 내부에 있는 경우
- 또 다른 예시) java.util.function 패키지의 제네릭 정적 팩터리 메서드인 Function.identity() 보다 똑같은 기능의 람다 (x -> x)를 직접 사용

#### 메서드 참조의 유형

| 메서드 참조 유형   | 예                     | 같은 기능을 하는 람다                                   |
| ------------------ | ---------------------- | ------------------------------------------------------- |
| 정적               | Integer::parseInt      | str -> Integer.parseInt(str)                            |
| 한정적(인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now();<br />t -> then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase    | str -> str.toLowerCase()                                |
| 클래스 생성자      | TreeMap<K, V>::new     | () -> new TreeMap<K, V>                                 |
| 배열 생성자        | int[]::new             | len -> new int[len]                                     |

- 정적 메서드를 가리키는 메서드 참조
- 인스턴스 메서드를 참조
  - 수신 객체(receiving object; 참조 대상 인스턴스)를 특정하는 한정적(bound) 인스턴스 메서드 참조
    - 근본적으로 정적 참조와 유사
    - 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다
  - 수신 객체를 특정하지 않는 비한정적(unbound) 인스턴스 메서드 참조
    - 함수 객체를 적용하는 시점에 수신 객체를 알려준다
    - 수신 객체 전달용 매개변수가 매개변수 목록의 첫번째로 추가되며, 그 뒤에 참조되는 메서드 선언에 정의된 매개변수들이 뒤따른다
    - 사용 예시) 스트림 파이프라인에서의 map(), filter()
- 클래스 생성자를 가리키는 메서드 참조
  - 팩터리 객체로 사용된다
- 배열 생성자를 가리키는 메서드 참조

-----

### [item44] 표준 함수형 인터페이스를 사용하라

- 자바가 람다를 지원하게 되면서 API 작성시 입력값과 반환값에 함수형 인터페이스 타입을 활용하게 됨
  - 예시) 템플릿 메서드 패턴: 상위 클래스의 기본 메서드를 재정의, 원하는 동작 구현
    - 대체법: 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공한다
    - 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 하는데, 이때 함수형 매개변수 타입을 올바르게 선택해야 한다
- 보통은 java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋다

#### 예시) LinkedHashMap의 removeEldestEntry 재정의

```java
/* removeEldestEntry를 재정의해 캐시처럼 사용한다.
*  맵에 원소가 100개가 될 때까지 커지다가, 그 이상이 되면 새로운 키가 더해질때마다
*  가장 오래된 원소를 하나씩 제거한다.
*/
protected boolean removeEldestEntry(Map.Entry<K, V> eldest){
    return size > 100;
}
```

- 이 함수 객체는 `Map.Entry<K, V>`를 받아 boolean을 반환한다?
  - size()를 호출, 맵의 원소 수를 알아낸다
    -  removeEldestEntry가 인스턴스 메서드이기 때문에 가능한 방식
    - 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메서드가 아니다
      - 팩터리나 생성자 호출시에는 맵의 인스턴스가 존재하지 않기 때문
      - 따라서 맵은 자기 자신도 함수 객체에 전달해야 한다

```java
/* 함수형 인터페이스로 구현: 불필요함! */
@FunctionalInterface interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K,V> map, Map.Entry<K, V> eldest);
}
```

- 자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 있다
  - `BiPredicate<Map<K, V>, Map.Entry<K, V>>`
- 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라
  - API가 다루는 개념의 수가 줄어들어 익히기 쉽다
  - 유용한 디폴트 메서드가 많아 다른 코드와의 상호운용성도 좋아진다

##### @FunctionalInterface 애너테이션

- 직접 만든 함수형 인터페이스에는 항상 이 애너테이션을 사용하라
- 이 애너테이션의 사용 목적은...
  - 코드나 설명 문서를 읽을 이에게 이 인터페이스가 람다용으로 설계된 것임을 알려준다
  - 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다
  - 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하도록 해준다

#### java.util.function 패키지의 표준 함수형 인터페이스

- 총 43개의 인터페이스가 담겨 있다
- 대부분 기본 타입만 지원한다
- 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하면 안 된다!
  - item61 '박싱된 기본 타입 대신 기본 타입을 사용하라'를 위배한다
  - 계산량이 많을때는 성능이 처참해진다

##### 기본 인터페이스 6가지

| 인터페이스        | 함수 시그니처       | 예                  |
| ----------------- | ------------------- | ------------------- |
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T>       | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept)T t     | System.out.println  |

- 기본 인터페이스는 기본 타입인 int, long, double용으로 각 3개씩 변형이 생겨난다
  - 이름도 기본 인터페이스의 이름 앖에 해당 기본 타입 이름을 붙인다
    - 예시) IntPredicate, LongBinaryOperator.....
  - 이 변형들 중 유일하게 Function의 변형만 매개변수화되었다
    - 반환 타입만 매개변수화되었다!
    - 예시) LongFunction<int[]> : long 인수를 받아 int[]을 반환한다
- Function 인터페이스에는 기본 타입을 반환하는 변형이 총 9개가 더 있다
  - Function 인터페이스의 변형은 입력과 결과의 타입이 항상 다르다
    - 입력과 결과 타입이 같은 경우는 UnaryOperator를 사용한다
  - 입력과 결과 타입이 모두 기본 타입이면 접두어로 SrcToResult를 사용한다(3*2=6가지)
    - 예시) LongToIntFunction : long을 받아 int를 반환
  - 입력이 객체 참조이고 결과가 int, long, double인 변형들(3가지)
    - 입력을 매개변수화하고 접두어로 ToResult를 사용한다
    - 예시) ToLongFunction<int[]> : int[] 인수를 받아 long을 반환한다
- 인수를 2개씩 받는 변형
  - predicate
    - BiPredicate<T,U>
  - Function
    - BiFunction<T,U,R>
    - ToIntBiFunction<T,U>
    - ToLongBiFunction<T,U>
    - ToDoubleBiFunction<T,U>
  - Consumer
    - BiConsumer<T,U>
    - ObjDoubleConsumer<T>
    - ObjIntConsumer<T>
    - ObjLongConsumer<T>
- BooleanSupplier 인터페이스
  - boolean을 반환하도록 한 Supplier의 변형

#### 직접 함수형 인터페이스를 작성해야 하는 경우

##### 예시) `Comparator<T>` 인터페이스의 경우

- 구조적으로는 `ToIntBiFunction<T,U>` 와 동일하다
- 자바 라이브러리에 `Comparator<T>`를 추가할 당시,  `ToIntBiFunction<T,U>` 가 이미 존재했더라도  `ToIntBiFunction<T,U>` 를 사용하면 안 됐다.
- Comparator가 독자적인 인터페이스로 살아남아야 하는 이유
  - API에서 굉장히 자주 사용되는데, 지금의 이름이 그 용도를 잘 설명해준다
  - 구현하는 쪽에서 반드시 지켜야 하는 규약을 담고 있다
  - 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 담고 있다
- 따라서 위의 특성들 중 하나 이상을 만족한다면, 직접 함수형 인터페이스를 구현하는 것을 진중하게 고민해볼 수 있다
  - 인터페이스이므로, 아주 주의해서 설계해야 한다(item21)

#### 함수형 인터페이스를 API에서 사용할때의 주의점

- 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안 된다
  - item52 '다중정의는 주의해서 사용하라'의 특수한 예시
  - 클라이언트에게 불필요한 모호함만 안겨준다
  - 이 모호함으로 인해 실제로 문제가 일어난다
  - 예시) ExecutorService의 submit 메서드
    - `Callable<T>`를 받는 것과 Runnable을 받는 것을 다중정의했다
    - 올바른 메서드를 알려주기 위해 형변환이 필요할 때가 있다(item52)

-----

### [item45] 스트림은 주의해서 사용하라

- 스트림을 과용하면 프로그램의 가독성과 유지보수성이 나빠진다
- 기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아보일때만 반영한다

#### 스트림 API

##### 스트림

- 데이터 원소의 유한 혹은 무한 시퀀스
- 스트림의 원소들은 어디로부터든 올 수 있다
  - 컬렉션,배열,파일,정규표현식 패턴매처,난수 생성기,다른 스트림 등
- 스트림 안의 데이터 원소들은 객체 참조 또는 기본 타입(int,long,double) 값이다

##### 스트림 파이프라인

- 스트림 원소들로 수행하는 연산 단계
- 소스 스트림에서 시작해 종단 연산(terminal operation)으로 끝난다
  - 그 사이에 하나 이상의 중간 연산(intermediate operation), 각 중간 연산은 스트림을 어떠한 방식으로 변환(transform)
  - 중간 연산은 한 스트림을 다른 스트림으로 변환
  - 종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다
- 스트림 파이프라인은 지연 평가(lazy evaluation)된다
  - 평가는 종단 연산이 호출될 때 이루어진다
  - 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다
  - 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 no-op와 같다
- 스트림 API는 메서드 연쇄를 지원하는 플루언트 API
  - 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다
  - 파이프라인 여러 개를 연결해 표현식 하나로 만들 수 있다
- 기본적으로 스트림 파이프라인은 순차적으로 수행된다
  - parallel 메서드 : 파이프라인 병렬 실행(효과를 보는 상황은 많지 않다)

#### 스트림과 코드블록(반복) 선택 지침

##### 예시) 아나그램 그룹 출력

```java
/* 아나그램 그룹 출력 */
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                // 맵 안에 키가 있는지 찾고, 있으면 단순히 그 키에 매핑된 값을 반환
                // 키가 없으면 건네진 함수 객체를 키에 적용,값을 계산후 키-값 매핑, 계산된 값 반환
                groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values()) {
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

```java
/* 아나그램 그룹 출력 - 스트림을 과용함 */
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word ->
                    word.chars().sorted().collect(StringBuilder::new, (sb, c) -> sb.append((char) c),
                            StringBuilder::append).toString())).values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group).forEach(System.out::println);
        }
    }
}
```

- 사전 파일을 여는 부분을 제외하면 프로그램 전체가 단 하나의 표현식으로 처리되고 있다
  - 짧지만 읽고 이해하기가 어렵다
- 사전을 여는 작업이 분리된 이유?
  - try-with-resource 문을 사용, 사전 파일을 제대로 닫기 위해

````java
/* 아나그램 그룹 출력 - 적절한 스트림 활용 */
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }
    
    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
````

- try-with-resource 블록에서 사전 파일을 연다
- 파일의 모든 라인으로 구성된 스트림을 얻는다
  - 스트림 변수의 이름을 words로 지어 스트림 안의 원소가 단어(word)임을 명시
- 이 스트림의 파이프라인에 중간 연산은 없으며, 종단 연산에서는 모든 단어를 수집해 맵으로 모은다
- 람다 매개변수의 이름은 주의해서 정해야 한다
  - 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다
- 스트림 파이프라인에서 도우미 메서드를 적절히 활용하라
  - 연산에 적절한 이름을 주고 세부 구현을 주 프로그램 로직 밖으로 빼 가독성을 높일 수 있다
  - 파이프라인에서는 타입 정보가 명시되지 않거나 임시 변수가 자주 사용된다
- alphabetize 메서드도 스트림을 사용해 구현 가능하다
  - 하지만 명확성이 떨어지고, 잘못 구현할 가능성이 있다
  - 자바가 기본 타입인 char용 스트림을 지원하지 않는다
    - 성능이 나빠진다
    - char값 처리 스트림은 사용하지 않는 것이 좋다

##### 함수 블록으로는 할 수 없지만 코드블록으로 할 수 있는 일

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다
  - 람다에서는 final이거나 사실상 final인 변수를 읽기만 가능하다(지역 변수 수정은 불가능)
- 코드 블록에서는 return 문을 사용해 메서드를 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 한번 건너뛸 수 있다

##### 스트림으로 처리하기 좋은 작업

- 원소들의 시퀀스를 일관되게 변환
- 원소들의 시퀀스를 필터링
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기,연결하기,최솟값 구하기...)
- 원소들의 시퀀스를 (공통된 속성을 기준으로) 컬렉션에 모은다
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다

##### 스트림으로 처리하기 어려운 작업

- 한 데이터가 파이프라인의 여러 단계를 통과할 때, 이 데이터의 각 단계에서의 값들에 동시에 접근하기 어려운 경우
  - 스트림 파이프라인은 한 값을 다른 값에 매핑하고 나면 원래의 값을 잃는다

##### 예시) 메르센 소수 출력 프로그램

```java
// 무한 스트림을 반환하는 메서드
static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
// 20개의 메르센 소수 출력
public static void main(String[] args) {
    primes().map(p->TWO.pow(p.intValueExact()).subtract(ONE))
            .filter(mersenne->mersenne.isProbablaPrime(50))
            .limit(20)
            .forEach(System.out::println);
}
```

```java
// 메르센 소수의 앞에 지수를 출력하고 싶은 경우
public static void main(String[] args) {
    primes().map(p->TWO.pow(p.intValueExact()).subtract(ONE))
            .filter(mersenne->mersenne.isProbablaPrime(50))
            .limit(20)
            .forEach(mp -> System.out.println(mp.bitLength()+": "+mp));
}
```

- 지수값은 초기 스트림에 나타나므로 종단 연산에서는 접근 불가
- 첫번째 중간 연산에서 수행한 매핑을 거꾸로 수행, 메르센 수의 지수를 쉽게 계산 가능하다
  - 지수: 숫자를 이진수로 표현한 후 몇 비트인지 세면 나옴

##### 예시) 카드 덱 초기화를 위한 데카르트 곱 계산

- 카드 덱 초기화
  - 카드는 숫자(rank), 무늬(suit)를 묶은 불변 값 클래스
  - 숫자와 무늬는 모두 열거 타입
  - 두 집합의 원소들로 만들 수 있는 가능한 모든 조합을 계산한다
    - 두 집합의 데카르트 곱을 찾는다

```java
/* 데카르트 곱 계산 - 반복 방식 구현 */
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values())
        for (Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}
/* 데카르트 곱 계산 - 스트림으로 구현 */
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
        .flatMap(Suit -> Stream.of(Rank.values()).map(rank -> new Card(suit, rank)))
        .collect(toList());
}
```

- flatMap() : 스트림의 원소 각각을 하나의 스트림으로 매칭 -> 그 스트림들을 다시 하나의 스트림으로 합친다
  - 평탄화(flattening)

-----

### [item46] 스트림에서는 부작용 없는 함수를 사용하라

#### 스트림 패러다임

- 계산을 일련의 변환(transformation)으로 재구성
- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수
  - 입력만이 결과에 영향을 주는 함수
  - 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다
  - 스트림 연산에 건네는 함수 객체는 모두 부작용(side effect)이 없어야 한다

##### 예시) 텍스트 파일에서 단어별 빈도수 체크, 빈도표 제작

```java
/* 스트림 코드를 가장한 반복적 코드 */
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()){
    words.forEach(word -> {freq.merge(word.toLowerCase(),1L,Long::sum);});
}
```

- 스트림, 람다, 메서드 참조를 사용함
- 스트림 API의 이점을 살리지 못함
  - 길고 어려우며 유지보수성도 나쁨
  - forEach()에서 외부 상태(빈도표)를 수정하는 작업을 하며 문제 발생

```java
/* 스트림의 올바른 활용 */
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()){
    freq = words.collect(groupingBy(String::toLowerCase,counting()));
}
```

##### forEach() 연산

- 자바 프로그래머들이 익숙하기 때문에 많이 사용한다
- for-each 반복문과 forEach 종단 연산은 비슷하게 생겼다
- forEach 연산은 종단 연산중 기능이 가장 적고 가장 덜 스트림답다
- 반복적이기 때문에 병렬화도 불가능하다
- **forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고 계산할때는 사용하지 않는 것이 좋다**

 #### Collectors의 메서드들

##### 수집기(collector)

- [java.util.stream.Collectors](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) 클래스는 메서드를 39개나 가지고 있다
  - 자바 10에서는 4개가 추가되어 총 43개이다
    - toUnmodifiableMap() 2개, toUnmodifiableSet(), toUnmodifiableList()
  - 축소(reduction) 전략(스트림의 원소들을 객체 하나에 취합)을 캡슐화한 블랙박스 객체로 생각하자
- 수집기(collector)가 생성하는 객체는 일반적으로 컬렉션이다
- 수집기를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다
- toList(),toSet(),toCollection(collectionFactory)....

##### toList()

```java
/* 빈도표에서 가장 흔한 10단어를 뽑는 스트림 파이프라인 */
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reserved())
    .limit(10)
    .collect(toList());
```

- toList는 Collectors의 메서드
  - Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인의 가독성이 좋아진다
- comparing 메서드 : 키 추출 함수를 받는 비교자 생성 메서드
- freq::get : 입력받은 단어(키)를 빈도표에서 찾아 그 빈도를 반환
- sorted를 이용해 비교자를 역순으로 정렬(가장 흔한 단어가 위로 오도록)

##### toMap(keyMapper, valueMapper)

```java
/* 문자열 - 열거 타입 상수 매핑 */
private static final Map<String, Operation> StringToEnum = 
    Stream.of(values()).collect(
						toMap(Object::toString, e->e));
```

- 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다
- 스트림의 각 원소가 고유한 키에 매핑되어있을때 적합하다
  - 스트림 원소 다수가 같은 키를 사용한다면 IllegalStateException 발생

##### toMap(keyMapper, valueMapper, `BinaryOperator<U>`)

```java
/* 키와 해당 키의 특정 원소 연관 */
Map<Artist, Album> topHits = albums.collect)
    toMap(Album::artist, a->a, maxBy(comparing(Album::sales)));
```

- 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관짓는 맵을 만든다
- 비교자로 BinaryOperator에서 정적 임포트한 maxBy 정적 팩터리 메서드 사용
- 앨범 스트림을 맵으로 변환하는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다

##### toMap(keyMapper, valueMapper, (oldVal, newVal)->newVal)

- 충돌이 나면 마지막 값을 취하는 수집기
- 많은 스트림의 결과가 비결정적
- 매핑 함수가 키 하나에 연결된 값이 모두 같을때, 혹은 값이 다르더라도 모두 허용되는 값일때 이렇게 동작하는 수집기가 필요

##### toMap(keyMapper, valueMapper, `BinaryOperator<U>`, collectionFactory)

- 네번째 인수로 맵 팩터리(EnumMap, TreeMap...)를 받는다

##### toConcurrentMap()

- 병렬 실행 된 후 결과로 ConcurrentHashMap 인스턴스 생성

##### groupingBy()

```java
words.collect(groupingBy(word -> alphabetize(word)))
```

- 입력으로 분류 함수를 받고, 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환
- 분류 함수는 입력받은 원소가 속하는 카테고리를 반환, 카테고리는 맵의 키로 사용됨
- groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하려면?
  - 분류 함수와 함께 다운스트림 수집기를 명시

```java
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase,counting()))
```

- 다운스트림 수집기로 counting()을 건넴
- 각 카테고리(키)를 해당 카테고리에 속하는 원소의 개수(값)와 매핑한 맵을 얻는다
- 또 다른 버전: 다운스트림 수집기+ 맵 팩터리 지정
  - 점층적 인수 목록 패턴에 어긋난다

##### minBy(), maxBy()

- 인수로 받은 비교자를 이용, 스트림에서 값이 가장 작은 혹은 가장 큰 원소를 찾아 반환
- Stream 인터페이스의 min과 max 메서드를 일반화
- java.util.function.BinaryOperator의 minBy, maxBy 메서드가 반환하는 이진 연산자의 수집기 버전

##### joining()

- (문자열 등)CharSequence 인스턴스의 스트림에만 적용
- 매개변수가 없는 joining: 단순히 원소들을 연결하는 수집기를 반환
- 매개변수 1개: CharSequence 타입 구분 문자를 매개변수로 받아 연결 부위에 이 구분문자를 삽입
  - 예시) ','을 삽입해 CSV 형태의 문자열을 만든다
- 매개변수 3개: 구분 문자+접두문자+접미문자

-----

### [item47] 반환 타입으로는 스트림보다 컬렉션이 낫다

- 원소 시퀀스를 반환하는 메서드를 작성할 때
  - 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나, 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 표준 컬렉션에 담아 반환(ArrayList 등)
  - 전용 컬렉션을 구현해 반환
  - 컬렉션에 담아 반환하는 것이 불가능한 경우
    - 스트림과 Iterable 중 더 자연스러운 것 반환
    - 나중에 Stream 인터페이스가 Iterable을 지원하도록 자바가 수정된다면 그때는 스트림을 반환하면 된다

#### 원소 시퀀스를 반환하는 메서드

##### 스트림으로 반환, 스트림으로 처리

- 스트림은 반복(iteration)을 지원하지 않는다
- Stream은 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고, 정의한 방식대로 동작한다
  - 하지만 Iterable을 확장(extend) 하지 않았기 때문에, for-each로 스트림을 반복할 수 없다

```java
for(ProcessHandle ph : ProcessHandle.allProcesses()::iterator){
    // 프로세스 처리
}
```
- 컴파일 오류 발생: method reference not expected here

```java
// Iterable로 메서드 참조 형변환
for(ProcessHandle ph : (Iterabel<ProcessHandle>)ProcessHandle.allProcesses()::iterator){
    // 프로세스 처리
}
```

- 작동하기는 하지만 너무 난잡하고 직관성이 떨어진다

```java
// Stream<E>를 Iterable<E>로 중개해주는 어댑터 메서드 제공
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}
...
for(ProcessHandle p : iterableOf(ProcessHandle.allProcesses())){
    // 프로세스 처리
}
```

- 자바의 타입 추론이 문맥을 잘 파악하여 어댑터 메서드 안에서 따로 형변환하지 않아도 된다
- 반환된 객체들이 반복문에서만 사용될 것을 안다면 Trerable 반환

```java
// Iterable<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
	return StreamSupport.stream(iterable.spliterator(),false);
}
```

- 이 메서드가 오직 스트림 파이프라인에서만 사용될 것을 안다면 스트림 반환

##### 컬렉션으로 반환(반복으로 처리)

- Collection, Set, List와 같은 컬렉션 인터페이스, 혹은 Iterable이나 배열
- Collection 인터페이스는 Iterable의 하위 타입이고, stream 메서드도 제공하여 반복과 스트림을 동시에 지원한다
- 원소 시퀀스를 반환하는 공개API의 반환 타입에는 Collection이나 그 하위 타입을 사용하는 것이 최선이다
- 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션 구현을 고려해본다

##### 전용 컬렉션 구현 예시) 멱집합

```java
/* AbstractList 활용, 멱집합 전용 컬렉션 구현 */
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최개 30개): " + s);
        // 30을 넘을 수 없는 이유: Collection의 size 메서드가 int값을 반환, 시퀀스의 최대 길이가 Integer.MAX_VALUE로 제한

        return new AbstractList<Set<E>>() {
            @Override
            public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }

            @Override
            public int size() {
                // 멱집합의 크기 == 2을 원래 집합의 원소 수만큼 거듭제곱한 것
                return 1 << src.size();
            }

            @Override
            public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set) o);
            }
        };
    }
}

```

- 원소 개수가 n이면 멱집합의 원소 개수는 2^n이므로 표준 컬렉션 사용은 위험하다
- AbstractList를 이용해 전용 컬렉션 구현
  - 각 원소의 인덱스를 비트 벡터로 사용
  - contains, size 구현
    - 이게 불가능하다면 Iterable을 반환
- 코드는 훨씬 지저분하지만, 스트림을 활용하는 구현보다 빠르다

##### 예시) 입력 리스트의 연속적인 부분 리스트를 모두 반환하는 메서드

```java
public class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubLists::suffixes));
    }
    public static <E> Stream<List<E>> prefixes(List<E> list){
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end->list.subList(0,end));
    }
    public static <E> Stream<List<E>> suffixes(List<E> list){
        return IntStream.rangeClosed(0, list.size())
                .mapToObj(start->list.subList(start, list.size()));
    }
}
```

- 컬렉션에 담는 코드는 3줄이면 충분하지만, 입력 리스트 크기의 거듭제곱만큼 메모리를 차지한다
- 스트림으로 구현하면 직관적이다

```java
/*
	for(int start=0;start<src.size();start++){
     	for(int end=start+1;end<=src.size();end++)
       		System.out.println(src.subList(start,end));
     }
     */
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0,list.size())
        .mapToObj(Start -> 
                  IntStream.rangeClosed(start+1,list.size())
                 		.mapToObj(end->list.subList(start,end)))
        .flatMap(x->x);
}

```

-----

### [item48] 스트림 병렬화는 주의해서 적용하라

#### 자바 동시성 프로그래밍의 역사

- 첫 릴리즈(1996)
  - 스레드,동기화,wait/notify 지원
- 자바 5
  - [java.util.concurrent](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/package-summary.html) 라이브러리 지원
  - Executer 프레임워크 지원
- 자바 7
  - 포크-조인(fork-join) 패키지
    - 고성능 병렬 분해(parallel decomposition) 프레임워크
- 자바 8
  - parallel 메서드 호출로 파이프라인 병렬 실행 가능한 스트림 지원
- 동시성 프로그래밍을 할 때에는 안전성(safety)과 응답 가능 상태(liveness)를 유지해야 한다

#### 스트림 파이프라인의 parallel()을 호출하면 성능이 무조건 좋아질까?

- 스트림 라이브러리가 파이프라인을 병렬화하는 방법을 찾아내지 못하면 응답 불가 상태가 된다
- 데이터소스가 Stream.iterato거나, 중간 연산으로 limit을 사용한다면 파이프라인 병렬화가 불가능하다
- 따라서 스트림 파이프라인을 마구잡이로 병렬화한다면 성능이 오히려 끔찍하게 나빠질 수 있다

#### 스트림과 병렬화

##### 성능 개선을 기대할 수 있는 경우

- 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나, 배열, int범위, long 범위일때 병렬화의 효과가 좋다
  - 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어, 일을 다수의 스레드에 분배하기 좋다
    - 나누는 작업은 Spliterator가 담당한다
    - Spliterator 객체는 Stream이나 Iterable의 spliterator 메서드로 얻어올 수 있다
  - 원소들을 순차적으로 실행할 때의 **참조 지역성**(locality of reference)이 뛰어나다
    - 이웃한 원소들의 참조들이 메모리에 연속해서 저장되어 있다
    - 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져 있는 경우는 참조 지역성이 나빠진다
    - 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할때 아주 중요한 요소로 작용한다
    - 참조 지역성이 가장 뛰어난 자료구조 : 기본 타입의 배열(데이터 자체가 메모리에 연속해서 저장)
- 종단 연산의 동작 방식도 병렬 수행 효율에 영향을 준다
  - 종단 연산에서 수행하는 작업량이 적고 순차적인 연산이라면 병렬수행 효과는 제한된다
  - 병렬화에 가장 적합한 것은 **축소**(reduction) 종단 연산
    - 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업
    - Stream의 reduce 메서드 중 하나 또는 min,max,count,sum 등의 완성된 형태의 메서드 중 하나를 선택해 반환
    - anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드도 적합
  - 가변 축소(mutable reduction)를 수행하는 Stream의 collect 메서드는 병렬화에 적합하지 않다
    - 컬렉션을 합치는 부담이 크기 때문이다
- 직접 구현한 Stream, Iterable, Collection이 병렬화의 이점을 누리게 하려면
  - spliterator 메서드를 반드시 재정의
  - 결과 스트림의 병렬화 성능을 강도 높게 테스트하라

##### 스트림 병렬화시 주의점

- 스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다
  - **안전 실패**(safety failure)
    - 병렬화한 파이프라인이 사용하는 mappers, filters, 프로그래머가 제공한 다른 함수 객체가 명세대로 동작하지 않을때 발생
    - Stream 명세는 이때 사용되는 함수 객체에 대해 엄중한 규약을 정의
      - 예시) Stream의 reduce 연산에 건네지는 accumulator, combiner 함수는 
        - 반드시 결합법칙을 만족하고(associative)`(a op b) op c== a op (b op c)`
        - 간섭받지 않고(non-interfering)
        - 상태를 갖지 않아야 한다(stateless)
      - 위의 조건이 지켜지지 않은 경우에도 순차적으로 수행한다면 올바른 결과를 얻을 수 있으나, 병렬 수행시에는 실패로 이어진다
- 파이프라인이 수행하는 진짜 작업이 병렬화에 드는 추가 비용을 상쇄하지 못한다면 성능 향상은 미미할 수 있다
  - (스트림 안의 원소 수 * 원소당 수행되는 코드 줄 수 >= 최소 수십만 줄) 일 때 성능 향상을 맛볼 수 있다
- 병렬화 전후로 반드시 성능을 테스트하여 병렬화를 사용할 가치가 있는지 확인해야 한다
- 조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 맛볼 수 있다
  - 머신러닝, 데이터 처리와 같은 분야에서 활용
- 무작위 수들로 이루어진 스트림 병렬화시
  - ThreadLocalRandom보다는 SplittableRandom 인스턴스 활용
    - SplittableRandom 인스턴스 활용시 병렬화하면 성능이 선형으로 증가한다
    - ThreadLocalRandom은 단일 스레드에서 사용하고자 만들어졌다
    - Random 연산은 모든 연산을 동기화해 병렬 처리하면 최악의 성능을 보인다

#### 예시) 소수 계산 스트림 파이프라인

```java
/* 일반적인 소수 계산 스트림 파이프라인 */
static long pi(long n){
    return LongStream.rangeClosed(2, n)
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
/* 병렬화를 통해 성능 개선 */
static long pi(long n){
    return LongStream.rangeClosed(2, n)
        .parallel()
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
```

- 하지만 n이 크다면 이 방식으로 계산하는 것은 좋지 않다
  - [레머의 공식](https://en.wikipedia.org/wiki/Meissel%E2%80%93Lehmer_algorithm)이라는 훨씬 효율적인 알고리즘이 존재한다



