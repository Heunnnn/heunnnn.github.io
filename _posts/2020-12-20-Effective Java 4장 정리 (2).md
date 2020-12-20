---
layout: post
title: 201220 Effective Java 4장 정리 (2)
tags:
  - Effective Java
---

## 4. 클래스와 인터페이스

### [item21] 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8 이전: 인터페이스에 메서드 추가시 컴파일 오류 발생.

모든 클래스가 `현재의 인터페이스에 새로운 메서드가 추가될 일은 영원히 없다`고 가정하고 작성되었다.

자바 8 이후: 디폴트 메서드

#### 디폴트 메서드

- 그 인터페이스를 구현한 후 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰인다
- 구현 클래스에 대해 아무것도 모른 채 합의 없이 무작정 삽입되었다
- 자바 8 이후로 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가되었다

  - 주로 람다를 활용하기 위해서였다
  - 자바 라이브러리의 디폴트 메서드는 코드 품질이 높고 범용적이라 대부분 상황에서 잘 작동한다
- 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기는 어렵다
- 디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 런다임 오류를 일으킬 수 있다
- 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해야 한다
- 추가하려는 디폴트 메서드가 기존 구현체와 충돌하지는 않을지 고려해야 한다
- 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는 데 아주 유용한 수단이다
- 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아니다

#### 예시: Collection 인터페이스에 추가된 removeIf

 주어진 불리언 함수(predicate)가 true를 반환하는 모든 원소를 제거한다

디폴트 구현: 반복자를 이용해 순회하면서 각 원소를 인수로 넣어 프레디키트 호출, 프레디키트가 true 반환시 반복자의 remove 메서드 호출, 운소 제거

```java
default boolean removeIf(Predicate<? super E> filter){
    Objects.requireNonNull(filter);
    boolean result = false;
    for(Iterator<E> it=iterator(); it.hasNext();){
        if(filter.test(it.next())){
            it.remove();
            result.true;
        }
    }
    return result;
}
```

- 현존하는 모든 Collection 구현체와 잘 어우러지는 것은 아니다
  - 예시: 아파치의 SynchronizedCollection 클래스
    - 이 클래스를 자바 8과 함께 사용할 수 없다
    - 특히 여러 스레드가 SynchronizedCollection 인스턴스를 공유하는 환경에서 한 스레드가 removeIf를 호출하면 ConcurrentModificationException이 발생하거나 다른 예기치 못한 오류가 발생한다

-----

### [item22] 인터페이스는 타입을 정의하는 용도로만 사용하라

#### 예시: 상수 인터페이스

```java
// 상수 인터페이스 안티패턴: 사용 금지!
public interface PhysicalConstants{
    // 아보가드로 상수(1/몰)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    ...
}
```

- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다
- 상수 인터페이스를 구현하는 것: 이 내부 구현을 클래스의 API로 노출하는 행위

#### 상수를 공개하는 방법

- 특정 클래스나 인터페이스와 강하게 연관된 상수: 그 클래스나 인터페이스 자체에 추가한다

  - 예시: 모든 숫자 기본 타입의 박싱 클래스(Integer, Double의 MIN_VALUE,MAX_VALUE)

- 열거 타입으로 나타내기 적합한 상수: 열거 타입으로 공개

- 인스턴스화할 수 없는 유틸리티 클래스에 담아 공개한다

  - ```java
    package constantutilityclass;
    
    // 상수 유틸리티 클래스
    public class PhysicalConstants{
        private PhysicalConstants(){}// 인스턴스화 방지
        // 아보가드로 상수(1/몰)
        public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
        ...
    }
    ```

  - 유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함께 명시해야 한다

  - 유틸리티 클래스의 상수를 빈번히 사용한다면 정적 임포트(static import)하여 클래스 이름을 생략할 수 있다

  - ```java
    import static constantutilityclass.PhysicalConstants.*;
    public class Test{
        double atoms(double mols){
            return AVOGADROS_NUMBER*mols;
        }
        ...
    }
    ```
-----

### [item23] 태그 달린 클래스보다는 클래스 계층구조를 활용하라

#### 태그 달린 클래스의 단점

- 열거 타입 선언, 태그 필드, switch문 등 쓸데없는 코드가 많다
- 여러 구현이 한 클래스에 혼합되어 가독성도 나쁘다
- 다른 의미를 위한 코드도 언제나 함께 해 메모리도 많이 사용한다
- 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다
- 인스턴스의 타입만으로는 현재 나타나는 의미를 알 길이 전혀 없다
- 태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류이다.

