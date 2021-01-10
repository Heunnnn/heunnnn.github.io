---

layout: post
title: 210103 Effective Java 6장 정리
tags:
  - Effective Java
---

## 6. 열거 타입과 애너테이션

### [item34] int 상수 대신 열거 타입을 사용하라

#### 정수 열거 패턴

- 정수 상수를 한 묶음 선언하여 사용한다
- 단점
  - 타입 안전을 보장할 방법이 없다
  - 표현력이 좋지 않다
  - 평범한 상수의 나열-컴파일시 그 값이 클라이언트 파일에 그대로 새겨진다
    - 상수의 값이 바뀌면, 클라이언트도 다시 컴파일해야 한다
  - 문자열로 출력하기 까다롭다

#### 문자열 열거 패턴

- 정수 열거 패턴보다 더 나쁜 방식이다!
  - 상수의 의미를 출력할 수 있다
  - 경험이 부족한 프로그래머가 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만든다
  - 런타임 오류 발생, 문자열 비교에 따른 성능 저하 발생

#### 열거 타입(Enum type)

```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

- 자바의 열거 타입은 완전한 형태의 클래스이다
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다
- 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final 이다.
  - private로 두고, 별도의 public 접근자 메서드를 둔다.
- 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없다
  - 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재한다
  - 열거 타입은 인스턴스 통제된다
  - 싱글턴을 일반화한 형태이다
- 컴파일 타임 타입 안전성을 제공한다
- 열거 타입마다 각자의 네임스페이스가 존재해 이름이 같은 상수도 존재할 수 있다
- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다
  - 공개되는 것은 필드의 이름뿐이기 때문에 상수값이 클라이언트로 컴파일되어 각인되지 않는다
- toString 메서드로 출력하기에 적합한 문자열을 내어준다
- 열거 타입에는 임의의 메서드나 필드를 추가할 수 있으며, 임의의 인터페이스를 구현하도록 할 수 있다
  - Object 메서드들을 높은 품질로 구현, Comparable, Serializable도 구현했다
  - 각 상수와 연관된 데이터를 해당 상수에 내재하려고 할 때
    - Apple, Orange: 과일의 색을 알려주거나, 과일 이미지를 반환하는 메서드
  - 열거 타입 상수 각각을 특정 데이터와 연결지을때
    - 생성자에서 데이터를 받아 인스턴스 필드에 저장
- values
  - 열거 타입 안에 정의된 상수들의 값을 배열에 담아 반환
  - 값은 선언된 순서로 저장된다
- toString
  - 상수 이름을 문자열로 반환, printf와 println에서 사용하기 좋다
- 열거 타입에서 상수를 하나 제거한다면?
  - 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다
  - 제거한 상수를 참조하는 클라이언트: 다시 컴파일시 컴파일 오류가 발생, 다시 컴파일하지 않는다면 런타임 오류 발생
- 일반 클래스와 마찬가지로 그 기능을 클라이언트에 노출해야 하는 이유가 없다면 private / package-private으로 선언한다.
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만든다
  - 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다

#### 상수별 메서드 구현을 활용한 열거 타입

```java
/* 값에 따라 분기하는 열거 타입 */
public enum Operation {
    PLUS,MINUS,TIMES,DIVIDE;
    
    public doulble apply(double x, double y){
        switch(this){
            case PLUS: return x+y;
            case MINUS: return x-y;    
            case TIMES: return x*y;
            case DIVIDE: return x/y;
        }
        throw new AssertionError("알 수 없는 연산: "+this);
    }
}
```

- 마지막 throw 문에서...
  - 실제로는 도달할 일이 없으나 기술적으로는 도달할 수 있어, 생략한다면 컴파일되지 않는다.
- 깨지기 쉬운 코드이다
  - 새로운 상수 주가시 해당 case문도 꼭 추가해야 한다

```java
/* 상수별 메서드 구현을 활용한 열거 타입 */
public enum Operation {
    PLUS {public double apply(double x, double y){return x+y;}},
    MINUS {public double apply(double x, double y){return x-y;}},
    TIMES {public double apply(double x, double y){return x*y;}},
    DIVIDE {public double apply(double x, double y){return x/y;}};
    
    public abstract double apply(double x, double y);
}
```

- apply라는 추상 메서드 선언, 각 상수병 클래스 몸체, 각 상수에서 자신에 맞게 재정의한다.

```java
/* 상수별 클래스 몸체와 데이터를 사용한 열거 타입 */
public enum Operation {
    PLUS("+") {public double apply(double x, double y){return x+y;}},
    MINUS("-") {public double apply(double x, double y){return x-y;}},
    TIMES("*") {public double apply(double x, double y){return x*y;}},
    DIVIDE("/") {public double apply(double x, double y){return x/y;}};
    
    private final String symbol;
    Operation(String symbol) {this.symbol=symbol;}
    @Override public String toString() {return symbol;}
    public abstract double apply(double x, double y);
    
