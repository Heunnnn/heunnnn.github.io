---
layout: post
title: 201220 Effective Java 4장 정리 (1)
tags:
  - Effective Java
---

## 4. 클래스와 인터페이스

### [item15] 클래스와 멤버의 접근 권한을 최소화하라

#### 정보 은닉/캡슐화

잘 설계된 컴포넌트는 모든 내부구현을 완벽히 숨겨, 구현과 API를 분리한다.

오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다

#### 정보 은닉/캡슐화의 장점

- 여러 컴포넌트를 병렬로 개발 가능, 시스템 개발 속도를 높인다
- 시스템 관리 비용을 낮춘다
- 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있어 성능 최적화에 도움을 준다
- 소프트웨어 재사용성을 높인다
- 큰 시스템을 제작하는 난이도를 낮춘다

#### 멤버에 부여할 수 있는 접근 수준

- private : 멤버를 선언한 톱레벨 클래스에서만 접근

- package-private : 멤버가 소속된 패키지 안의 모든 클래스, 접근 제한자를 명시하지 않았을 때 적용됨

- protected : package-private 접근범위 포함, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근가능

- pubilc : 모든 곳에서 접근 가능

  클래스의 공개 API를 설계하고, 그 외의 모든 멤버는 private로 만든다.

#### 멤버 접근성을 좁히지 못하는 제약

상위 클래스의 메서드를 재정의할때는 그 접근수준을 상위클래스에서보다 좁게 설정할 수 없다

리스코프 치환 원칙(상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다)을 지키기 위함

테스트만을 위해 클래스,인터페이스,멤버를 공개API로 만들어서는 안 된다

#### public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다

public 가변 필드를 가진 클래스는 스레드 안전하지 않다.

하지만 해당 클래스가 표현하는 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 상수라면 public static final 필드로 공개해도 좋다

- 대문자 알파벳으로 쓰며, 단어 사이에 밑줄(_)을 넣는다

- 이런 필드는 반드시 기본 타입 값이나 불변 객체를 참조해야 한다

- public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다

  - 길이가 0이 아닌 배열은 모두 변경 가능하기 때문

  - 배열을 private으로 만들고 public 불변 리스트를 추가

    - ```java
      private static final Thing[] PRIVATE_VALUES={...};
      public static final List<Thing> VALUES = 
          Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
      ```

  - 배열을 private으로 만들고 그 복사본을 반환하는 public 메서드 추가

    - ```java
      private static final Thing[] PRIVATE_VALUES={...};
      public static final Thing[] values(){  
          return PRIVATE_VALUES.clone();
      }
      ```

-----

### [item16] public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

인스턴스 필드들을 모아놓은 클래스

- 데이터 필드에 직접 접근할 수 있어 캡슐화의 이점을 제공하지 못한다

- 예시) java.awt.package 패키지의 Point, Dimension 클래스

```java
class Point {
    public double x;
    public double y; 
}
```

접근자와 변경자(mutator) 메서드(getter/setter)를 활용해 데이터를 캡슐화한다

- public 클래스라면 이와 같은 방식이 옳다.

```java
class Point {
    public double x;
    public double y; 
    
    public Point(double x, double y){
        this.x=x;
        this.y=y;
    }
    
    public double getX(){ return x;}
    public double getY(){ return y;}
    
    public void setX(double x){ this.x=x;}
    public void setY(double y){ this.y=y;}
}
```

하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 문제가 없다

- 이 클래스가 표현하려는 추상 개념만 올바로 표현하면 된다
- 종종 불변이든 가변이든 필드를 노출하는 편이 나을 때도 있다

public 클래스의 필드가 불변이고 노출되어 있다면?

- 불변식을 보장할 수 있다

