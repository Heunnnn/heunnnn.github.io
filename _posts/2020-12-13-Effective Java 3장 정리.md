---
layout: post
title: 201213 Effective Java 3장 정리
tags:
  - Effective Java
---

## 3. 모든 객체의 공통 메서드

Object에서 final이 아닌 메서드들(equals, hashCode, toString, clone, finalize)은 모두 재정의(overriding)를 염두에 두고 설계된 것 

-> 재정의시 지켜야 하는 일반 규약이 명확하게 정의되어 있다.

### [item 10] equals는 일반 규약을 지켜 재정의하라

```
꼭 필요한 경우가 아니면 equals를 재정의하지 말라
재정의해야 하는 경우 규약을 확실히 지켜 비교해야 한다
```

#### equals를 재정의하지 않아야 하는 상황들

##### 1. 각 인스턴스가 본질적으로 고유함

- 값이 아니라 동작하는 개체를 표현하는 클래스
- 예시: Thread

##### 2. 인스턴스의 '논리적 동치(logical equality)'를 검사할 일이 없음

- 예시: java.util.regex.Pattern
  - 두 Pattern의 인스턴스가 같은 정규표현식을 나타는지를 검사하는 법?
  - 설계자가 위 방법이 필요하지 않다고 여기는 경우 Object의 기본 equals로 해결

##### 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞을때

- 예시: AbstractSet과 대부분의 Set 구현체, AbstractList와 List 구현체, AbstractMap와 Map 구현체 

##### 4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없을때

- 다음과 같이 정의하면 equals 호출을 막을 수 있다
  
- ```java
  @Override pubilc boolean equals(Object o){
      throw new AssertionError();
  }
  ```

#### equals 재정의가 필요한 경우

##### -> 논리적 동치성 확인이 필요하지만 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때

- 값 클래스(Integer, String)
- 논리적 동치성을 확인하도록 재정의하면, 값을 비교할 수 있고 Map의 키와 Set의 원소로도 사용할 수 있다.
- 값 클래스라고 해도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals의 재정의가 필요하지 않다
  - 예시: Enum
  - 논리적 동치성과 객체 식별성이 같은 의미가 된다(논리적 의미가 같은 인스턴스가 두 개 이상 만들어지지 않음)

#### Object 명제에 적힌 equals 메서드 재정의 규약

한 클래스의 인스턴스는 다른곳으로 빈번히 전달되며, 컬렉션 클래스를 포함한 수많은 클래스는 전달받은 객체가 equals 규약을 지킨다고 가정하고 동작한다.

##### 1. 반사성(reflexivity): null이 아닌 모든 참조값 x에 대해 x.equals(x)는 true

- 객체는 자기 자신과 같아야 한다(일부러 어기는 경우가 아니면 만족시키지 못하기가 더 어려워보인다....?)
- 이 요건을 어긴 클래스는 인스턴스를 컬렉션에 넣은 다음 contains 메서드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것이다.

##### 2. 대칭성(symmetry): nul이 아닌 모든 참조값 x,y에 대해 x.equals(y)가 true면 y.equals(x)도 true이다

- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다

##### 3. 추이성(transitivity): null이 아닌 모든 참조값 x,y,z에 대해 x.equals(y)가 true이고 y.equals(z)가 true이면 x.equls(z)도 true이다

- 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황
- 구체 클래스를 확장해 새값을 추가하면서 equals 규약을 만족시킬 방법은 없다
- 상속 대신 컴포지션을 사용하라
  - 상속 대신 private 필드로 두고, 뷰 메서드를 public으로 추가한다.
- 예시: java.sql.Timestamp
  - java.util.Date를 확장한 후 nanoseconds 필드를 추가
  - Timestamp의 equals는 대칭성을 위배하며, Date객체와 함께 섞어 사용하면 엉뚱하게 동작할 수 있다
- 추상 클래스의 하위 클래스에서라면 equals 규약을 지키면서도 값을 추가할 수 있다

##### 4. 일관성(consisitency): null이 아닌 모든 참조값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다

- 가변 객체는 비교 시점에 따라 서로 다를 수도, 같을 수도 있지만 불변 객체는 한번 다르면 끝까지 달라야 한다.
- equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.
  - 예시: java.net.URL의 equals
    - 주어진 URL - 매핑된 호스트의 IP주소를 이용해 비교
    - 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없다.
    - URL의 equals가 일반 규약을 어기게 되어 종종 문제가 발생한다
  - equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다