    private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(toMap(Object::toString,e->e));
    // 지정한 문자열에 해당하는 Operation이 존재한다면 반환한다.
    public static Optional<Operation> fromString(String symbol){
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
}
```

- 상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다
- 열거 타입의 toString 메서드를 재정의시, toString이 반환하는 문자열을 해당 열거타입 상수로 변환하는 fromString 메서드도 함께 제공하는 것을 고려하자
  - Operation 상수가 stringToEnum맵에 추가되는 시점: 열거 타입 상수 생성 후 정적 필드가 초기화될때.
- 하지만 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다
  - 모든 상수에 중복하여 넣기
  - 도우미 메서드를 작성하여 각 상수가 자신에게 필요한 메서드를 호출하도록 하기

#### 열거 타입을 사용하기 좋은 상황들

- 필요한 원소를 컴파일타임에 다 알 수 있는 사웃 집합
  - 태양계 행성, 한 주의 요일, 체스 말..
  - 메뉴 아이템, 연산 코드, 명령줄 플래그...
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다
  - 나중에 상수가 추가되더라도 바이너리 수준에서 호환되도록 설계되어 있다

-----

### [item35] ordinal 메서드 대신 인스턴스 필드를 사용하라

#### ordinal

- 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 메서드
- 메서드 내부에서 이 메서드를 사용하게 된다면, 상수 선언 순서를 바꾸는 순간 메서드가 오동작하게 된다
- 더미 상수를 추가해야 할 수도 있다
- 따라서 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하는 편이 낫다

```java
public enum Ensemble{
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8),
    DOUBLE_QUARTET(8), NONET(9), DECTET(10), TRIPLE_QUARTET(12);
    
