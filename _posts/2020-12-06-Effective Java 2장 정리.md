---
layout: post
title: 201206 Effective Java 2장 정리
tags:
  - Effective Java
---

## 2. 객체 생성과 파괴

### [item 1] 생성자 대신 정적 팩터리 메소드를 고려하자

```
정적 팩터리 메서드와 public 생성자의 장단점을 이해한 후 사용하자.
정적 팩터리 메서드 사용이 유리한 경우가 더 많으므로, 무작정 public 생성자 제공은 지양하자
```

#### 정적 팩터리 메서드가 생성자보다 좋은 점

##### 1. 이름을 가질 수 있다

- 생성자 자체와 매개변수로는 반환된 객체의 특성을 제대로 설명할 수 없다
- 하나의 시그니처(메서드이름+입력 매개변수)=생성자 하나
- 시그니처가 같은 생성자가 여러 개 필요하다면 정적 팩터리 메서드 고려

##### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다

- 불변 클래스(immutable class): 인스턴스를 미리 생성, 새로 생성,캐싱
- 생성 비용이 큰 같은 객체가 자주 요청되는 상황에서 성능 개선
- 인스턴스 통제(instance-controlled)
  - 반복되는 요청에 같은 객체를 반환
  - 클래스를 싱글턴으로 만들 수 있고, 인스턴스화 불가로 만들 수 있다.
  - 불변값 클래스에서 동치인 인스턴스가 단 하나임을 보장한다.
    - 플라이웨이트 패턴의 근간, 열거타입은 인스턴스가 하나만 만들어짐을 보장한다.

##### 3. 반환 타입의 하위 타입 객체를 반환할 수 있다

- 구현 클래스를 공개하지 않고도 그 객체를 반환 가능->API를 작게 유지 가능
- 자바 8부터 인터페이스도 정적 메서드를 가질 수 있다

##### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체 반환이 가능하다

- 클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다
  - 해당 클래스의 하위 클래스이기만 하면 됨

##### 5. 정적 팩터리 메서드를 작성하는 시점에서는 반환할 객체의 클래스가 존재하지 않아도 된다

- '서비스 제공자 프레임워크(service provider framework)'의 근간이 된다
  - ex: JDBC

#### 정적 팩터리 메서드의 단점

##### 1. 상속을 하려면 public/protected 생성자가 필요하다

- 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다

##### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

- 명명 방식 이해, API문서 작성

-----

### [item2] 생성자에 매개변수가 많다면 빌더를 고려하라

```
생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 고려하자
특히 매개변수중 다수가 필수가 아니거나, 같은 타입인 경우!
```

- 정적 팩터리와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다.

#### 1. 점층적 생성자 패턴

- 점층적 생성자 패턴을 사용할 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

#### 2. 자바 빈즈 패턴

- 매개변수들은 기본값이 있다면 기본값으로 초기화하고, 세터를 사용한다.
- 심각한 단점 존재
  - 객체 하나를 만들기 위해 메서드 여러개 호출이 필요하다
  - 객체가 완전히 생성되기 전까지는 일관성이 깨진 객체가 된다
    - 일관성이 깨진 객체가 만들어지면, 버그를 심은 코드와 그 버그때문에 런타임 문제를 겪는 코드가 물리적으로 떨어져 있어 디버깅도 어렵다
- 단점 완화 방안
  - 생성이 끝난 객체를 프리징한다->다루기 어려워서 거의 사용하지 않는다
    - freeze 메소드 호출 여부를 컴파일러가 보증할 수 없다

#### 3. 빌더 패턴

- 점층적 생성자 패턴의 안전성과, 자바빈즈 패턴의 가독성을 가진다.
- 필수 매개변수만으로 생성자(정적팩터리)를 호출, 빌더객체를 얻는다
  - 빌더 객체의 세터로 원하는 선택 매개변수들을 설정한다
  - 매개변수가 없는 build 메서드를 호출해 불변 객체를 얻는다
- 빌더 패턴은 계층적으로 설계된 클래스와 함게 쓰기 좋다
- 추상 클래스: 추상 빌더, 구체 클래스: 구체 빌더
- 단점
  - 객체 생성에 앞서 빌더부터 만들어야 하므로 성능에 민감한 경우 문제
  - 코드가 장황해지므로 매개변수가 4개 이상일 때부터 효율적

#### 불변과 불변식

- 불변(immutable): 어떠한 변경도 허용하지 않는다
- 불변식(invariant): 프로그램이 실행되는 동안/정해진 기간 동안 반드시 만족해야 하는 조건->주어진 조건 내에서만 변경 가능함

#### 공변 반환 타이핑(covariant return typing)

- 하위 클래스의 메서드가...
  - 상위 클래스의 메서드가 정의한 반환 타입(X)
  - 그 하위 타입을 반환하는 기능(O)
- 클라이언트가 형변환에 신경쓰지 않고 빌더를 사용할 수 있다

-----

### [item3] private 생성자나 열거 타입으로 싱글턴임을 보장하라