##### 5. null-아님: null이 아닌 모든 참조값 x에 대해, x.equals(null)은 false다

- 모든 객체가 null과 같지 않아야 한다

- ```java
  // 묵시적 null 검사
  @Override public boolean equals(Object o){
      if(!(o instanceof MyType)) return false;
      Mytype mt = (MyType) o;
      ...
  }
  ```

  - 타입을 확인하지 않으면 잘못된 타입이 인수로 주어졌을때 ClassCastException 발생
  - instanceof 는 첫번째 피연산자가 null이면 false를 반환한다.
  - 따라서 입력이 null이면 타입 확인 단계에서 false를 반환하기 때문에 null검사를 명시적으로 하지 않아도 된다

#### equals 메서드 구현 방법

##### 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다

##### 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다

- 올바른 타입: equals가 정의된 클래스 + 그 클래스가 구현한 특정 인터페이스(Set,List,Map,Map.Entry)

##### 3. 입력을 올바른 타입으로 형변환한다

##### 4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다

- 2단계에서 인터페이스를 사용했다면 입력의 필드값을 가져올 때도 그 인터페이스의 메서드를 사용해야 한다
- 기본 타입 필드는 == 연산자로 비교
- float, double 필드는 Float.compare(float, float) 와 Double.compare(double,double)로 비교한다
  - Float.NaN, -0.0f, 특수한 부동소수값 등을 다루기 위함
  - Float.equals와 Double.equals 사용
    - 오토박싱을 수반할 수 있어 성능상 나쁨
- null도 정상값으로 취급하는 참조 타입 필드
  - Object.equals(object,object)로 비교, NullPointerException 방지
- 비교하기 아주 복잡한 필드를 가진 클래스
  - 필드의 표준형(canonical form) 저장, 표준형끼리 비교
  - 불변 클래스인 경우 경제적
- 다를 가능성이 더 크거나, 비교하는 비용이 싼 필드를 먼저 비교한다
- 동기화용 락(lock) 필드와 같은 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다
- 파생 필드를 비교하는 것이 더 빠른 경우
  - 예시 : Polygon 클래스(자신의 영역 캐싱) -> 모든 변과 정점을 비교할 필요 없이 캐시해둔 영역만 비교하면 결과를 알 수 있다.

##### 5. 대칭적인가? 추이성이 있는가? 일관적인가?

- 단위 테스트를 작성하여 돌려본다

