---
layout: post
title: 201227 Effective Java 5장 정리
tags:
  - Effective Java
---

## 5. 제네릭

제네릭은 자바 5부터 사용할 수 있다.

제네릭을 지원하기 전

- 컬렉션에서 객체를 꺼낼 때마다 형변환 필요
- 엉뚱한 타입의 객체를 넣어두면 런타임에 형변환 오류 발생

제네릭 지원

- 컴파일러는 알아서 형변환 코드를 추가
- 엉뚱한 타입의 객체를 넣으려는 시도를 컴파일 과정에서 차단

용어 정리

![](https://raw.githubusercontent.com/Heunnnn/heunnnn.github.io/master/images/post/20201227.jpg)

### [item26] 로 타입은 사용하지 말라

- 제네릭 타입을 하나 정의하면 그에 딸린 로 타입(raw type)도 함께 정의된다
  - 로 타입: 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때
  - 예시: `List<E>`의 로 타입은 `List`
  - 제네릭 지원 전 코드와 호환되도록 하기 위한 궁여지책이다

#### 로 타입 사용 예

```java
/* Stamp 인스턴스만 취급 */
private final Collection stamps = ...; // 컬렉션 로타입 : 사용X
stamps.add(new Coin(...)); // unchecked call 경고를 보여주나, 오류없이 컴파일 발생,실행됨
for(Iterator i = stamps.iterator();i.hasNext();){
    Stamp stamp = (Stamp) i.next(); // ClassCastException!
    stamp.cancel();
}
```

- 오류는 가능한 한 발생 즉시, 이상적으로는 컴파일할때 발견하는 것이 좋다
- 위 예시에서는 오류가 발생하고 한참 뒤인 런타임에 문제를 발견
  - 런타임에 문제를 겪는 코드와 원인은 제공한 코드가 물리적으로 상당히 떨어져 있을 가능성이 높다
  - 주석은 컴파일러가 이해하지 못하므로, 별 도움이 되지 않는다

#### 제네릭 활용, 타입 선언 자체에 정보 추가

```java
private final Collection<Stamp> stamps = ...;
```

- 컴파일러는 stamps에는 Stamp 인스턴스만 넣어야 함을 인지한다
- 컬렉션에서 원소를 꺼내는 모든 곳에 보이지 않는 형변환을 추가, 절대 실패하지 않음을 보장한다.

#### 로 타입 사용을 막지 않은 이유

- raw타입 사용시 제네릭이 안겨주는 안전성/표현력을 모두 잃게 된다!
- 자바가 제네릭을 받아들이기까지 오랜 시간이 걸려, 기존 코드를 모두 수용하면서도 제네릭을 사용하는 새로운 코드와도 맞물려 돌아가도록 해야 했다
- 로 타입 매개변수/메서드 - 매개변수화 인스턴스/메서드 호환이 필요했음
  - 이 호환성 지원을 위해 로 타입 지원, 제네릭 구현에는 소거(erasure) 방식 사용

#### List와 `List<Object>`

- List는 제네릭 타입에서 완전히 발을 뺐다

- `List<Object>`는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한 것

- `List<String>`은 로 타입인 List의 하위 타입이나, `List<Object>`의 하위 타입은 아니다

- 따라서 `List<Object>`같은 매개변수화 타입을 사용할 때와 달리 List같은 로 타입을 사용하면 타입 안정성을 잃는다

- ```java
  public static void main(String[] args){
      List<String> strings=new ArrayList<>();
      unsafeAdd(strings,Integer.valueOf(42));
      String s = strings.get(0); // 런타임에 ClassCastException
  }
  private static void unsafeAdd(List list, Object o){
      list.add(o);				// 컴파일 경고 발생
  }
  
  private static void unsafeAdd(List<Object> list, Object o){
      list.add(o);				// (incompatible types)컴파일 오류 발생
  }
  ```

#### 원소의 타입을 몰라도 되는 로 타입 사용

```java
static int numElementsInCommon(Set s1, Set s2){
    int result = 0;
    for(Object o1:s1){
        if(s2.contains(o1))
            result++;
    }
    return result;
}
```

- 위 메서드는 동작은 하지만 로 타입을 사용하므로 안전하지 않다

- 비한정적 와일드카드 타입 사용

  - 제네릭 타입을 쓰고 싶지만, 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않다면 `<?>` 사용

  - 비한정적 와일드카드 타입 사용, 위 메서드를 다시 선언하면

    - ```java
      static int numElementsInCommon(Set<?> s1, Set<?> s2)
      ```

#### 비한정적 와일드카드 타입 VS 로 타입

- 와일드카드 타입은 안전하고, 로 타입은 안전하지 않다
  - 로 타입 컬렉션에는 아무 원소나 넣을 수 있어 타입 불변식을 훼손하기 쉽다
  - Collection<?>에는 (null외에는) 어떤 원소도 넣을 수 없다.
    - 컴파일시 오류 발생
    - 이러한 제약을 받아들일 수 없다면 제네릭 메서드 또는 한정적 와일드카드 타입을 사용한다

#### 로 타입을 사용할 수 있는 예외

- class 리터럴에는 로 타입을 사용해야 한다

  - 예시: `List.class`, `String[].class`, `int.class`는 허용된다 / `List<String>.class`, `List<?>.class`는 허용되지 않는다

- instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다

  - 런타임에는 제네릭 타입 정보가 지워짐

  - 로 타입이든 비한정적 와일드카드 타입이든 instanceof 는 완전히 똑같이 동작한다

  - 제네릭 타입에 instanceof를 사용하는 올바른 예

    - ```java
      if(o instanceof Set) {		// 로 타입
          Set<?> s = (Set<?>) o;	// 와일드카드 타입
      }
      ```

-----

### [item27] 비검사 경고를 제거하라

#### 할 수 있는 한 모든 비검사 경고를 제거하라

- 제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 볼 수 있다
  - 비검사 형변환 경고
  - 비검사 메서드 호출 경고
  - 비검사 매개변수화 가변인수 타입 경고
  - 비검사 변환 경고.......
- 비검사 결고를 제거한다면 그 코드는 타입 안전성이 보장된다
  - 런타임에 ClassCastException이 발생할 일이 없다
- 경고를 제거할 수 없으나, 타입 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨긴다
  - 안전하다고 검증된 비검사 경고를 숨기지 않고 둔다면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다.

#### @SuppressWarnings 애너테이션은 항상 가능한 한 좁은 범위에 적용하라

- 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다

- 절대로 클래스 전체에 적용해서는 안 된다

  - 변수 선언, 아주 짧은 메서드 혹은 생성자

  - 한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 애너테이션은 지역변수 선언 쪽으로 옮긴다

  - ```java
    // 지역 변수를 추가하여 @SuppressWarnings의 범위를 좁힌다
    public <T> T[] toArray(T[] a){
        if(a.length < size){
            /*
            *	생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로
            *	올바른 형변환이다.
            */
            @SuppressWarnings("unchecked") T[] result = 
                (T[]) Arrays.copyOf(elements,size,a.getClass());
            return result;
        }
        System.arraycopy(elements,0,a,0,size);
        if(a.length>size)
            a.size = null;
        return a;
    }
    ```
    
  - `@SuppressWarnings("unchecked")` 애너테이션을 사용할 때, 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다
-----

### [item28] 배열보다는 리스트를 사용하라

#### 배열과 제네릭 타입의 차이

- 배열은 공변(共變, covariant)타입

  - Sub가 Super의 하위 타입이라면, 배열 Sub[]은 배열 Super[]의 하위 타입이다

  - 즉, 서로 다른 타입 T1과 T2가 있을때, `List<T1>`은 `List<T2>`의 하위 타입도 아니고 상위 타입도 아니다.

  - ```java
    // 문법상 허용됨(런타임에 실패한다)
    Object[] objectArray = new Long[1];
    objectArray[0]="타입이 달라 넣을 수 없다"; // ArrayStoreException을 던진다
    
    // 문법에 맞지 않음(컴파일되지 않는다)
    List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입
    ol.add("타입이 달라 넣을 수 없다")
    ```
    
  - 배열에서는 실수를 런타임에 알 수 있으나, 리스트 사용시 컴파일 시점에서 알 수 있다
  
- 배열은 실체화(reify) 된다.

  - 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다
    - 앞의 사례에서, Long 배열에 String을 넣으려 하면 ArrayStoreException이 발생한다
    - 제네릭은 타입 정보가 런타임에는 소거(erasure) 된다
      - 원소 타입을 컴파일타임에만 검사하며, 런타임에는 알 수 없다
      - 소거: 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 한다

- 위의 두 가지 차이점으로 인해 배열과 제네릭은 잘 어우러지지 않는다

  - 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.
  - `new List<E>[]`,`new List<String>[]`, `new E[]`와 같이 작성하면 컴파일시 제네릭 배열 생성 오류를 일으킨다

#### 제네릭 배열을 만들지 못하도록 막은 이유

- 타입 안전하지 않기 때문이다
- 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException 발생
  - 런타임에 ClassCastException을 방지하겠다는 제네릭 타입 시스템의 취지에 어긋나는 것이다

```java
List<String>[] stringLists = new List<String>[1];	// (1)
List<Integer> intList = List.of(42);				// (2)
Object[] objects = stringLists;						// (3)
objects[0] = intList;								// (4)
String s = stringLists[0].get(0);					// (5)
```

- (1)이 허용된다고 가정하면
  - (2)는 원소가 하나인 `List<Integer>`를 생성한다
  - (3)은 (1)에서 생성한 `List<String>` 배열을 Object 배열에 할당한다(배열은 공변이므로 문제가 없다)
  - (4)는 (2)에서 생성한 `List<Integer>`의 인스턴스를 Object 배열의 첫 원소로 저장한다(제네릭슨 소거 방식으로 구현되어 이도 성공한다)
  - 즉, 런타임에는  `List<Integer>` 인스턴스의 타입은 단순히 List가 되고,  `List<Integer>[]`인스턴스의 타입은 List[]가 되면서 (4)에서도 ArrayStoreException을 일으키지 않는다.
  - 하지만 `List<String>` 인스턴스만 담겠다고 선언한 stringLists 배열에는 현재 `List<Integer>` 인스턴스가 저장되어 있고, (5)에서 이 배열의 처음 리스트에서 첫 원소를 꺼내려 한다.
  - 컴파일러는 꺼낸 원소를 자동으로 String으로 형변환하는데, 이 원소는 Integer이므로 런타임에 ClassCastException이 발생한다
- 이런 일을 방지하려면 제네릭 배열이 생성되지 않도록 (1)에서 컴파일 오류를 내야 한다.
- 배열을 제네릭으로 만들 수 없어 발생하는 귀찮은 일들
  - 제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는 게 보통은 불가능하다
  - 제네릭 타입과 가변인수 메서드를 함께 쓰면 해석하기 어려운 경고 메시지를 받게 된다
    - 가변인수 메서드를 호출할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어지는데, 이때 그 배열의 원소가 실체화 불가 타입이라면 경고 발생(@SafeVarargs 애너테이션으로 대처)
- 배열로 형변환할때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우, 대부분은 배열인 `E[]`대신 컬렉션인 `List<E>`를 사용하면 해결된다
  - 코드가 조금 복잡해지고 성능이 조금 나빠질 수 있다
  - 타입 안정성과 상호운용성이 좋아진다

#### 예시: Chooser 클래스 구현

컬렉션 안의 원소 중 하나를 무작위로 선택해 반환하는 choose 메서드를 제공하는 클래스이다.

```java
// 1. 제네릭 사용 없이 구현
public class Chooser{
    private final Object[] choiceArray;
    
    public chooser(Collection choices){
        choiceArray = choices.toArray();
    }
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

- 이 클래스를 사용하려면 choose 메서드를 호출할때마다 반환된 Object를 원하는 타입으로 형변환해야 한다
- 타입이 다른 원소가 들어있었다면 런타임에 형변환 오류가 난다

```java
// 2. 제네릭으로 만들기 위한 첫 시도
public class Chooser<T>{
    private final T[] choiceArray;
    
    public chooser(Collection<T> choices){
        choiceArray = (T[]) choices.toArray(); // Object 배열을 T 배열로 형변환
    }
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

- Object 배열을 T 배열로 형변환?
  - 컴파일시 경고메시지 발생
  - T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다

```java
// 3. 리스트 기반 Chooser
public class Chooser<T>{
    private final List<T> choiceList;
    
    public chooser(Collection<T> choices){
        choiceList = new ArrayList<>(choices); 
    }
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size());
    }
}
```

- 비검사 형변환 경고를 제거하기 위해 배열 대신 리스트를 사용했다.
- 런타임에 ClassCastException을 만날 가능성이 사라졌다

-----

### [item29] 이왕이면 제네릭 타입으로 만들라

#### 제네릭 타입으로 변환하기 예시: Stack 클래스

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제!
        return result;
    }
    
    public boolean isEmpty(){
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

- 지금 상태의 클라이언트: 스택에서 꺼낸 객체를 형변환해야 하는데, 이때 런타임 오류가 날 위험이 있다

```java
/* 일반 클래스를 제네릭 클래스로 만드는 첫 단계 
*  1. 클래스 선언에 타입 매개변수를 추가한다
*  2. Object를 적절한 타입 매개변수로 바꾸고 컴파일한다
*/
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    ...
}
```

- 이 단계에서는 컴파일 오류가 발생한다

- E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다

  - 제네릭 배열 생성을 금지하는 제약을 우회한다: Object 배열 생성 후 제네릭 배열로 형변환

    - 컴파일러는 오류 대신 경고를 내보낸다(warning: [unchecked] unchecked cast...)

    - 일반적으로 타입 안전하지 않다: 이 비검사 형변환이 프로그램의 타입 안전성을 해치지 않는지 우리가 확인해야 한다

    - 비검사 형변환이 안전함을 직접 증명했다면 범위를 최소로 좁혀 `@SuppressWarnings` 애너테이션으로 해당 경고를 숨긴다

    - ```java
      @SuppressWarnings("unchecked")
      public Stack() {
      	this.elements = new E[DEFAULT_INITIAL_CAPACITY];
      }
      ```

    - 가독성이 더 좋다

      - 배열의 타입을 E[]로 선언, 오직 E타입 인스턴스만 받음을 확실히 어필한다

    - 코드가 더 짧다

      - 형변환을 배열 생성시 단 한번만 해주면 된다

    - 이 방식을 더 선호하지만, 힙 오엽(heap pollution)을 일으킨다

      - 배열의 런타임 타입이 컴파일타임 타입과 다르다.

  - elements 필드의 타입을 E[]에서 Object[]로 바꾼다

    - 오류 발생(incompatible types)

    - 배열이 반환한 원소를 E로 형변환시 오류 대신 경고가 뜬다

    - 이번에도 형변환이 안전함을 증명하고, 경고를 숨긴다

    - ```java
      public E pop() {
      	if (size == 0)
          	throw new EmptyStackException();
          // push에서 E 타입만 허용하므로 이 형변환은 안전하다
          @SuppressWarnings("unchecked")
          E result = (E) elements[--size];
          elements[size] = null;
          return result;
      }
      ```

```java
public static void main(String[] args){
    Stack<String> stack = new Stack<>();
    for(String arg: args)
        stack.push(arg);
    while(!stack.isEmpty())
        System.out.println(stack.pop().toUpperCase());
}
```

- Stack에서 꺼낸 원소에서 String의 toUpperCase 메서드를 호출할 때 명시적 형변환을 수행하지 않는다
- 컴파일러에 의해 자동 생성된 이 형변환이 항상 성공함을 보장한다

#### 대다수의 제네릭 타입 특징

- 타입 매개변수에 아무런 제약을 두지 않는다

  - 단, 기본 타입은 사용할 수 없다.(컴파일 오류 발생)
  - 박싱된 기본 타입을 사용해 우회한다

- 타입 매개변수에 제약을 두는 제네릭 타입도 있다

  - 예시: java.util.concurrent.DelayQueue

    - ```java
      class DelayQueue<E extends delayed> implements BlockingQueue<E>
      ```

    - `<E extends Delayed>`: java.util.concurrent.Delayed의 하위 타입만 받겠다

      - DelayQueue의 원소에서 형변환 없이 곧바로 Delayed 클래스의 메서드를 호출할 수 있다.
      - 모든 타입은 자기 자신의 하위타입이므로, `DelayQueue<Delayed>`와 같이 사용할 수 있다

-----

### [item30] 이왕이면 제네릭 메서드로 만들라

```java
// 로 타입 사용: 사용X
public static Set union(Set s1, Set s2){
    Set result=new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

- 컴파일은 되지만 경고 발생

- 경고를 없애려면 이 메서드를 타입 안전하도록 만든다

  - 메서드 선언에서의 세 집합(입력 2개, 반환 1개)의 원소 타입을 타입 매개변수로 명시

  - 메서드 안에서도 이 타입 매개변수만 사용하도록 수정

  - ```java
    public static <E> Set<E> union(Set<E> s1, Set<E> s2){
        Set<E> result=new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
    ```

#### 제네릭 싱글턴 팩터리

- 불변 객체를 여러 타입으로 활용할 수 있도록 만들어야 할 때가 있다

- 제네릭은 런타임에 타입 정보가 소거
  - 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다
  - 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다
- Collections.reverseOrder 같은 함수 객체나 Collections.emptySet 같은 컬렉션용으로 사용한다.

```java
// 항등 함수: 입력 값을 수정 없이 반환한다
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressedWarinings("unchecked")
public static <T> UnaryOperator<T> identityFunction(){
    return (UnaryOperator<T>) IDENTITY_FN;
}

public static void main(String[] args){
    String[] strings = {"삼베", "대마", "나일론"};
    UnaryOperator<String> sameString= identityFunction();
    for(String s: strings){
        System.out.println(sameString.apply(s));
    }
    
    Number[] numbers={1,2.0,3L};
    UnaryOperator<Number> sameNumber=identityFunction();
    for(Number n: numbers){
        System.out.println(sameNumber.apply(n));
    }
}
```

#### 재귀적 한정 타입

- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다

- 주로 Comparable 인터페이스와 함께 쓰인다

  - ```java
    public interface Comparable<T>{
        int compareTo(T o);
    }
    ```

  - 실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교 가능하다

- Comparable을 구현 : 정렬, 검색, 최소/최대값 구하기....

  - 이 기능을 사용하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 한다

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

  - `<E extends Comparable<E>>` : 모든 타입 E는 자신과 비교할 수 있다 == 상호 비교 가능하다

```java
// 컬렉션에 담김 원소의 자연적 순서를 기준으로 최댓값을 계산
// 컴파일 오류나 경고는 발생하지 않는다
public static <E extends Comparable<E>> E max(Collection<E> c){
    if(c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다");
    E result = null;
    for(E e: c)
        if(result == null || e.compareTo(result)>0){
            result = Objects.requireNonNull(e);
        }
    return result;
}
```

----

### [item31] 한정적 와일드카드를 사용해 API유연성을 높이라

item29의 Stack 클래스의 public API만 추려본다면

```java
public class Stack<E>{
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

여기에 일련의 원소를 스택에 넣는 메서드를 추가한다고 가정하자

```java
public void pushAll(Iterable<E> src){
    for(E e: src){
        push(e);
    }
}
```

- 이 메서드는 깨끗하게 컴파일되지만 완벽하지 않다
- 하지만 `Stack<Number>`로 선언하고, pushAll(intVal)로 호출한다면?
  - Integer는 Number의 하위 타입이니 잘 동작해야 할 것으로 예상되지만....
  - 컴파일 오류 메시지가 발생한다(매개변수화 타입이 불공변이기 때문이다!)

#### 한정적 와일드카드 타입

```java
public void pushAll(Iterable<? extends E> src){
    // Stack이 사용할 E 인스턴스를 생산한다
    for(E e: src){
        push(e);
    }
}
```

- pushAll의 입력 매개변수 타입은 `E의 Iterable`이 아닌 `E의 하위 타입의 Iterable`이어야 한다

- 모든 것이 타입 안전하고, 깨끗하게 컴파일되며 실행된다!

- 이것을 popAll에 적용해서 작성해보자

- ```java
  xpublic void popAll(Collection&lt;? super E&gt; dst){
  xㅏ    // Stack으로부터 E 인스턴스를 소비한다
      while(!isEmpty())
          dst.add(pop());
  }
  ```

- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라

#### PECS 공식: 생산자(producer)는 extends, 소비자(consumer)는 super

- 매개변수화 타입 T가 생산자라면 `<? extends T>` 사용
- 매개변수화 타입 T가 소비자라면 `<? super T>` 사용

#### 예시: 앞 장의 메서드와 생성자 선언 수정

item28 Chooser 클래스 생성자

```java
public Chooser(Collection<? extends T> choices)
```

- 이 생성자로 넘겨지는 choices 컬렉션은 T 타입의 값을 생산하기만 한다
- `Chooser<Number>`의 생성자에 `List<Integer>`를 넘기고 싶다?
  - 수정 전: 컴파일조차 되지 않는다
  - 수정 후: 생성자에서는 문제가 사라진다

item30 union  메서드

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

- s1과 s2 모두 E의 생산자이다

- 반환 타입은 여전히 `Set<E>`이다 : 반환 타입에는 한정적 와일드카드 타입을 사용해서는 안 된다

- 하지만 이 코드는 자바 8부터 정상 컴파일된다

- 자바7까지는 명시적 타입 인수를 사용해야 한다

  - ```java
    Set<Number> numbers = Union.<Number>union(integers,doubles);
    ```

item30 max 메서드

```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

- PECS 공식이 두번 적용되었다
  - 입력 매개변수에서는 E 인스턴스를 생산: `List<E>` -> `List<? extends E>`
  - 타입 매개변수에서는 `Comparable<E>` 인스턴스를 소비한다
    - `Comparable<E>`->`Comparable<? super E>`

#### 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체라하

```java
public static void swap(List<?> list, int i, int j){
    list.set(i,list.set(j,list.get(i)));
}
```

- 컴파일시 오류 발생
  - `List<?>`에는 null 외에는 어떤 값도 넣을 수 없다
  - 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용한다
  - 실제 타입을 알아내려면, 이 도우미 메서드는 제네릭 메서드여야 한다

```java
public static void swap(List<?> list, int i, int j){
    swapHelper(list,i,j);
}
// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
public static <E> void swapHelper(List<E> list, int i, int j){
    list.set(i,list.set(j,list.get(i)));
}
```

- swap 메서드 내부에서는 더 복잡한 제네릭 메서드를 이용했지만, 외부에서는 와일드카드 기반 선언을 유지할 수 있다

----

### [item32] 제네릭과 가변인수를 함께 쓸 때는 신중하라

- 가변인수와 제네릭은 궁합이 좋지 않다
  - 가변인수 기능: 배열을 노출하면서 추상화가 완벽하지 못하다
  - 배열과 제네릭의 타입 규칙이 서로 다르기 때문이다
- 제네릭 varargs 매개변수는 타입 안전하지는 않으나 허용된다

```java
// 제네릭과 varargs를 혼용하면 타입 안정성이 깨진다
static void dangerous(List<String>... stringLists){
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;						
	objects[0] = intList;		// 힙 오염 발생!
	String s = stringLists[0].get(0);	// ClassCastException
}
```

- 이 메서드에서는 형변환하는 곳이 보이지 않아도, 인수를 건네 호출하면 ClassCastException이 발생한다
  - 마지막줄에 컴파일러가 생성한 보이지 않는 형변환이 숨겨져있기 때문이다.
- 이처럼 타입 안정성이 깨지니, 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

#### 제네릭 배열을 생성하는것은 허용하지 않으나, 제네릭 varargs 배열 매개변수를 선언할 수 있는 이유?

- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하다!
- 예시
  - Arrays.asList(T... a)
  - Collections.addAll(Collection<? super T> c, T.. elements)
  - EnumSet.of(E first, E... rest)

#### @SafeVarargs

- 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치

- 자바 7에서 추가, 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다

  - 메서드가 안전한 것이 확실하지 않다면 절대 @SafeVarargs 애너테이션을 달아서는 안 된다

- 안전한 제네릭 varargs 메서드?

  - 가변인수 메서드를 호출할때 varargs 매개변수를 담는 제네릭 배열이 만들어진다
    - 메서드가 이 배열에 아무것도 저장하지 않는다
    - 이 배열의 참조가 밖으로 노출되지 않는다
  - varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 역할만 한다면 그 메서드는 안전하다

- 하지만 varargs 매개변수 배열에 아무것도 저장하지 않고도 타입 안정성을 깰 수도 있다

- ```java
  static <T> T[] toArray(T... args){
      return args;
  }
  ```

  - 반환 배열 타입: 컴파일타임에 결정.
  - 자신의 varargs 매개변수 배열을 그대로 반환하면서 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로 전이한다.

- ```java
  static <T> T[] pickTwo(T a, T b, T c){
      switch(ThreadLocalRandom.current().nextInt(3)){
          case 0: return toArray(a,b);
          case 1: return toArray(a,c);
          case 2: return toArray(b,c);
      }
      throw new AssertionError(); // 도달할 수 없다
  }
  
  public static void main(String[] args){
      String[] arrtibutes=pickTwo("좋은","빠른","저렴한");
  }
  ```

  - 경고 없이 컴파일되고, 실행할 때 ClassCastException 발생
  - pickTwo의 반환값을 arrtibutes에 저장하기 위해 String[]로 형변환하는 코드를 컴파일러가 자동 생성한다
  - Object[]는 String[]의 하위 타입이 아니므로 이 형변환은 실패한다

- @SafeVarargs 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다

  - 재정의한 메서드도 안전할지 보장할 수 없기 때문
  - 자바8: 정적 메서드, final 인스턴스 메서드에만 붙일 수 있다
  - 자바9: private 인스턴스 메서드에도 허용

#### 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하는 예외

- @SafeVarArgs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다
- 그저 이 배열 내용의 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다

#### 예시: 제네릭 varargs 매개변수 안전하게 사용하기

```java
// 1. @SafeVarargs 애너테이션 사용
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists){
    List<T> result = new ArrayList<>();
    for (List<? extends T> list:lists)
        result.addAll(list);
    return result;
}
```

```java
// 2. 제네릭 varargs 매개변수를 List로 대체
@SafeVarargs
static <T> List<T> flatten(List<List<? extends T>> lists){
    List<T> result = new ArrayList<>();
    for (List<? extends T> list:lists)
        result.addAll(list);
    return result;
}
```

위의 2. 방식을 pickTwo 메서드에 적용하면

```java
static <T> List<T> pickTwo(T a, T b, T c){
    switch(ThreadLocalRandom.current().nextInt(3)){
        case 0: return List.of(a,b);
        case 1: return List.of(a,c);
        case 2: return List.of(b,c);
    }
    throw new AssertionError();
}
public static void main(String[] args){
    List<String> arrtibutes=pickTwo("좋은","빠른","저렴한");
}
```

-----

### [item33] 타입 안전 이종 컨테이너를 고려하라

제네릭은 컬렉션(`Set<T>`, `Map<K,V>`)과 단일 원소 컨테이너(`ThreadLocal<T>`,`AtomicReference<T>`)에도 흔히 쓰인다

- 이런 모든 쓰인에서 매개변수화되는 대상은 (원소가 아닌) 컨테이너 자신이다
- 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다
  - 예: Set에는 원소의 타입을 뜻하는 단 하나의 타입 매개변수, Map에는 키와 값의 타입을 뜻하는 2개 필요...

더 유연한 수단이 필요할 때도 있다

- 예시: 데이터베이스의 행은 임의 개수의 열을 가질 수 있다 : 모든 열을 타입 안전하게 이용할 수 있다면 좋을 것이다
  - 해법: 컨테이너 대신 키를 매개변수화, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다
  - 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장한다

#### 타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern)

```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objacts.requireNonNull(type), instance);
    }
    public <T> T getFavorite(Class<T> type){
        return type.cast(favorites.get(type));
    }
}
public static void main(String[] args){
    Favorites f = new Favorites();
    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, 0xcafebabe);
    f.putFavorite(Class.class, Favorites.class);
    
    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);
    
    System.out.printf("%s %x %s%n",favoriteString,favoriteInteger,favoriteClass.getName());
    // Java cafebabe Favorites
}
```

- favorites 인스턴스는 타입 안전하다

- 모든 키의 타입이 제각각이기 때문에, 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있다

- 위 클래스의 제약 두가지

  - 악의적인 클라이언트가 Class 객체를 제네릭이 아닌 로 타입으로 넘기면 Favorites 인스턴스의 타입 안전성이 쉽게 깨진다

    - 하지만 클라이언트 코드에서 컴파일 시 비검사 경고가 뜰 것이다.

    - HashSet과 HashMap 등 일반 컬렉션 구현체에도 똑같은 문제가 있다

    - Favorites가 타입 불변식을 어기는 일이 없도록 보장하려면 putFavorite 메서드에서 인수로 주어진 instance의 타입이 type으로 명시된 타입과 같은지 확인하면 된다

    - ```java
      public <T> void putFavorite(Class<T> type, T instance){
              favorites.put(Objacts.requireNonNull(type), type.cast(instance);
          }
      ```

    - java.util.Collections 에는 checkedList, CheckedMap, CheckedSet 같은 메서드가 있는데, 이 방식을 적용한 컬렉션 래퍼들이다.

    - 이 래퍼들은 제네릭과 로 타입을 섞어 사용하는 애플리케이션에서 클라이언트 코드가 컬렉션에 잘못된 타입의 원소를 넣지 못하도록 추적하는 데 도움이 된다

  - 실체화 불가 타입에는 사용할 수 없다

    - String, String[]은 저장 가능해도 `List<String>`은 저장할 수 없다
    - `List<String>`을 저장하려는 코드는 컴파일되지 않는다.
      - `List<String>`.class == 문법 오류

#### cast 메서드

- 형변환 연산자의 동적 버전

- 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지 검사

  - 맞다면 그 인수를 그대로 반환
  - 아니라면 ClassCastException 던짐

- cast 메서드가 단지 인수를 그대로 반환하기만 한다면, 굳이 사용하는 이유는?

  - cast 메서드의 시그니처가 Class 클래스가 제네릭이라는 이점을 완벽하게 활용하고 있다

  - ```java
    public class Class<T>{
        T cast(Object obj);
    }
    ```

  - T로 비검사 형변환하는 손실 없이도 Favorites 를 타입 안전하게 만든다