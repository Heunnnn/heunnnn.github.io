---
layout: post
title: 201226 자바 Collection Framework
tags:
  - Java
  - Collection
---

## 컬렉션 프레임워크의 전체적인 구성과 설계

수많은 코딩테스트 연습(ㅜㅜ)을 통해 컬렉션 프레임워크 내부 각 클래스들의 기능과 활용법은 이미 알고 있다.

'이펙티브 자바'를 공부하면서 그 구조와 설계 원리에 대한 좀더 깊은 이해가 필요할 듯 해, 

추가로 공부한 내용을 정리해 본다!



### 컬렉션 프레임워크 클래스 다이어그램

![출처: 생활코딩](https://prashantgaurav1.files.wordpress.com/2013/12/java-util-collection.gif)

Collection은 Iterable이라는 인터페이스를 상속받고 있다.

#### Iterable 인터페이스

- Iterable 인터페이스의 내용은 아주 간단하다

- ```java
  public interface Iterable<T>{
      Iterator<T> iterator();
      
      default void forEach(Consumer<? super T> action){
          ...
      }
      
      default Spliterator<T> spliterator(){
  		...
      }
  }
  ```

- Iterator (반복자) 필드를 가지고 있어, 요소를 순회할 수 있다.

- ```java
  public interface Iterator<E>{
      boolean hasNext();
      // 반복자 뒤에 요소가 남아있는 경우 true
      E next();
      // 반복자의 다음 요소를 반환
      default void remove(){
          ...
      }// 반복자에서 마지막 요소를 삭제
      default void forEachRemaining(Consumer<? super E> action){
          ...
      }// 반복자 내부 모든 요소가 처리되거나 예외 발생시까지 각 요소에 대해 지정 작업 수행
  }
  ```

- Collection 인터페이스는 Iterable을 모두 구현하고 있으므로,  반복자 제공이 강제되어 있다고 할 수 있다.

#### Collection과 AbstractCollection

- AbstractCollection은 Collection 인터페이스의 추상 골격 구현체이다(이펙티브 자바 [item 20](https://heunnnn.github.io/Effective-Java-4%EC%9E%A5-%EC%A0%95%EB%A6%AC-(1)/) 참고!)

- 공식 문서의 AbstractCollection 클래스 설명을 보자

- > This class provides a skeletal implementation of the `Collection` interface, to minimize the effort required to implement this interface.

  이 클래스는 Collection 인터페이스의 추상 골격 구현체를 제공하여 이 인터페이스를 구현하는 데 필요한 작업량을 최소한으로 줄입니다.

  > To implement an unmodifiable collection, the programmer needs only to extend this class and provide implementations for the `iterator` and `size` methods. (The iterator returned by the `iterator` method must implement `hasNext` and `next`.)

  불변 컬렉션을 구현하려면, 프로그래머는 이 클래스를 상속하여 iterator 메서드, size 메서드 구현을 제공합니다.(iterator 메서드가 반환하는 반복자는 반드시 hasNext와 next를 반환해야 합니다)

  > To implement a modifiable collection, the programmer must additionally override this class's `add` method (which otherwise throws an `UnsupportedOperationException`), and the iterator returned by the `iterator` method must additionally implement its `remove` method.

  가변 컬렉션을 구현하려면, 프로그래머는 추가적으로 이 클래스의 add 메서드를 오버라이드 해야 하며(오버라이드 하지 않은 경우 UnsupportedOperationException을 던집니다), itertor 메서드가 반환하는 반복자는 반드시 remove 메서드를 구현해야 합니다.

  > The programmer should generally provide a void (no argument) and `Collection` constructor, as per the recommendation in the `Collection` interface specification.

  프로그래머는 Collection 인터페이스 명세에서 추천되는것과 같이, 인수 없는 생성자와 Collection 생성자를 제공해야 합니다.

  > 이하 생략...

- AbstractSet과 AbstractList는 AbstractCollection을 상속받아 구현하고 있다.

- 따라서 이 인터페이스와 추상클래스들의 최종 구현체인 ArrayList에 대해, 다음과 같이 모두 사용할 수 있다!

  - ```java
    Collection<String> list = new ArrayLst<>(); 			
    AbstractCollection<String> list = new ArrayList<>(); 	
    List<String> list = new ArrayList<>(); 					
    AbstractList<String> list = new ArrayList<>();			
    ArrayList<String> list = new ArrayList<>(); 			
    ```
-----

##### 출처

Do it! 자바 프로그래밍

[생활코딩 'Collection Framework'](https://opentutorials.org/course/1223/6446)

[자바8 공식문서 'Class AbstractCollection'](https://docs.oracle.com/javase/8/docs/api/java/util/AbstractCollection.html)