#### 태그 달린 클래스->클래스 계층 구조

- 계층구조의 root가 될 추상 클래스 정의
- 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다
- 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다
- 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다
- 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다
- 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다

-----

### [item24] 멤버 클래스는 되도록 static으로 만들라

```
멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자
```

#### 중첩 클래스(nested class)

- 다른 클래스 안에 정의된 클래스
- 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다
- 중첩 클래스의 종류
  - 정적 멤버 클래스
  - (비정적) 멤버 클래스
  - 익명 클래스
  - 지역 클래스

#### 정적 멤버 클래스

- 다른 클래스 안에 선언된다
- 바깥 클래스의 private 멤버에도 접근할 수 있다
- 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다
- 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다
- 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재 가능하다면 정적 멤버 클래스로 만들어야 한다

#### 비정적 멤버 클래스

- 바깥 클래스의 인스턴스와 암묵적으로 연결된다
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다
  - 정규화된 this: 클래스명.this 형태로 바깥 클래스의 이름을 명시
- 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될때 확립되며, 더이상 변경할 수 없다.
- 어댑터를 정의할 때 자주 쓰인다
  - 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용
  - 예시: Map 인터페이스의 구현체들은 자신의 컬렉션 뷰를 구현할 때 비정적 멤버 클래스를 사용한다.

#### 익명 클래스

- 이름이 없으며, 바깥 클래스의 멤버도 아니다
- 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다
- instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다
- 비정적인 문맥에서 사용될때만 바깥 클래스의 인스턴스를 참조할 수 있다
  - 상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다.
- 작은 함수 객체나 처리 객체를 만드는 데 이용
- 정적 팩터리 메서드를 만드는 데 이용
  - 자바가 람다를 지원하면서 사용이 줄었다

#### 지역 클래스

- 가장 드물게 사용
- 지역변수를 선언할 수 있는 곳이면 실질적으로 어디든 선언 가능
- 유효 범위도 지역변수와 같다
- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다
- 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다
- 정적 멤버는 가질 수 없다
- 가독성을 위해 짧게 작성해야 한다

-----

### [item25] 톱레벨 클래스는 한 파일에 하나만 담으라

#### 예시

Main 클래스는 다른 톱레벨 클래스 Utensil과 Dessert 를 참조한다

`Main.java`

```java
public class Main{
    public static void main(String[] args){
        System.out.println(Utensil.NAME+Dessert.NAME);
    }
}
```

두 클래스가 `Utensil.java` 한 파일에 정의되었다

```java
class Utensil{
    static final String NAME="pan";
}
class Dessert{ 
	static final String NAME="cake";
}
```

우연히 똑같은 두 클래스를 담은 `Dessert.java`라는 파일을 만들었다

```java
class Utensil{
    static final String NAME="pan";
}
class Dessert{ 
	static final String NAME="cake";
}
```

운좋게 `javac Main.java Dessert.java`로 컴파일하면 컴파일 오류가 나고, 클래스 중복 정의를 알려준다

- 가장 먼저 Main.java 컴파일
- 그 안에서 Utensil 참조를 만나면 Utensil.java파일에서 Utensil과 Dessert를 모두 찾아낸다
- 그 다름 컴파일러가 두번째 명령줄 인수로 넘어온 Dessert.java를 처리하려 할 떄 같은 클래스의 정의가 이미 있음을 알게 된다

그러나 `javac Main.java` 또는 `javac Main.java Utensil.java`로 컴파일하면 컴파일 오류가 나지 않고,  `pancake`를 출력한다

#### 해결책

1.톱레벨 클래스를 서로 다른 소스파일로 분리한다

2.정적 멤버 클래스를 사용한다

- 다른 클래스에 딸린 부차적인 클래스라면 정적 멤버 클래스로 만드는 것이 낫다

  - 가독성이 좋다
  - private로 선언시 접근 범위도 최소로 관리 가능하다

- ```java
  public class Test{
      public static void main(String[] args){
          System.out.println(Utensil.NAME+Dessert.NAME);
      }
      private static class Utensil{
      	static final String NAME="pan";
  	}
  	private static class Dessert{ 
  		static final String NAME="cake";
  	}
  }
  ```

- 