- API를 변경하지 않고는 표현 방식을 바꿀 수 없다
- 필드를 읽을 때 부수작업을 수행할 수 없다

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    public final int hour;
    public final int minute;
    
    public Time(int hour, int minute){
        if(hour<0||hour>=HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: "+hour);
        if(minute<0||minute>=MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: "+minute);
        this.hour=hour;
        this.minute=minute;
    }
    ...
}
```

-----

### [item17] 변경 가능성을 최소화하라

#### 불변 클래스

인스턴스의 내부 값을 수정할 수 없는 클래스

불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다

예시: String, 기본 타입의 박싱클래스, BigInteger, BigDecimal

#### 불변 클래스를 만드는 법

- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다
  - getter가 있다고 해서 무조건 setter를 제공하지 말자
- 클래스를 확장할 수 없도록 한다
- 모든 필드를 final로 선언한다
  - 시스템이 강제하는 수단을 이용, 설계자의 의도를 명확히 드러내는 방법
- 모든 필드를 private로 선언한다
  - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근, 수정하는 것을 막아준다
  - public final로만 선언해도 불변 객체가 되나, 이렇게 하면 다음 릴리즈에서 내부 표현을 바꾸지 못한다
  - 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분은 최소한으로 줄어야 한다.
  - 다른 합당한 이유가 없다면 모든 필드는 private final 이어야 한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다
  - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다.
  - 생성자,접근자,readObject 메서드 모두에서 방어적 복사를 수행하라
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다
  - 확실한 이유가 없다면 생성자와 정적 팩터리 외 그 어떤 초기화 메서드도 public으로 제공하지 않는다.

#### 예시: 복소수를 표현하는 클래스

*예시일 뿐, 반올림 처리와 NaN, 무한대를 다루지 못하므로 실무에서 사용은 X

```java
public final class Complex{
    private final double re;
    private final double im;
    
    public Complex(double re, double im){
        this.re = re;
        this.im = im;
    }
    
    public double realPart(){ return re; }
    public double imaginaryPart(){ return im;}
    
    public Complex plus(Complex c){
        return new Complex(re+c.re,im+c.re);
    }
    
    public Complex minus(Complex c){
        return new Complex(re-c.re,im-c.im);
    }
    
    public Complex times(Complex c){
        return new Complex(re*c.re-im*c.im,
                          re*c.im+im*c.re)
    }
    
    public Complex dividedBy(Complex c){
        double tmp=c.re * c.re + c.im*c.im;
        return new Complex((re*c.re+im*c.im)/tmp),
        					(im*c.re-re*c.im)/tmp);
    }
    
    @Override public boolean equals(Object o){
        if(o==this) return true;
        if(!(o instanceof Complex)) return false;
        
        Complex c=(Complex) o;
        
        return Double.compare(c.re,re)==0
            && Double.compare(c.im,im)==0;
    }
    
    @Override public int hashCode(){
        return 31* Double.hashCode(re) + Double.hashCode(im);
    }
    
    @Override public String toString(){
        return "("+re+"+"+im+"i)");
    }
}
```
- 실수부,허수부 값 반환 접근자 메서드와 사칙연산 메서드
  - 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환한다
  - 함수형 프로그래밍: 피연산자에 함수를 적용해 그 결과를 반환하나 피연산자 자체는 그대로인 패턴
    - 메서드 이름으로 동사(add) 대신 전치사(plus) 사용
      - 해당 메서드가 객체의 값을 변경하지 않는다는 사실 강조
    - 이 방식으로 프로그래밍하면 코드에서 불변이 되는 영역의 비율이 높아진다
  
- 불변 객체는 근본적으로 스레드 안전하며, 따로 동기화가 필요 없다

  - 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다

  - 가장 쉬운 재활용 방법: 자주 쓰이는 값들을 상수로 제공한다

    - ```java
      public static final Complex ZERO = New Complex(0,0);
      public static final Complex ONE = New Complex(1,0);
      public static final Complex I = New Complex(0,1);
      ```

  - 자주 사용되는 인스턴스를 캐싱, 같은 인스턴스를 중복 생성하지 않게 해주는 적정한 팩터리를 제공한다

    - 예시: 박싱된 기본 타입 클래스 전부와 BigInteger
    - 여러 클라이언트가 인스턴스를 공유하므로 메모리 사용량과 가비지 컬렉션 비용이 줄어든다

- 불변 객체를 공유할 수 있으므로, 방어적 복사도 필요하지 않다

  - clone 메서드나 복사 생성자를 제공하지 말자
  - String 클래스의 복사 생성자는 되도록 사용하지 않아야 한다

- 불변 객체는 자유롭게 공유할 수 있으며, 불변 객체끼리는 내부 데이터를 공유할 수 있다.

#### 객체를 만들 때 다른 불변 객체들을 구성요소로 사용

- 그 구조가 아무리 복잡해지더라도 불변식을 유지하기 쉬워진다
  - 맵의 키나 셋의 원소로 쓰기 쉽다.
- 불변 객체는 그 자체로 실패 원자성을 제공한다
  - 상태가 절대 변하지 않으므로, 잠깐이라도 불일치 상태에 빠질 가능성이 없다
- 값이 다르면 반드시 독립된 객체로 만들어야 한다는 단점 존재
  - 값의 가지수가 많다면 이들을 모두 만드는 데 큰 비용 소모
    - 예시: BigInteger는 불변이므로 flipBit 메서드는 새로운 BigInteger 인스턴스를 생성한다
      - BitSet은 가변 클래스이므로, flip 메서드는 원하는 비트 하나만 상수 시간 안에 변경한다
- 원하는 객체를 완성하기까지 단계가 많고, 그 중간단계에서 만들어진 객체가 모두 버려진다면 성능 문제가 커진다
  - 1.다단계 연산을 예측하여 기본 기능으로 제공한다
  - 2.예측 불가능하다면 public으로 제공한다
    - 예시: String 클래스와 가변 동반 클래스 StringBuilder 

#### 자신을 상속하지 못하게 하는 방법: 모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩터리를 제공한다

```java
public class Complex{
    private final double re;
    private final double im;
    