##### 전형적인 equals 메서드 예시

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;
    
    public PhoneNumber(int areaCode,int prefix, int lineNum){
        this.areaCode=rangeCheck(areaCode,999, "지역코드");
        this.prefix=rangeCheck(prefix, 999, "프리픽스");
        this.lineNum=rangeCheck(lineNum, 9999, "가입자 번호");
    }
    
    private static short rangeCheck(int val,int max,String arg){
        if(val<0 || val>max)
            throw new IllegalArgumentException(arg+": "+val);
        return (short) val;
    }
    
    @Override public boolean equals(Object o){
        if(o==this) return true;
        if(!(o instanceof PhoneNumber)) return false;
        PhoneNumber pn=(PhoneNumber) o;
        return pn.lineNum===lineNum && pn.prefix == prefix && pn.areaCode==areaCode;
    }
}
```

#### equals 재정의시 주의사항

##### 1. equals를 재정의할때는 hashCode도 반드시 재정의한다

##### 2. 너무 복잡해게 해결하려 들지 말자

- 필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다
- 예시: File 클래스 - 심볼릭 링크를 비교해 같은 파일 확인 금지

##### 3. Object 외의 타입을 매개변수로 받는 equals 메서드를 선언하지 말자

```java
// 잘못된 예 -  매개변수는 반드시 Object 여야 한다
public boolean equals(MyClass o){
    ...
}
```

- 이 메서드는 입력타입이 Object가 아니므로, 재정의(Overriding)가 아닌 다중 정의(Overloading)이다

```java
// 잘못된 예 - 컴파일 오류!
@Override public boolean equals(MyClass o){
    ...
}
```

equals 작성과 테스트 : AutoValue 프레임워크 이용

lombok으로도 값 클래스 생성 가능 -> @EqualsAndHashCode() 어노테이션

-----

### [item11] equals 재정의시 hashCode도 재정의하라 

#### Object 명세 내부 규약

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 일관되게 항상 같은 값을 반환해야 한다
- equals(Object)가 두 객체를 같다고 판단했다면 두 객체의 hashCode는 똑같은 값을 반환해야 한다
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

#### 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다

- HashMap에 값을 넣고, 논리적 동치인 인스턴스로 get메서드를 사용하는 경우
  - hashCode 클래스를 재정의하지 않았다면, 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하게 된다

##### 최악의 hashCode 구현

```java
@Override public int hashCode(){ return 42; }
```

- 모든 객체에게 똑같은 값만 내어준다
- 해시테이블 버킷이 연결리스트처럼 동작하게 되면서 O(n)의 수행시간이 된다
- 이상적인 hashCode 구현은 주어진 서로 다른 인스턴스들을 32비트 정수 범위에 균일하게 분배한다.

#### hashCode 구현 요령

##### 1. int 변수 result를 선언, 값 c로 초기화한다(c: equals 비교에 사용되는 필드(핵심필드) 중 첫번째 필드 계산값)

##### 2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다

- 해당 필드의 해시코드 c를 계산
  - 기본타입 필드라면, Type.hashCode(f)를 수행(박싱 클래스의 hashCode 메서드)
  - 참조타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals 메서드를 재귀적으로 호출한다면 이 필드의 hashCode를 재귀적으로 호출
    - 계산이 복잡해질 것 같다면 표중형을 만들어 표준형의 hashCode 호출
  - 필드의 값이 null이라면 0을 사용한다
  - 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룬다
    - 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다
- 단계 2.a에서 계산한 해시코드 c로 result 갱신

##### 3. result를 반환한다

##### 전형적인 hashCode 메서드

```java
@Override public int hashCode(){
    int result=Short.hashCode(areaCode);
    result=31 * result+Short.hashCode(prefix);
    result=31*result+Short.hashCode(lineNum);
    return result;
}
// 한 줄짜리 hashCode 메서드
// hash 메서드에서 배열이 만들어지고, 박싱과 언박싱을 거치므로 느리다
@Override public int hashCode(){
    return Objects.hash(lineNum,prefix,areaCode);
}
```

- 해시코드 계산시 핵심 필드를 생략해서는 안 된다
- hashCode 반환값 생성 규칙을 API사용자에게 공표하지 않는다
  - 클라이언트가 이 값에 의지하지 않게 되고, 결함이나 더 나은 방식을 발견시 계산 방식을 바꿀 수도 있다

#### 불변 클래스, 해시코드 계산 비용이 큰 경우

- 인스턴스가 만들어 질 때 해시코드를 계산

- 해시의 키로 이용되지 않는 경우

  - hashCode가 처음 불릴때 계산하는 지연 초기화 방식 사용

  - ```java
    private int hashCode;// 자동으로 0으로 초기화
    
    @Override public int hashCode(){
        int result=hashCode;
        if(result==0){
            int result=Short.hashCode(areaCode);
            result=31 * result+Short.hashCode(prefix);
            result=31*result+Short.hashCode(lineNum);
            hashCode=result;
        }
        return result;
    }
    ```

-----

### [item12] toString을 항상 재정의하라

toString을 잘 구현한 클래스는 사용하기 좋고, 디버깅하기 쉽다.

- toString 메서드는 객체를 println,printf,문자열 연결 연산자(+),assert구문에 넘길때,디버거가 객체를 출력할 때 자동으로 불린다
- toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다
- 반환값의 포맷을 문서화할지 정해야 한다
  - 전화번호와 행렬 같은 값 클래스: 문서화
  - 그 값 그대로 입출력에 사용하거나 csv파일처럼 사람이 읽을 수 이쓴 데이터 객체로 저장
  - 포맷을 명시할 때: 명시한 포맷에 맞는 문자열과 객체를 상호 전환 가능한 정적 팩터리나 생성자를 함께 제공
    - 예시: BigInteger, BigDecimal, 대부분의 기본 타입 클래스
  - 하지만 포맷을 한번 명시하면 계속 그 포맷에 얽매이게 된다
    - 향후 릴리스에서 포맷을 바꾼다면 이를 사용하던 코드들과 데이터들은 엉망이 된다
- 포맷을 명시하든 아니든 의도는 명확하게 밝혀야 한다
- 포맷 명시 여부와 관계없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공해야 한다
- 정적 유틸리티 클래스는 toString을 제공할 이유가 없다
  - 대부분의 열거 타입도 따로 재정의 필요 X
  - 하위클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스라면 toString 재정의 필요
    - 예시: 대다수의 컬렉션 구현체 - 추상 컬렉션 클래스의 toString 메서드 상속

-----

### [item13] clone 재정의는 주의해서 진행하라

```
새로운 인터페이스를 만들 때는 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. final 클래스에서는 구현해도 위험이 크지 않으나 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 허용해야 한다.
기본 원칙은 '복제 기능은 생성자와 팩터리를 이용'이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔하다.
```

#### clone 메서드의 일반 규약

이 객체의 복사본을 생성해 반환한다.

'복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.

일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.

`x.clone()!=x` 또한 다음 식도 참이다. `x.clone().getClass() == x.getClass()`

하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다.

한편 다음 식도 일반적으로 참이지만, 역시 필수는 아니다.

`x.clone().equals(x)`

관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.

`x.clone().getClass() == x.getClass()`

관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

#### 클래스가 가변 객체를 참조하는 경우

- 예시: 스택 클래스

- ```java
  public class Stack {
      private Object[] elements;
      private int size=0;
      private static final int DEFAULT_INITIAL_CAPACITY=16;
      
      public Stack(){
          this.elements=new Object[DEFAULT_INITIAL_CAPACITY];
      }
      public void push(Object e){
          ensureCapacity();
          elements[size++]=e;
      }
      public Object pop(){
          if (size==0)
              throw new EmptyStackException();
          Object result=elements[--size];
          elements[size]=null;
          return result;
      }
      
      private void ensureCapacity(){
          if(elements.length==size)
              elements= Arrays.copyOf(elements,2*size+1);
      }
  }
  
  ```

  - clone메서드가 super.clone의 결과를 그대로 반환한다면 Stack 인스턴스의 size필드는 올바른 값을 가지나, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조하게 된다
  - 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다

- clone은 원본 객체에 아무런 해를 끼치지 않으면서 동시에 복제된 객체의 불변식을 보장해야 한다

- elements 필드에 대해 재귀적 clone 호출

- ```java
  @Override public Stack clone(){
          try{
              Stack result=(Stack) super.clone();
              result.elements=elements.clone();
              return result;
          } catch (CloneNotSupportedException e){
              throw new AssertionError();
          }
      }
  ```

- '깊은 복사'를 지원하도록 보강

- 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드 호출

  - 저수준 복사보다 느리다
  - 필드 단위 객체 복사를 우회하기 때문에 전체 Cloneable 아키텍처와는 어울리지 않는다

-----

### [item14] Comparable을 구현할지 고려하라

compareTo는 Object의 메서드가 아니지만, Object의 equals와 성격이 유사하다.

- 단순 동치성 비교, 순서 비교, 제네릭
- Comparable을 구현했다
  - 그 클래스의 인스턴스들에는 자연적인 순서가 있다
  - Arrays.sort(a); 로 정렬 가능하다

#### compareTo 메서드 일반 규약

이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다.

이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.

Comparable을 구현한 클래스는

- 모든 x,y에 대해 sgn(x.compareTo(y))==-sgn(y.compareTo(x))여야 한다
  - 따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다
- 추이성을 보장해야 한다
  - x.compareTo(y) >0이고 y.comparaTo(z)>0이면 x.compareTo(z)>0이다.
- 모든 z에 대해 x.compareTo(y)==0이면 sgn(x.compareTo(z))==sgn(y.compareTo(z))이다
- 필수는 아니지만 지켜져야 하는 것
  - (x.compareTo(y)==0)==(x.equals(y))여야 한다
  - Comparable을 구현하였으나 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다

#### compareTo 메서드

- 각 필드가 동치인지를 비교하는 것이 아닌, 순서를 비교한다

- 자바7부터 compareTo 메서드에서 관계 연산자 <와 >를 사용하는 이전 방식은 오류를 유발한다

  - 박싱된 기본 타입 클래스들에 새로 추가된 compare 메서드를 사용해야 한다

    - 이 방식은 간결하지만 성능 저하가 있다
  - 비교자 생성 메서드를 사용한다
- '값의 차' 비교자는 사용하면 안 된다(추이성 위배)

```java
static Comparator<Object> hashCodeOrder = new Comparator<>(){
    public int compare(Object o1, Object o2){
        return o1.hashCode() - o2.hashCode();
    }
}
```
정수 오버플로우를 일으키거나, 부동소수점 계산방식에 따라 오류 발생

```java
// 정적 compare 메서드 활용 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>(){
    public int compare(Object o1, Object o2){
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
}
// 비교자 생성 메서드 활용 비교자
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(o->o.hashCode);
```