#### 싱글턴

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
  - ex: 함수등의 stateless 객체, 설계상 유일해야 하는 컴포넌트
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트 테스트가 어렵다

#### 싱글턴을 만드는 방식

- 생성자를 private로 감춘다
- 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 마련한다

##### 1. public static final

``` java
public class Elvis{
    public static final Elvis INSTANCE = new Elvis();
    private Elvis(){...}
    
    public void leaveTheBuilding() {...}
}
```

- public,protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다
  - 예외: 권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다
  - 생성자를 수정해 두번째 객체 생성시 예외를 던지도록 한다
- 해당 클래스가 싱글턴임이 API에 명백하게 드러나며, 간결하다.

##### 2. 정적 팩터리 메서드를 public static 멤버로 제공

``` java
public class Elvis{
    public static final Elvis INSTANCE = new Elvis();
    private Elvis(){...}
    public static Elvis getInstance() { return INSTANCE; }
    
    public void leaveTheBuilding() {...}
}
```

- API의 변경 없이 싱글턴이 아니게 변경할 수 있다
  - 예시: 유일한 인스턴스 반환->스레드별 다른 인스턴스 반환
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
- 정적 팩터리의 메서드 참조를 공급자로 이용 가능하다.
  - 예시: Elvis::getInstance -> Supplier<Elvis>

##### 싱글턴 클래스 직렬화

Serializable 구현 + 모든 인스턴스 transient 선언 + readResolve 메소드 제공

- 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다

##### 3. 원소가 하나인 열거타입 싱글턴

``` java
public enum Elvis{
    INSTANCE;
    public void leaveTheBuilding() {...}
}
```

- public 필드 방식보다 더 간결하고, 직렬화 가능하며, 아주 복잡한 직렬화 상황이나 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 막아준다.
- 그러나 만들려는 싱글턴이 enum 외의 클래스를 상속한다면 이 방법은 사용할 수 없다

-----

### [item4] 인스턴스화를 막으려면 private 생성자를 사용하라

- 정적 메서드와 정적 필드만을 담을 클래스가 필요할때
  - 객체지향적이진 않으나 쓰임새가 있다
  - 예시: java.lang.Math, java.util.Arrays, java.util.Collections
  - 생성자를 명시하지 않으면 컴파일러가 기본 pubilc 생성자를 생성한다.
- 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다
  - 컴파일러가 기본 생성자를 만드는 경우: 오직 명시된 생성자가 없을 때 뿐이다.
- private 생성자를 추가하여 클래스의 인스턴스화를 막는다.

``` java
public class UtilityClass {
    // 기본 생성자가 만들어지는 것을 막는다
    private UtilutyClass(){
        throw new AssertionError();
    }
    ...
}
```

- 상속을 불가능하게 하는 효과도 있다
  - 모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 된다
    - private으로 선언하였기 때문에 상속불가

-----

### [item5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

```
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 대신 필요한 자원 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성,재사용성,테스트 용이성을 개선해준다.
```

- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.
- 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식
  - 의존 객체 주입의 한 형태

``` java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary){
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) {...}
    public List<String> suggestions(String typo) {...}
}
```

- 의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 큰 프로젝트에서는 코드를 어지럽게 만든다
  - spring 등의 의존 객체 주입 프레임워크 사용

-----

### [item6]  불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는편이 낫다
- `String s=new String("bikini");` 보다 `String s="bikini";`를 사용한다
- 생성자 대신 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피한다
  - `Boolean(String)` 생성자 대신 `Boolean.valueOf(String)`팩터리 메서드를 사용한다

#### 생성 비용이 아주 비싼 객체

- 캐싱하여 재사용하길 권한다

- 예시

  - as-is

    ``` java
    static boolean isRomanNumeral(String s) {
    	return s.matches(정규표현식);
    }
    ```

  -   to-be

      ``` java
      public class RomanNumerals {
          private static final Pattern ROMAN = Pattern.compile(정규표현식);
          
          static boolean isRomanNumeral(String s) {
      		return ROMAN.matcher.matches();
      	}
      }
      ```

  -   String.matches는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다.

  -   Pattern 인스턴스를 static final 필드로 끄집어내고 이름을 지어주어 코드의 의미가 더 잘 드러난다.


#### 명확하지 않고, 직관에 반대되는 상황

- 어댑터(뷰)
  - 실제 작업은 뒷단 객체에 위임, 자신은 제 2의 인터페이스 역할을 해주는 객체
  - 어댑터는 뒷단 객체만 관리: 뒷단 객체 하나당 어댑터 하나씩
  - 예시 : Map 인터페이스
    - keySet 메서드는 Map 객체 안의 키 전부를 담은 Set 뷰를 반환
    - 반환된 Set 인스턴스가 일반적으로 가변이더라도 반횐된 인스턴스들은 기능적으로 모두 똑같다
    - 반환한 객체 중 하나를 수정하면 다른 모든 객체가 따라서 바뀐다.(모두가 똑같은 Map 인스턴스)