    private Complex(double re, double im){
        this.re = re;
        this.im = im;
    }
    
    public static Complex valueOf(double re, double im){
        return new Complex(re,im);
    }
    ....
}
```

패키지 바깥의 클라이언트에서 바라본 이 불변 객체는 사실상 final 이다.

#### BigInteger와 BigDecimal

- 설계 당시 불변 객체가 사실상 final이라는 생각이 널리 퍼지지 않았다

- 이 두 클래스의 메서드들은 모두 재정의 가능하도록 설계되었고, 하위 호환성 문제로 아직 문제를 고치지 못했다

- BigInteger 혹은 BigDecimal의 인스턴스를 인수로 받는 경우, 이 값들이 불변이어야 클래스의 보안을 지킬 수 있다면 이 인수들은 가변이라 가정하고 방어적으로 복사하여 사용해야 한다

  - ```java
    public static BigInteger safeInstance(BigInteger val){
        return val.getClass() == BigInteger.class?
            val:new BigInteger(val.toByteArray());
    }
    ```

-----

### [item18] 상속보다는 컴포지션을 사용하라

메서드 호출과 달리 상속은 캡슐화를 깨뜨린다

- 상위 클래스가 어떻게 구현되느냐에 따라 하위클래스의 동작에 이상이 생길 수 있다.
- 상위 클래스는 릴리즈마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할수 있다.

#### 잘못된 예시: 상속을 잘못 사용하고 있음

상속을 염두에 두지 않고 설계했으며, 상속할 때의 주의점도 문서화해놓지 않은 외부 클래스를 상속할 때의 위험성

```java
public class InstrumentedHashSet<E> extends HashSet<E>{
    private int addCount = 0;
    public InstrumentedHashSet(){}
    public InstrumentedHashSet(int initCap, float loadFactor){
        super(initCap, loadFactor);
    }
    @Override public boolean add(E e){
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c){
        addCount+=c.size();
        return super.addAll(c);
    }
    public int getAddCount(){
        return addCount;
    }
}
```

자바 9부터 지원하는 List.of로 리스트 생성, 

```java
InstrumentedHashSet<String> s=  new InstrumentedHashSet<>();
s.addAll(List.of("틱","탁탁","펑"));
```

getAddCount를 호출하면? -> 예상값:3, 실제 반환값: 6

- HashSet의 addAll 메서드가 add 메서드를 사용해 구현되었기 때문
- InstrumentedHashSet의 addAll이 addCount에 3을 더한 후, HashSet의 addAll 구현을 호출한다
- HashSet의 addAll은 각 원소를 add메서드(InstrumentedHashSet에서 재정의한 메서드!)를 호출해 추가한다
- 따라서 addCount에 값이 중복해서 더해지므로, 최종값이 6으로 늘어난다.

#### 문제 해결을 위한 방법들...

- 하위 클래스에서 addAll 메서드를 재정의하지 않는다
  - 하지만 HashSet의 addAll이 add 메서드를 이용해 구현했음을 가정한 해법이다
  - 자기사용(self-use)여부는 해당 클래스의 내부 구현 방식에 해당하므로 다음 릴리스에서 유지될지는 알 수 없다.
- addAll 메서드를 다른 식으로 재정의한다
  - 주어진 컬렉션을 순회하며 원소 하나당 add메서드를 한번만 호출한다
  - 어렵고,시간도 더 들며, 오류를 내거나 성능을 떨어뜨릴 수 있다.
  - 하위 클래스에서는 접근할 수 없는 private 필드를 써야 하는 상황이라면 이 방식으로는 구현 자체가 불가능하다

다음 릴리스에서 상위 클래스에 새로운 메서드를 추가하는 경우

- 예시: 보안 떄문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야 하는 프로그램
  - 그 컬렉션을 상속하여 원소를 추가하는 모든 메서드를 재정의해 필요한 조건을 먼저 검사하도록
    - 상위 클래스에 또 다른 원소 추가 메서드가 만들어지기 전까지만 가능
    - 하위 클래스에서 재정의하지 못한 새로운 메서드를 사용해 허용되지 않은 원소를 추가할 수 있게 된다

#### 메서드 재정의가 원인이므로, 클래스를 확장하더라도 매서드를 재정의하지 않고 새로운 메서드를 추가한다

- 다음 릴리즈에서 상위 클래스에 새 메서드가 추가되었으나, 하필 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입은 다르다면 컴파일조차 되지 않는다.
  - 반환 타입마저 같다면 상위 클래스의 새 메서드를 재정의한 모습이 된다
  - 상위 클래스의 메서드 규약도 만족하지 못할 가능성이 크다

#### 컴포지션 구성

- 기존 클래스 대신 새로운 클래스를 만들고, private 필드로 기존 클래스 인스턴스를 참조하게 만든다
- 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다
  - '전달(forwarding)' 방식
- 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

#### 예시: 컴포지션 방식과 전달 방식으로 구현된 InstrumentedHashSet

```java
// 다른 Set 인스턴스를 감싸고 있다: 래퍼 클래스
// 다른 Set에 계측 기능을 덧씌운다: 데코레이터 패턴
// 컴포지션과 전달의 조합은 넓은 의미로 위임(delegation)이라고 부른다
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount=0;
    
    public InstrumentedHashSet(Set<E> s){
        super(s);
    }
    
    @Override public boolean add(E e){
        addCount++;
        return super.add(e);
    }
    
    @Override public boolean addAll(Collection<? extends E> c){
        addCount+=c.size();
        return super.addAll(c);
    }
    public int getAddCount(){
        return addCount;
    }
}