    private final int numberOfMusicians;
    Ensemble(int size){this.numberOfMusicians = size;}
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

- 실제로, Enum api문서의 ordinal에 대한 설명은 아래와 같다.

- > ... Most programmers will have no use for this method. 
  >
  > It is designed for use by sophisticated enum-based data structures, such as [`EnumSet`](https://docs.oracle.com/javase/7/docs/api/java/util/EnumSet.html) and [`EnumMap`](https://docs.oracle.com/javase/7/docs/api/java/util/EnumMap.html).
  
  대부분의 프로그래머는 이 메서드를 쓸 일이 없다. 
  
  이 메서드는 EnumSet과 EnumMap과 같은 열거 타입 기반 범용 자료구조에 쓸 목적으로 설계되었다.

-----

### [item36] 비트 필드 대신 EnumSet을 사용하라

#### 비트 필드 사용

```java
/* 비트 필드 열거 상수 */
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) { ... }
}
```

- ```java
  text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
  ```

- 비트 필드: 비트별 OR을 사용하여 여러 상수를 하나의 집합으로 모을 수 있다

- 비트별 연산을 사용, 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다

- 단점

  - 정수 열거 상수의 단점을 그대로 지닌다

    - >- 타입 안전을 보장할 방법이 없다
      >- 표현력이 좋지 않다
      >- 평범한 상수의 나열-컴파일시 그 값이 클라이언트 파일에 그대로 새겨진다
      >  - 상수의 값이 바뀌면, 클라이언트도 다시 컴파일해야 한다
      >- 문자열로 출력하기 까다롭다

  - 비트 필드값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다

  - 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다

  - 최대 몇 비트가 필요한지를 API작성시 미리 예측하여 적절한 타입(보통 int나 long)을 선택해야 한다

    - API를 수정하지 않고서는 비트 수를 늘릴 수 없기 때문이다

#### EnumSet - 비트 필드를 대체한다

```java
/* EnumSet: 비트 필드를 대체하는 현대적 기법 */
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    // 어떤 Set을 넘겨도 되지만, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) { ... }
}
```

- ```java
  text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
  ```

- EnumSet은 집합 생성 등 다양한 기능의 정적 팩터리를 제공

  - 비트 필드 수준의 명료함과 성능을 제공함
  - 열거 타입의 장점을 제공한다

- applyStyles 메서드가 `EnumSet<Style>`이 아닌 `Set<Style>`을 받은 이유는?

  - 모든 클라이언트가 EnumSet을 넘길 것이라고 짐작되는 상황이라도, 이왕이면 인터페이스로 받는게 일반적으로 좋은 습관
    - 다른 Set 구현체를 넘기더라도 처리할 수 있다

- 단점

  - 불변 EnumSet을 만들 수 없다
    - Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다
      - 성능과 명확성이 떨어진다
    - Guava 라이브러리의 불변 EnumSet(https://guava.dev/releases/21.0/api/docs/com/google/common/collect/Sets.html#immutableEnumSet-E-E...-)
      - 내부에서는 EnumSet을 이용해 구현하였기 때문에 성능 면에서 손해

-----

### [item37] ordinal 인덱싱 대신 EnumMap을 사용하라

item35의 특수한 사례 중 하나이다.

#### 예제: ordinal()을 배열 인덱스로 사용

```java
class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

정원에 심은 식물들을 배열 하나로 관리,

- 이들을 생애주기(한해살이, 여러해살이, 두해살이) 별로 묶는다
  - 생애주기별 총 3개의 집합을 만들고, 정원을 한 바퀴 돌며 각 식물을 해당 집합에 넣는다
  - 집합들을 배열 하나에 넣고 생애주기의 ordinal 값을 그 배열의 인덱스로 사용하려 한다

```java
/* ordinal()을 배열 인덱스로 사용 - 따라 하지 말것! */
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for(int i=0;i<plantsByLifeCycle.length;i++)
    plantsByLifeCycle[i]=new HashSet<>();
for(Plant p : garden)
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

for(int i=0;i<plantsByLifeCycle.length;i++)
    System.out.printf("%s: %s%n",Plants.LifeCycle.values()[i],PlantsByLifeCycle[i]);
```

- 위 코드는 동작은 하지만 문제가 많다
  - 배열은 제네릭과 호환되지 않으므로 비검사 형변환을 수행해야 하고, 깔끔하게 컴파일되지 않는다
  - 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다
  - 정확한 정수값을 사용한다는 것을 직접 보증해야 한다
    - 정수는 열거 타입과 다르게 타입 안전하지 않다

#### 해결 방법) EnumMap을 사용하여 데이터와 열거 타입 매핑

```java
Map<Plant.LifeCycle,Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for(Plant.LifeCycle lc: Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc,new HashSet<>());
for(Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```

- 더 짧고 명료하며, 안전하고 성능도 원래 버전과 비등하다
  - EnumMap의 내부에서 배열을 사용한다 
    - 내부 구현 방식을 안으로 숨겨 Map의 타입 안전성과 배열의 성능을 모두 얻는다
    - EnumMpa의 생성자가 받는 키 타입 Class객체: 한정적 타입 토큰, 런타임 제네릭 타입 정보를 제공한다
- 안전하지 않은 형변환은 사용하지 않는다
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하므로, 출력 결과에 직접 레이블을 달 필요도 없다
- 배열 인덱스 계산 과정에서 오류가 날 가능성도 없다

#### 스트림을 사용한 EnumMap 관리

```java
/* 1. EnumMap을 사용하지 않음 */
System.out.println(Arrays.stream(garden).collect(groupringBy(p->p.lifeCycle)));
```

- EnumMpa이 아닌 고유한 맵 구현체 사용, EnumMap을 써서 얻은 공간과 성능 이점이 사라진다

```java
/* 2. EnumMap을 사용, 데이터와 열거 타입 매핑 */
System.out.println(Arrays.stream(garden).collect(groupringBy(p->p.lifeCycle,
                                                             () -> new EnumMap<>(LifeCycle.class), toSet()
                                                            )));
```

- MapFactory 매개변수에 원하는 맵 구현체를 명시해 호출
- 스트림을 사용하면 EnumMap만 사용했을때와는 살짝 다르게 동작
  - EnumMap: 언제나 식물의 생애주기당 하나의 중첩 맵을 만든다
  - 스트림: 해당 생애주기에 속하는 식물이 있을 때만 만든다

#### 예제: 두 열거 타입 값을 매핑하기 위한 ordinal()을 두번 사용

```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        
        //행을 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다
        private static final Transition[][] TRANSITIONS = {
            {null, MELT, SUBLIME};
            {FREEZE , null, BOIL},
            {DEPOSIT, CONDENSE,	null}
        };
        
        // 한 상태에서 다른 상태로의 전이를 반환
        public static Transition from(Phase from, Phase to){
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

- 컴파일러는 ordinal과 배열 인덱스의 관계를 알 수 없다
  - Phase나 Phase.Transition 열거 타입을 수정하면서 상전이 표인 TRANSITIONS를 함께 수정하지 않거나 잘못 수정한다면 런타임 오류가 발생한다
  - 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며, null로 채워지는 칸도 늘어난다.
  - 따라서 EnumMap을 사용하는 편이 낫다

```java
/* 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결 */
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT(SOLID,LIQUID),
        FREEZE(LIQUID,SOLID), 
        BOIL(LIQUID,GAS), 
        CONDENSE(GAS,LIQUID), 
        SUBLIME(SOLID,GAS), 
        DEPOSIT(GAS,SOLID);
        
        private final Phase from;
        private fianl Phase to;
        
        Transition(Phase from, Phase to){
            this.from = from;
            this.to = to;
        }
        
        // 상전이 맵 초기화
        private static final Map<Phase, Transition>>
            m = Stream.of(values()).collect(groupingBy(t->t.from, 
                                                      ()->new EnumMap<>(Phase.class),
                                                      toMap(t->t.to, t->t,
                                                           (x,y)->y,()->new EnumMpa<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

- `Map<Phase, Map<Phase,Transition>>`

  - 이전 상태에서 '이후 상태에서 전이로의 맵' 에 대응시키는 맵

- 맵의 맵을 초기화하기 위해 수집기 2개를 차례로 사용

  - `groupingBy`
    - 전이를 이전 상태를 기준으로 묶는다
  - `toMap`
    - 이후 상태를 전이에 대응시키는 EnumMap 생성
    - 두번째 수집기의 병합 함수인 (x,y) -> y는 선언만 하고 쓰이지 않는다
      - EnumMap을 얻으려면 맵 팩터리가 필요하고, 수집기들은 점층적 팩터리를 제공하기 때문이다

- 여기에 새로운 상태인 PLASMA를 추가한다

  - 이 상태와 연결되는 전이는 2개이다

    - 기체->플라즈마(IONIZE)
    - 플라즈마->기체(DEIONIZE)

  - 배열로 만큰 코드를 수정하려면 수정 범위가 넓다

    - Phase에 새로운 상수 1개, Phase.Transition에 2개를 추가
    - 원소 9개짜리인 배열들의 배열->원소 16개짜리로 교체
    - 이중 하나라도 잘못되면 런타임 오류 발생

  - EnumMap 버전에서는 상태 목록에 PLASMA를 추가하고, 전이 목록에 IONIZE(GAS,PLASMA), DEIONIZE(PLASMA,GAS)를 추가하면 된다

  - ```java
    public enum Phase {
        SOLID, LIQUID, GAS, PLASMA;
        
        public enum Transition {
            MELT(SOLID,LIQUID), FREEZE(LIQUID,SOLID), 
            BOIL(LIQUID,GAS), CONDENSE(GAS,LIQUID), 
            SUBLIME(SOLID,GAS), DEPOSIT(GAS,SOLID)
            IONIZE(GAS,PLASMA), DEIONIZE(PLASMA,GAS);
            .....
        }
    }
    ```

-----

### [item38] 확장할 수 있는 열거 타입이 필요하다면 인터페이스를 사용하라

#### 타입 안전 열거 패턴과 열거 타입

- 타입 한전 열거 패턴은 확장할 수 있다
  - 열거한 값들을 그대로 가져온 다음, 다음 값을 더 추가하여 다른 목적으로 쓸 수 있다.
- 열거 타입은 확장할 수 없다
- 대부분의 상황에서 열거 타입을 확장하는 것은 좋은 설계가 아니다
  - 확장한 타입의 원소는 기반 타입의 원소로 취급<->그 반대는 성립하지 않는다면?
  - 기반타입과 확장된 타입들의 원소 모두를 순회할 방법도 마땅하지 않다
  - 확장성을 높이려면 고려할 요소가 늘어나므로 설계와 구현이 모두 복잡해진다

#### 확장할 수 있는 열거타입 예시: 연산 코드(operation code 혹은 opcode)

- 연산 코드의 각 원소: 특정 기계가 수행하는 연산

- 이따금 API가 제공하는 기본 연산 외 사용자 확장 연산을 추가할 수 있도록 열어줘야 하는 경우가 있다

- 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용한다

  - 연산 코드용 인터페이스 정의
  - 열거 타입이 이 인터페이스를 구현

- 하지만 단점이 존재한다

  - 열거 타입끼리 구현을 상속할 수 없다

  - 아무 상태에도 의존하지 않는다면 디폴트 구현을 이용, 인터페이스에 추가한다

  - 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리, 코드 중복을 없앤다

    - java.nio.file.LinkOption 열거 타입(https://docs.oracle.com/javase/8/docs/api/java/nio/file/LinkOption.html)

      - CopyOption과 OpenOption 인터페이스를 구현

        - > public enum LinkOption
          > extends Enum<LinkOption>
          > implements OpenOption, CopyOption

```java
/* 인터페이스 구현을 통해 열거타입 구현 */
public interface Operation {
    double apply(double x, double y);
}
public enum BasicOperation implements Operation {
    PLUS("+") {public double apply(double x, double y){return x+y;}},
    MINUS("-") {public double apply(double x, double y){return x-y;}},
    TIMES("*") {public double apply(double x, double y){return x*y;}},
    DIVIDE("/") {public double apply(double x, double y){return x/y;}};
    
    private final String symbol;
    
    BasicOperation(String symbol) {this.symbol=symbol;}
    @Override public String toString() {return symbol;}
}
```

- 열거 타입인 BasicOperation은 확장할 수 없지만, 인터페이스인 Operation은 확장할 수 있으므로 이 인터페이스를 연산의 타입으로 사용한다.
- Operation을 구현한 또 다른 열거타입을 정의해 기본 타입인 BasicOperation을 대체할 수 있다.

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y){
            return Math.pow(x,y);
        }
    },
    REMAINDER("%"){
        public double apply(double x, double y){
        	return x%y;
        }
    }
    private final String symbol;
    
    ExtendedOperation(String symbol) {this.symbol=symbol;}
    @Override public String toString() {return symbol;}
}
```

- 이 연산은 기존 연산을 사용하던 곳이면 어디든지 사용할 수 있다.(Operation 인터페이스를 사용하도록 작성되어 있다면!)
- apply가 인터페이스에 선언되어 있으니, 열거 타입에 따로 추상 메서드로 선언하지 않아도 된다
- 개별 인스턴스 수준에서뿐만 아니라 타입 수준에서도, 기본 열거 타입 대신 확장된 열거 타입을 넘겨서 확장된 열거 타입의 원소 모두를 사용하게 할 수도 있다

```java
/* 기본 열거타입 대신 확정된 열거 타입 Class 객체 넘기기 */
public static void main(String[] args){
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class,x,y);
}
private static <T extends Enum<T> & Operation> void test(
	Class <T> opEnumType, double x, double y){
    for(Operation op:opEnumType.getEnumConstants()){
        System.out.printf("%f %s %f = %f%n",x,op,y,op.apply(x,y));
    }
}
```

- main 메서드는 test메서드에 ExtendedOperation 의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다
  - class 리터럴: 한정적 타입 토큰
- ` <T extends Enum<T> & Operation> Class<T> `
  - Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다
  - 열거 타입이어야 원소 순회 가능
  - Operation이어야 원소가 뜻하는 연산을 수행할 수 있다

```java
/* 한정적 열거 타입 Collection<? extends Operation> 넘기기 */
public static void main(String[] args){
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()),x,y);
}
private static void test(Collection<? extends Operation> opSet, double x, double y){
    for(Operation op:opSet){
        System.out.printf("%f %s %f = %f%n",x,op,y,op.apply(x,y));
    }
}
```

- 덜 복잡하며, test 메서드가 좀더 유연해진다
- 하지만 특정 연산에서는 EnumSet과 EnumMap을 사용할 수 없다

-----

### [item39] 명명 패턴보다 애너테이션을 사용하라

- 애너테이션이 명명 패턴보다 낫다
  - 테스트는 애너테이션으로 할 수 있는 일중 극히 일부
  - 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다

- 다른 프로그래머가 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면 적당한 애너테이션 타입도 함께 정의해서 제공한다

- 도구 제작자를 제외하고는 일반 프로그래머가 애너테이션 타입을 직접 저의할 일은 거의 없다
- 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들을 사용해야 한다
- IDE나 정적 분석 도구가 제공하는 애너테이션을 사용
  - 해당 도구가 제공하는 진단 정보의 품질을 높인다
  - 표준이 아니므로, 도구를 바꾸거나 표준이 만들어진다면 수정 작업을 조금 거쳐야 한다

#### 명명 패턴

- 예시: JUnit 버전 3까지 - 테스트 메서드 이름을 test로 시작하도록 했다
- 단점
  - 오타가 나면 안된다
    - 실수로 이름을 `tsetSafetyOverride`로 짓는다면 JUnit3은 이 메서드를 무시하고 지나친다
    - 개발자는 이 테스트가 (실패하지 않았으므로) 통과했다고 오해할 수 있다
  - 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다
    - 클래스 이름을 `TestSafetyMechanisms`로 지어 JUnit에 던져준다?
      - 개발자들은 이 클래스에 정의된 테스트 메서드들을 수행하기를 기대한다
      - 그러나 JUnit은 클래스 이름에는 관심이 없다
      - 이번에도 JUnit은 경고 메시지조차 출력하지 않으나, 개발자가 의도한 테스트는 전혀 수행되지 않는다
  - 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다
    - 특정 예외를 던져야만 성공하는 테스트, 기대하는 예외 타입을 테스트에 매개변수를 전달해야 하는 경우
      - 예외의 이름을 테스트 메서드 이름에 덧붙인다?
        - 보기에도 나쁘고 깨지기도 쉽다!
        - 컴파일러는 메서드 이름에 덧붙인 문자열이 예외를 가리키는지 알 도리가 없다
        - 테스트를 실행하기 전에는 그런 이름의 클래스가 존재하는지, 예외가 맞는지조차 알 수 없다!

#### 애너테이션의 동작 방식

- 애너테이션: JUnit에서는 4부터 도입됨

#### 예시 1) 마커 애너테이션

```java
/* marker 애너테이션 타입 선언 */
import java.lang.annotation.*;

/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적 메서드 전용이다!
*/
@Retension(RetensionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{}
```

- 위와 같은 애너테이션을 "아무 매개변수 없이 단순히 대상에 마킹한다"는 뜻에서 마커 애너테이션이라고 한다
  - 이 애너테이션 사용시 프로그래머가 Test 이름에 오타를 내거나, 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 낸다
- 메타애너테이션: `Retension`, `Target`
  - `@Retension(RetensionPolicy.RUNTIME)` : Test가 런타임에도 유지되어야 한다
    - 이 메타애너테이션을 생략하면 테스트 도구는 @Test를 인식할 수 없다
  - `@Target(ElementType.METHOD)` : @Test가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다
    - 클래스 선언, 필드 선언 등 다른 프로그램 요소에서는 달 수 없다
- 주석에서 "매개변수 없는 정적 메서드 전용이다"라고 쓰여 있음
  - 이 제약을 컴파일러가 강제할 수 있으면 좋겠으나, 그렇게 하려면 애너테이션 처리기를 직접 구현해야 함
    - 애너테이션 처리기 없이 인스턴스 메서드나 매개변수가 있는 메서드에 단다면, 컴파일은 되지만 테스트 도구를 실행할 때 문제가 된다
  - 애너테이션 처리기 구현은 [javax.annotation.processing 패키지의 API 문서](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/package-summary.html)를 참조하자

```java
/* marker 애너테이션을 실제 적용한 모습 */
public class Sample{
    @Test public static void m1 {} // 성공해야 한다
    public static void m2 {}
    @Test public static void m3 {  // 실패해야 한다
        throw new RunTimeException("실패");
    }
    public static void m4 {}
    @Test public void m5 {}	// 잘못 사용한 예: 정적 메서드 아님
    public static void m6 {}
    @Test public static void m7 {  // 실패해야 한다
        throw new RunTimeException("실패");
    }
    public static void m8 {}
}
```

- 정적 메서드 7개(1,2,3,4,6,7,8) / 4개에 @Test 달려있다(1,3,5,7)
- m3,m7 메서드는 예외를 던지며 m1,m5는 예외를 던지지 않는다
  - m5는 인스턴스 메서드이므로 @Test를 잘못 사용한 예이다
- 총 4개 테스트 메서드중 1개는 성공, 2개는 실패, 1개는 잘못 사용했다
- @Test를 붙이지 않은 나머지 4개 메서드는 테스트 도구가 무시한다

```java
import java.lang.reflect.*;

public class RunTests{
    public static void main(String[] args) throws Exception{
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for(Method m : testClass.getDeclaredMethods()){
            if(m.isAnnotationPresent(Test.class)){
                tests++;
                try{
                    m.invoke(null);
                    passed++;
                } catch(InvocationTargetException wrappedExc){
                    throwable exc = wrappedExc.getCause();
                    System.out.println(m + "실패" + exc);
                } catch (Exception exc){
                    System.out.println("잘못 사용한 @Test: "+m);
                }
            }
        }
        System.out.printf("성공: %d, 실패\: %d%n", passed, tests-passed);
    }
}
```

- 명령줄로부터 완전 정규화된 클래스 이름을 받고, 그 클래스에서 @Test 애너테이션이 달린 메서드를 차례로 호출
- `isAnnotationPresent` : 실행할 메서드를 찾아주는 메서드
- 테스트 메서드가 예외를 던지면 리플렉션 매커니즘이 `InvocationTargetException`으로 감싸서 다시 던진다
  - `InvocationTargetException`을 잡아 원래 예외에 담긴 실패 정보를 추출(`getCause()`), 출력한다
- `InvocationTargetException` 외의 예외가 발생한다면 @Test 애너테이션을 잘못 사용한 것
  - 인스턴스 메서드, 매개변수가 있는 메서드, 호출할 수 없는 메서드 등에 달았다

#### 예시 2) 매개변수 하나를 받는 애너테이션 타입

```java
import java.lang.annotation.*;

/**
* 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
*/
@Retension(RetensionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

- `Class<? extends Throwable>`
  - Throwable을 확장한 클래스의 Class 객체
  - 모든 예외(와 오류) 타입을 수용한다

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1(){	// 성공해야 함
        int i=0;
        i=i/1;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2(){	// 실패해야 함 (다른 예외 발생)
        int[] a= new int[0];
        int i = a[1];
    }
     @ExceptionTest(ArithmeticException.class)
    public static void m3(){}	// 실패해야 함 (예외가 발생하지 않음)
}
```

```java
import java.lang.reflect.*;

public class RunTests{
    public static void main(String[] args) throws Exception{
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for(Method m : testClass.getDeclaredMethods()){
            if(m.isAnnotationPresent(ExceptionTest.class)){
                tests++;
                try{
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n",m);
                } catch(InvocationTargetException wrappedExc){
                    throwable exc = wrappedExc.getCause();
                    Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                    if(excType.isInstance(exc)){
                        passed++;
                    } else{
                        System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
                    }
                } catch (Exception exc){
                    System.out.println("잘못 사용한 @Test: "+m);
                }
            }
        }
        System.out.printf("성공: %d, 실패\: %d%n", passed, tests-passed);
    }
}
```

- 애너테이션 매개변수의 값을 추출, 테스트 메서드가 올바른 예외를 던지는지 확인하는 데 사용한다.
- 형변환 코드가 없으니 ClassCastException 걱정이 없다
  - 문제 없이 컴파일되면 애너테이션 매개변수가 가리키는 예외가 올바른 타입
- 해당 예외의 클래스 파일이 컴파일 타임에는 존재했으나 런타임에 존재하지 않는 경우
  - TypeNotPresentException을 던진다

#### 예시 3) 배열 매개변수를 받는 애너테이션 타입

```java
/**
* 여러 개 명시한 예외중 하나를 던지면 성공하는 테스트 메서드용 애너테이션
*/
@Retension(RetensionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}
```

```java
@ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
public static void doublyBad(){
    List<String> list = new ArrayList<>();
    // IndexOutOfBoundsException 또는 NullPointerException을 던질 수 있다
    list.addAll(5,null);
}
```

```java
import java.lang.reflect.*;

public class RunTests{
    public static void main(String[] args) throws Exception{
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for(Method m : testClass.getDeclaredMethods()){
            if(m.isAnnotationPresent(ExceptionTest.class)){
                tests++;
                try{
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n",m);
                } catch(InvocationTargetException wrappedExc){
                    throwable exc = wrappedExc.getCause();
                    int oldPassed=passed;
                    Class<? extends Throwable> excTypes = m.getAnnotation(ExceptionTest.class).value();
                    for(Class<? extends Throwable> excType : excTypes){
                    	if(excType.isInstance(exc)){
                        	passed++;
                            break;
                    	}
                    }
                    if(passed == oldPassed)
	                    System.out.printf("테스트 %s 실패: %s%n", m, exc);
                    }
                } 
            }
        }
        System.out.printf("성공: %d, 실패\: %d%n", passed, tests-passed);
    }
}
```

#### 예시 4) @Repeatable 메타애너테이션 사용

```java
/**
* 반복 가능 애너테이션 사용
*/
@Retension(RetensionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

//컨테이너 애너테이션
@Retension(RetensionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

- @Repeatable 애너테이션 사용시 주의점
  - @Repeatable 을 단 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더 정의, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다
  - 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다
  - 컨테이서 애너테이션에는 적절한 보존 정책(@Retension)과 적용 대상(@Target)을 명시해야 한다

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad(){...}
```

```java
if(m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)){
    tests++;
    try{
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n",m);
    } catch (Throwable wrappedExc){
        throwable exc = wrappedExc.getCause();
        int oldPassed=passed;
        ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
        for(ExceptionTest excTest : excTests){
            if(excTest.value().isInstance(exc)){
                passed++;
                break;
            }
        }
        if(passed == oldPassed)
	       System.out.printf("테스트 %s 실패: %s%n", m, exc);
    }
}
```

- 처리할 때에도 주의해야 한다
- 반복 가능 애너테이션을 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다.
  - `getAnnotationsByType` 메서드는 이 둘을 구분하지 않기에 반복 가능 애너테이션과 컨테이너 애너테이션을 가져온다
  - `isAnnotationPresent` 메서드는 둘을 정확하게 구분한다
    - 반복 가능 애너테이션을 여러번 달고, `isAnnotationPresent`로 반복 가능 애너테이션이 달렸는지 검사한다면 '그렇지 않다'가 나온다(컨테이너가 달려있기 때문)
    - 같은 이유로 `isAnnotationPresent`로 컨테이너 애너테이션이 달렸는지 검사한다면 반복 가능 애너테이션을 한번만 단 메서드를 무시하고 지나치게 된다
  - 따라서 랄려있는 수와 관계없이 모두 검사하려면 둘을 따로따로 확인해야 한다

----

### [item40] @Override 애너테이션을  일관되게 사용하라

- 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자
  - 예외: 구체 클래스에서 상위 클래스의 추상 메서드를 재정의하는 경우

- IDE는 재정의할 메서드를 선택하면 @Override를 자동으로 붙인다
- IDE는 @Override를 일관되게 사용하도록 한다
  - IDE에서 관련 설정을 활성화해놓으면 @Override가 달려있지 않은 메서드가 실제로는 재정의를 했다면 경고를 준다
- @Override는 클래스 뿐만 아니라 인터페이스의 메서드를 재정의할때도 사용할 수 있다
  - 디폴트 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에도 @Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신할 수 있다.
  - 구현하려는 인터페이스에 디폴트 메서드가 없음을 안다면, 이를 구현한 메서드에서는 @Override를 생략해 코드를 좀더 깔끔하게 유지해도 좋다
- 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 다는 것이 좋다
  - 상위 클래스가 구체 클래스/추상 클래스일때 모두
  - 예시: Set 인터페이스
    - Collection 인터페이스를 확장했지만 새로 추가한 메서드는 없다
    - 따라서 모든 메서드 선언에 @Override 를 달아 실수로 추가한 메서드가 없음을 보장했다

#### @Override

- 메서드 선언에만 달 수 있으며, 상위 타입의 메서드를 재정의했음을 뜻한다
- 이 애너테이션을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방할 수 있다.

#### 예시) 바이그램

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second){
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b){
        return b.first == first && b.second == second;
    }
    
    public int hashCode(){
        return 31*first+second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for(int i=0;i<10;i++){
            // 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복
            // 집합에 추가한 후, 그 집합의 크기를 출력
            for(char ch='a';ch<='z';ch++)
                s.add(new Bigram(ch,ch));
            // Set은 중복을 허용하지 않아 26이 출력될 것으로 예상되나, 실제로는 260이 출력된다
            System.out.println(s.size());
        }
    }
}
```

- `equals`, `hashCode` 메서드를 재정의하려 한 것으로 보인다
  - 하지만 위와 같은 코드는 '재정의`overriding`' 가 아닌 '다중정의`overloading`' 이다!
    - Object.equals의 재정의시 매개변수 타입을 Object로 해야 하는데, Bigram으로 넘어오고 있다
    - Object의 equals는 ==연산자와 똑같이 객체 식별성(identity)만을 확인한다
    - 따라서 같은 소문자를 소유한 바이그램 10개가 각각 다른 객체로 인식되어 260을 출력한다
- @Override 애너테이션을 단다면 컴파일 오류가 발생한다

```java
@Override
public boolean equals(Bigram b){
        return b.first == first && b.second == second;
}
// java: method does not override or implement a method from a supertype
```

- 잘못한 부분을 명확하게 알려주므로, 곧장 올바르게 수정할 수 있다!

```java
@Override
public boolean equals(Object o){
    if(!(o instanceof Bigram))
        return false;
    Bigram b= (Bigram) o;
    return b.first == first && b.second == second;
}
```

-----

### [item41] 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

새로 추가하는 메서드 없이, 단지 타입 정의가 목적이라면 마커 인터페이스를 선택한다.

#### 마커 인터페이스

- 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스
- 예시: Serializable 인터페이스
  - 자신을 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 쓸 수 있다(직렬화할 수 있다)고 알려줌

#### 마커 인터페이스가 마커 애너테이션보다 나은 점

- 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있다
  - 마커 애너테이션은 그렇지 않다!
  - 마커 인터페이스는 타입이기 때문에, 마커 애너테이션을 사용했다면 런타임에야 발견될 오류를 컴파일타임에 잡을 수 있다
  - 예시: 자바 직렬화
    - Serializable 마커 인터페이스를 보고 그 대상이 직렬화할 수 있는 타입인지 확인
    - 예를 들어 ObjectOutputStream.writeObject메서드는 당연히 인수로 받은 객체가 Serializable을 구현했을 것이라 가정한다
    - 하지만 이 메서드는 Serializable이 아닌 Object를 받도록 설계되었다
    - 직렬화할 수 없는 객체를 넘겨도 런타임에 문제를 확인할 수 있다!
- 적용 대상을 더욱 정밀하게 지정할 수 있다
  - 적용 대상(@Target)을 ElementType.TYPE으로 선언한 애너테이션은 모든 타입(클래스,인터페이스,열거타입,애너테이션)에 달 수 있다.
    - 부착할 수 있는 타입을 더 세밀하게 지정할 수 없다
  - 특정 인터페이스를 구현한 클래스에만 적용하고 싶은 마커가 있다면?
    - 이 마커를 인터페이스로 정의했다면, 그냥 마킹하고 싶은 클래스에서만 그 인터페이스를 구현하면 된다
    - 마킹된 타입은 자동으로 그 인터페이스의 하위 타입임이 보장된다
  - 예시: Set 인터페이스 (일종의 제약이 있는 마커 인터페이스)
    - Collection의 하위 타입에만 적용 가능
    - Collection이 정의한 메서드 외에는 새로 추가한 것이 없다
    - 보통은 Set을 마커 인터페이스로 생각하지는 않는다
      - add, equals, hashCode 등 Collection의 메서드 몇 개의 규약을 살짝 수정했음
  - 특정 인터페이스의 하위 타입에만 적용 가능하며, 아무 규약에도 손대지 않은 마커 인터페이스는 존재 가능하다
    - 객체의 특정 부분을 불변식(invariant)으로 규정
    - 그 타입의 인스턴스는 다른 클래스의 특정 메서드가 처리할 수 있다는 사실을 명시하는 용도

#### 마커 애너테이션이 마커 인터페이스보다 나은 점

- 애너테이션 시스템의 지원을 받는다
  - 애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 쓰는 것이 일관성을 지키는 데 유리하다

#### 적절한 쓰임새

- 클래스와 인터페이스 외의 프로그램 요소(모듈,패키지,필드,지역변수 등) 에 마킹해야 할 때는 애너테이션 사용
  - 클래스와 인터페이스만이 인터페이스를 구현하거나 확장할 수 있음
- '이 마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있는가?'
  - O -> 마커 인터페이스
    - 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용, 컴파일 타임에 오류를 잡아낼 수 있음
  - 절대 X 또는 애너테이션을 적극 활용하는 프레임워크 사용중?-> 마커 애너테이션