- 오토박싱
  - 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호변환해주는 기술
  - 의미상으로는 별다를 것 없지만 성능에서는 그렇지 않다
  - 박싱된 기본 타입 `Long` 보다는 기본 타입`long`을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

- 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 자체 객체 풀을 만들지는 말자
  - 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리며, 성능을 떨어뜨린다

-----

### [item7] 다 쓴 객체 참조를 해제하라

```
메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.
```

- 예시: 스택

``` java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY=16;
    
    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e){
        ensureCapacity();
        elements[size++]=e;
    }
    
    public Object pop(){
        /* 메모리 누수 발생
        if(size==0)
            throw new EmptyStackException();
        return elements[--size];
        */
        if(size==0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size]=null; // 다 쓴 참조는 해제!!
        return result;
    }
    
    private viod ensureCapacity(){
        if(elements.length==size)
            elements = Arrays.copyOf(elements, 2*size+1);
    }
}
```

- 다 쓴 참조는 null 처리(참조 해제) 하면 된다.
- 자기 메모리를 직접 관리하는 클래스라면 메모리 누수에 주의해야 한다
- 캐시 역시 메모리 누수를 일으킨다
  - WeakHashMap을 사용해 캐시를 만든다
- 리스너 혹은 콜백도 메모리 누수를 일으킨다
  - 콜백을 약한 참조로 저장(WeakHashMap에 키로 저장)하면 가비지 컬렉터가 수거한다

-----

### [item8] finalizer와 cleaner 사용을 피하라

```
cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용한다(불확실성화 성능 저하에 주의)
```

- finalizer는 예측할 수 없고, 상황에 따라 위험할 수도 있어 일반적으로 불필요함
  - 오동작,낮은 성능, 이식성 문제....
- 자바 9에서는 cleaner를 대안으로 소개했다
  - 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다

#### finalizer와 cleaner를 사용할 때의 단점

- 즉시 수행된다는 보장이 없다
  - finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다
  - 예시: 파일 닫기 - 시스템이 finalizer, cleaner 실행을 미루다가 파일을 계속 열어둘 수 있다
- 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다
  - 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다
  - 예시: 데이터베이스 같은 공유 자원의 영구 락 해제를 finalizer,cleaner에 맡겨 놓으면 분산 시스템 전체가 서서히 멈출 것이다
- finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다
- 심각한 성능 문제를 동반한다
  - finalizer를 사용한 객체를 생성하고 파괴하니 50배나 느렸다
  - cleaner도 클래스의 모든 인스턴스를 수거하는 형태로 사용하면 성능은 finalizer와 비슷하다
- finalizer를 사용한 클래스는 finalizer 공격에 노출되에 심각한 보안 문제를 일으킬 수 있다
  - 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다

#### finalizer와 cleaner를 적절히 활용하는 예

- 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
  - 늦는것이 아예 안 하는 것보다 나을때
- 네이티브 피어(native peer)와 연결된 객체
  - 네이티브 피어: 일반 자바객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
  - 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못하므로, 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못한다.
  - 성능 저하를 감당할 수 있고, 네이티브 피어가 심각한 자원을 가지고 있지 않을때에만 사용한다.
  - 그렇지 않다면 close 메서드를 사용해야 한다.

-----

### [item9] try-finally 보다는 try-with-resource를 사용하라

```
꼭 회수해야 하는 자원을 다룰 때는 try-finally보다 try-with-resource를 사용하자. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다.
```

- 자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다
  - InputStream, OutputStream, java.sql.Connection 등...

#### try-finally

``` java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try{
        return br.readLine();
    } finally{
        br.close();
    }
}
```



``` java
static void copy(String src, String dst) throws IOException{
    InputStream in = new FileInputStream(src);
    try{
        outputStream out = new FileOutputStream(dst);
        try{
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n=in.read(buf))>=0)
                out.write(buf,0,n);
        }finally{
            out.close();
        }
    } finally{
        in.close();
    }
}
```

- 자원이 둘 이상인 경우 try-finally 방식은 지저분해진다.

#### try-with-resources

``` java
static String firstLineOfFile(String path) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    }
}
```
- readLine, close 호출 양쪽에서 예외가 발생하면 close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다.
  - getSuppressed 메서드를 이용하여 close 예외를 가져올 수 있다.
``` java
static void copy(String src, String dst) throws IOException{
    try(InputStream in = new FileInputStream(src);
        outputStream out = new FileOutputStream(dst)){
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n=in.read(buf))>=0)
                out.write(buf,0,n);
    }
}
```

- try-with-resource 버전이 짧고 읽기 수월하며, 문제를 진단하기도 훨씬 좋다.
- try-with-resource 에서도 catch 절을 쓸 수 있다
  - try문 중첩 없이 다수의 예외 처리가 가능하다

``` java
static String firstLineOfFile(String path, String defaultVal) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    } catch (IOException e){
        return defaultVal;
    }    
```