public class ForwardingSet<E> implements Set<E>{
    private final Set<E> s;
    public ForwardingSet(Set<E> s){ this.s = s; }
    
    public void clear() { s.clear(); }
    public boolean contains(Object o) { return s.contains(o);}
    public boolean isEmpty() { return s.isEmpty();}
    public int size() { return s.size();}
    public Iterator<E> iterator() { return s.iterator();}
    public boolean add(E e) {return s.add(e);}
    public boolean remove(Object o) {return s.remove(o);}
    public boolean containsAll(Collection<?> c){ return s.containsAll(c);}
    public boolean addAll(Collection<? extends E> c){
        return s.addAll(c);
    }
    public boolean remobeAll(Collection<?> c){
        return s.removeAll(c);
    }
    public boolean retainAll(Collection<?> c){
        return s.retainAll(c);
    }
    public Object[] toArray() { return s.toArray();}
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    @Override public boolean equals(Object o){ return s.equals(o);}
    @Override public int hashCode() { return s.hashCode();}
    @Override public String toString(){ return s.toString();}
}
```

- 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만든다

- 컴포지션 방식: 한번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다

  - ```java
    Set<Instant> times= new InstrumentedSet<>(new TreeSet<>(cmp));
    Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
    ```

  - 대상 Set 인스턴스를 특정 조건하에서만 임시로 계측하기도 가능하다

    - ```java
      static void walk(Set<Dog> dogs){
          InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
      }
      ```

#### 래퍼 클래스

- 단점이 거의 없다
- 래퍼 클래스는 콜백 프레임워크와는 어울리지 않는다
  - SELF 문제
    - 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백)때 사용하도록 한다
    - 내부 객체는 래퍼의 존재를 모르므로 자기 자신의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다
    - 재사용할 수 있는 전달 클래스를 인터페이스당 하나씩 만들어둔다.

#### 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 사용되어야 한다

- 클래스 B와 A가 서로 is-a 관계일 떄만 클래스 A를 상속해야 한다
- 이 원칙을 위반하는 예시: Stack은 Vector를 확장하고 있으며, Properties는 HashTable을 확장하여 구현중이다
- 확장하려는 클래스의 API에 아무런 결함이 없는가?
  - 컴포지션은 이 결함을 숨기는 새 API를 설계할 수 있으나, 상속은 상위 클래스의 API를 그 결함까지도 그대로 승계한다

#### 컴포지션을 써야 할 상황에서 상속을 사용하는 것은 내부 구현을 불필요하게 노출한다

- API가 내부 구현에 묶이게 되며, 클래스의 성능도 제한된다.
- 클라이언트의 노출된 내부에 직접 접근 가능해진다

-----

### [item19] 상속을 고려해 설계하고 문서화하라. 그렇지 않다면 상속을 금지하라

#### 상속을 고려한 설계와 문서화

- 메서드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서로 남겨야 한다

  - 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다
  - 예시: java.util.AbstractCollection의 remove()

- 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다

  - 예시: java.util.AbstractCollection의 removeRange()

- 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증한다

- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다

  - ```java
    public class Super {
        public Super(){
            // 생성자가 재정의 가능 메서드 호출중!
            overrideMe();
        }
        public void overrideMe(){
            
        }
    }
    
    public final class Sub extends Super {
        // 초기화되지 않은 final 필드: 생성자에서 초기화한다
        public final Instant instant;
        
        Sub(){
            instant= Instant.now();
        }
        
        @Override public void overrideMe(){
        	// 재정의 가능 메서드. 상위 클래스 생성자가 호출한다
            System.out.println(instant);
        }
        
        public static void main(String[] args){
            Sub sub = new sub();
            sub.overrideMe();
        }
    }
    ```

    - 이 프로그램은 instant를 두번 출력할 것으로 기대되지만 첫 번째는 null을 출력한다
    - 상위 클래스의 생성자가 하위 클래스 생성자가 인스턴스 필드를 초기화하기 전 overrideMe 호출
    - 이 프로그램에서는 final 필드의 상태가 두 가지가 된다(정상이라면 하나)
    - overrideMe에서 instant 객체의 메서드를 호출하려 한다면, 상위 클래스의 생성자가 overrideMe를 호출할 때 NullPointerException을 던지게 된다

- clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다

  - readObject: 하위 클래스의 상태가 미처 다 역직렬화되기 전에 재정의한 메서드부터 호출하게 된다
  - clone: 하위 클래스 clone 메서드가 복제본의 상태를 수정하기 전 재정의한 메서드 호출
    - clone은 잘못되면 복제본뿐만 아니라 원본 객체에도 피해를 줄 수 있다

- 클래스를 상속용으로 설계하려면 엄청난 노력이 들고, 그 클래스에 안기는 제약도 상당하다

- 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이다.

  - 클래스를 final로 선언한다
  - 모든 생성자를 private, package-private으로 선언하고 public 정적 팩터리를 만들어주는 방법이다.

-----

### [item20] 추상 클래스보다는 인터페이스를 우선하라

```
일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다.
복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현 방법을 함께 제공하는 방법을 꼭 고려해 본다.
골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다.
인터페이스의 구현상 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하다.
```

기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다

- 인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 implements 구문을 추가한다
- 예시 : Comparable, Iterable, AutoCloseable 인터페이스 가 새로 추가됐을 때 표준 라이브러리의 수많은 기존클래스가 이 인터페이스들을 구현했다.

기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어렵다

#### 믹스인(mixin) 정의

- 클래스가 구현할 수 있는 타입

- 믹스인을 구현한 클래스에 원래의 '주된 타입'외에도 특정 선택적 행위를 제공(혼합)한다고 선언하는 효과를 준다

- 예시:Comparable

  - 자신을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언하는 믹스인 인터페이스이다

- 추상 클래스로는 믹스인을 정의할 수 없다

  - 기존 클래스에 덧씌울 수 없다

- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다

  - 현실에는 계층을 엄격하게 구분하기 어려운 개념도 있다

  - ```java
    public interface Singer {
        AudioClip sing(Song s);
    }
    public interface SongWriter {
        Song compose(int chartPosition);
    }
    
    public interface SingerSongWriter extends Singer, SongWriter{
        AudioClip strum();
        void actSensitive();
    }
    ```

  - 이와 같은 구조를 클래스로 만들려면 가능한 조합 전부를 각각의 클래스로 정의해야 한다

    - 조합 폭발(combinatorial explosion): 속성이 n개라면 지원해야 할 조합의 수는 2^n개가 된다

- 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다

#### 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공

- 템플릿 메서드 패턴

  - 인터페이스로는 타입을 정의, 필요하면 디폴트 메서드 몇 개도 함께 제공
  - 골격 구현 클래스는 나머지 메서드까지 구현한다
  - 골격 구현을 확장하는 것만으로도 이 인터페이스를 구현하는 데 필요한 일이 완료된다

- Interface 인터페이스-> 그 골격 구현 클래스의 이름은 AbstractInterface로 짓는다.

  - 예시: AbstractCollection, AbstractSet, AbstractMap

- ```java
  static List<Instger> intArrayList(int[] a){
      Objects.requireNonNull(a);
      
      // 자바 9 이후부터 가능!
      return new AbstractList<>(){
          @Override public Integer get(int i){
              return a[i]; // 오토박싱
          }
          @Override public Integer set(int i, Integer val){
              int oldVal = a[i];
              a[i]=val; // 오토언박싱
              return oldval; // 오토박싱
          }
          @Override public int size(){
              return a.length();
          }
      }
  }
  ```

  - int 배열을 받아 Integer 인스턴스의 리스트 형태로 보여주는 어댑터 이기도 하다.
  - 이 구현에서는 익명 클래스 형태를 사용했다

- 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다

  - 골격 구현을 확장하는 것으로 인터페이스 구현이 거의 끝난다!
  - 구조상 골격 구현을 확장하지 못한다면 인터페이스를 직접 구현해야 한다
    - 디폴트 메서드의 이점을 여전히 누릴 수 있다

- 시뮬레이트한 다중 상속

  - 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의
  - 각 메서드 호출을 내부 클래스의 인스턴스에 전달
  - 래퍼 클래스와 유사한 방식

- 골격 구현 작성

  - 인터페이스에서 다른 메서드들의 구현에 사용되는 기반 메서드 선정(추상 메서드)
  - 기반 메서드를 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공
    - equals, hashCode 등의 Object 메서드는 디폴트 메서드로 제공하면 안된다
  - 만약 인터페이스의 메서드 모두가 기반 메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 별도로 만들 필요는 없다
  - 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성해 넣는다

- 단순 구현: 골격 구현의 작은 변종

  - 예시: AbstractMap.SimpleEntry