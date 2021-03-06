---

layout: post
title: 210123 Effective Java 9장 정리
tags:
  - Effective Java
---

## 9. 일반적인 프로그래밍 방식

### [item 57] 지역변수의 범위를 최소화하라

- 지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고, 오류 가능성은 낮아짐
- C 등의 언어에서 지역변수를 코드변수의 첫 머리에 선언함
  - 이 방식을 습관처럼 따름
  - 자바에서는 문장을 선언할 수 있는 곳이면 어디서든 변수 선언 가능

#### 지역변수 범위를 줄이는 방법 1. 가장 처음 쓰일 때 선언한다

- 지역변수의 범위: 선언된 지점~ 그 지점을 포함한 블록 끝
  - 실제 사용하는 블록 바깥에 선언한 변수는 그 블록이 끝나도 살아있게 된다
  - 실수로 의도한 범위의 앞이나 뒤에서 그 변수를 사용하면 나쁜 결과로 이어짐

#### 2. 거의 모든 지역변수는 선언과 동시에 초기화해야 한다

- 초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미룬다
- 예외: try-catch
  - 변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록 안에서 초기화
    - 그렇지 않으면 예외가 블록을 넘어 메서드에까지 전파
  - 변수 값을 try 블록 바깥에서도 사용해야 한다면, try블록 앞에서 선언

##### while문보다 for문 사용을 권장

```java
Iterator <Element> i = c.iterator();
while (i.hasNext()){
	doSomething(i.next());
}
...
/*
	복사-붙여넣기 시 새로운 반복 변수 i2를 초기화했지만, 
	실수로 while 문 내에서 이전 while문의 i를 사용했다.
	i의 유효 범위는 끝나지 않았기 때문에 컴파일/실행까지 되지만, c2를 순회하지 않고 바로 끝나게 된다
*/
Iterator <Element> i2 = c2.iterator();
while (i.hasNext()){
	doSomething(i.next());
}
/* 
	for문을 사용하면 위와 같은 복사-붙여넣기 오류를 컴파일타임에 잡아준다
	(첫 번째 반복문이 사용한 원소와 반복자의 유효 범위, 변수의 유효 범위가 반복문 종료와 함께 끝난다)
	(따라서 똑같은 이름의 변수를 여러 반복문에서 써도 서로 아무런 영향이 없다)
*/
for(Iterator <Element> i = c.iterator(); i.hasNext();) {
	Element e = i.next();
	...
}
// 다음 코드는 "i를 찾을수 없다" 는 컴파일 오류를 낸다
for(Iterator <Element> i2 = c2.iterator(); i.hasNext();) {
	Element e = i.next();
	...
}
```

- for 형태든 for-each 형태든, 반복문에서는 반복 변수의 범위가 반복문의 몸체, 그리고 for 키워드와 몸체 사이의 괄호 안으로 제한
- while문은 반복변수의 값을 반복문 종료 이후에도 사용하는 경우 아니라면 사용 지양

##### 예시) 컬렉션/배열 순회 권장 관용구

```java
for( Element e : c){
	... // e로 무언가를 한다
}
```

##### 예시) 반복자가 필요할 때의 관용구

```java
for(Iterator<Element> i = c.iterator(); i.hasNext();){
	Element e = i.next();
	... // e와 i로 무언가를 한다
}
```

##### 예시) 또 다른 반복문 관용구

```java
for( int i=0, n= expensiveComputation(); i<n; i++){
... // i로 무언가를 한다
}
```

- i와 n 두 반복 변수의 범위가 정확히 일치한다
  - 반복 여부를 결정짓는 변수 i의 한계값을 변수 n에 저장하여, 반복때마다 다시 계산해야 하는 비용을 없앤다
  - 같은 값을 반환하는 메서드를 매번 호출한다면 이 관용구를 사용한다

#### 3. 메서드를 작게 유지하고, 한 가지 기능에 집중한다

- 한 메서드에서 여러 기능을 처리한다면
  - 그중 한 기능과만 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근 가능
  - 메서드를 기능별로 쪼갠다

---

### [item 58] 전통적인 for 문보다는 for-each 문을 사용하라

#### for문의 단점

```java
// 컬렉션 순회
for(Iterator <Element> i = c.iterator(); i.hasNext();) {
	Element e = i.next();
	...
}
// 배열 순회
for(int i=0;i<a.length;i++){
	...
}
```

- 우리에게 필요한 것은 원소 뿐이다 - 반복자와 인덱스 변수는 모두 코드를 지저분하게 한다
- 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다
  - 1회 반복에서 반복자는 3번, 인덱스는 4번 등장하면서 변수를 잘못 사용할 틈새가 넓어진다
- 잘못된 변수를 사용했을 때 컴파일러가 잡아주리라는 보장도 없다
- 컬렉션이냐 배열이냐에 따라 사용하는 코드 형태가 상당히 달라진다

#### for-each문

```java
// elements 안의 각 원소 e에 대해
for (Element e : elements) {
	...// e로 무언가를 한다
}
```

- 정식 명칭: 향상된 for문(enhanced for statement)
- Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다
  - 원소들의 묶음을 표현하는 타입을 작성해야 한다면, Collection 인터페이스는 구현하지 않더라도 Iterable은 구현하는 쪽으로 고민하자
- 반복자와 인덱스 변수를 사용하지 않아 코드가 깔끔하고, 오류가 날 일도 없다
- 하나의 관용구로 컬렉션과 배열 모두를 순회 가능하므로, 어떤 컨테이너를 다루는지 신경쓰지 않아도 된다

#### 예시) 컬렉션 중첩 순회시 for-each문의 이점

```java
enum Suit {	CLUT, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }
...
static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext();)
	for(Iterator<Rank> j = ranks.iterator(); j.hasNext();)
		deck.add(new Card(i.next(), j.next()));
```

##### 문제점

- 바깥 컬렉션(suits)의 반복자에서 next 메서드가 너무 많이 불린다
- 마지막 줄의 i.next()
  - '숫자'하나당 한 번씩만 불려야 하는데, 안쪽 반복문에서 호출되면서 '카드' 하나당 한 번씩 불리고 있다
  - 숫자가 바닥나면 NoSuchElementException을 던진다
    - 바깥 컬렉션 크기=안쪽 컬렉션 크기의 배수라면 이 반복문은 예외를 던지지 않고 종료한다

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(Face.class);

for(Iterator<Face> i = faces.iterator(); i.hasNext(); )
	for(Iterator<Face> j = faces.iterator(); j.hasNext(); )
		System.out.println(i.next() + " " + j.next() );
```

- 예외를 던지진 않지만, 가능한 조합을 { "ONE", "ONE" } 부터 {"SIX", "SIX"} 까지 단 여섯쌍만 출력 후 끝난다
  - 예상 결과는 6*6=36개 조합이다

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
...
Collection<Face> faces = EnumSet.allOf(Face.class);

for(Iterator<Face> i = faces.iterator(); i.hasNext(); )
	Suit suit = i.next();
	for(Iterator<Face> j = faces.iterator(); j.hasNext(); )
		System.out.println(suit + " " + j.next() );
```

- 바깥 반복문에 바깥 원소를 저장하는 변수를 하나 추가한다
  - 하지만 가독성이 나쁘다

```java
for (Suit suit: suits)
	for (Rank rank : ranks)
		deck.add(new Card(suit, rank));
```

- for-each문을 중첩하면 깔끔해진다

#### for-each문을 사용할 수 없는 경우

##### 파괴적인 필터링(destructive filtering)

- 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다
- 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다

##### 변형(transforming)

- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다

##### 병렬 반복(parallel iteration)

- 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해야 한다

---

### [item 59] 라이브러리를 익히고 사용하라

#### 예시) random 메서드 작성

##### 가장 흔하게 작성하는 random 메서드

```java
static Random rnd = new Random();

static int random(int n) {
	return Math.abs(rnd.nextInt()) % n;
```

- 문제점
  - n이 그다지 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다
  - n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다
    - n값이 클수록 이 현상이 더 두드러진다
    - random 메서드는 무작위로 생성된 수 중 2/3이 중간값보다 낮은 쪽으로 쏠린다
  - 지정한 범위 '바깥'의 수가 종종 튀어나올 수 있다
    - rnd.nextInt()가 반환한 값을 Math.abs()를 이용해 음수가 아닌 수로 매핑한다
      - n이 2의 제곱수가 아닐 때, nextInt()가 Integer.MIN_VALUE를 반환하면 Math.abs()도 Integer.MIN_VALUE를 반환, 나머지 연산자는 음수를 반환한다
- 모든 문제는 [Random.nextInt(int)](https://docs.oracle.com/javase/8/docs/api/java/util/Random.html#nextInt-int-)를 사용하면 해결된다
- 자바 7부터는 ThreadLocalRandom으로 대체하는 것이 좋다
  - 더 고품질이고 속도도 빠르다
  - 포크-조인 풀이나 병렬 스트림에서는 SplittableRandom을 사용한다

#### 표준 라이브러리 사용시 장점

##### 코드를 작성한 전문가의 지식과, 다른 프로그래머들의 경험을 활용할 수 있다

##### 핵심적인 일과 크게 관련없는 일을 해결하느라 시간을 허비하지 않아도 된다

##### 따로 노력하지 않아도 성능이 지속해서 개선된다

- 사용자가 많고, 업계 표준 벤치마크를 사용해 성능을 확인한다

##### 기능이 점점 많아진다

##### 작성한 코드가 많은 사람들에게 낯익은 코드가 된다

#### 알아두면 좋을 표준 라이브러리

##### java.lang

##### java.util

- java.util.concurrent

##### java.io

---

### [item 60] 정확한 답이 필요하다면 float과 double은 피하라

#### float, double 타입은 과학/공학 계산용으로 설계되었다

- 이진 부동소수점 연산
- 넓은 범위의 수를 빠르고 정밀한 '근사치'로 계산하도록 설계되었다
- 따라서 금융 등의 정확한 계산이 필요한 계산과는 맞지 않다
  - 0.1 혹은 10의 음의 거듭제곱 수를 표현할 수 없다

#### 예제: float과 double 계산식

```java
System.out.println(1.03-0.42);
// 예상결과: 0.61 / 실제결과: 0.610000000000001
System.out.println(1.00-9*0.10)
// 예상결과: 0.1 / 실제결과: 0.0999999999999998
```

#### 금융 계산에는 BigDecimal, int 혹은 long을 사용하라

```java
public static void main(String[] args) {
    final bigDecimal TEN_CENTS = new BigDecimal(".10");
    
    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for(BigDecimal price = TEN_CENTS;
       funds.compareTo(price)>=0;
       funds=funds.add(TEN_CENTS)){
        funds=funds.substract(price);
        itemsBought++;
    }
    System.out.println(itemsBought +"개 구입");
    System.out.println("잔돈(달러): "+funds);
}
```

##### BigDecimal의 단점

- 기본 타입보다 쓰기 불편하고 훨씬 느리다
- 대안: int, long 타입을 사용?
  - 값의 크기가 제한되며, 소수점 아래 값을 관리해야 한다
  - 위의 계산은 달러 대신 센트로 단위를 바꿔 계산하면 int로 변환 가능하다

---

### [item 61] 박싱된 기본 타입보다는 기본 타입을 사용하라

#### 기본 타입과 박싱 타입

- 각각의 기본 타입에는 대응하는 참조 타입이 하나씩 있다

##### 기본 타입과 박싱 타입의 차이점

- 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값과 식별성(identity)을 가진다
  - 박싱된 기본 타입의 두 인스턴스는 값이 같아도 서로 다르다고 식별될 수 있다
- 기본 타입의 값은 언제나 유효하나, 박싱된 타입은 유효하지 않은 값(null)을 가질 수 있다
- 기본 타입이 박싱된 기본 타입보다 메모리 사용 면에서 더 효율적이다

##### 박싱된 기본 타입에 ==연산자를 사용 시 오류가 일어난다

- 박싱된 값을 비교할때 ==연산자를 사용한다면, 객체 참조의 식별성을 검사하게 된다
- 따라서 박싱된 기본 타입은 기본 타입 정수 지역 변수로 저장후 사용한다.

##### 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다

- null 참조를 언박싱하면 NullPointerException 발생

##### 박싱된 기본 타입을 사용해야 하는 때

- 컬렉션의 원소,키,값으로 사용
- 매개변수화 타입이나 매개변수화 메서드의 타입 변수
- 리플렉션을 통한 메서드 호출

---

### [item 62] 다른 타입이 적절하다면 문자열 사용을 피하라

#### 문자열은 열거 타입을 대신하기에 적합하지 않다

- 상수를 열거할때는 열거 타입 사용

#### 문자열은 혼합 타입을 대신하기에 적합하지 않다

- 여러 요소가 혼합된 데이터를 하나의 문자열로 표현하지 말라
- 구분자를 통해 구분,사용?
  - 문자열을 파싱해야 하기 때문에 느리고,귀찮고,오류 가능성이 크다
  - 적절한 equals,toString,compareTo 메서드를 제공할 수 없다
- private 정적 멤버 클래스로 선언하라

#### 문자열은 권한을 표현하기에 적합하지 않다

- 예상과 다른 동작, 보안 취약

---

### [item 63] 문자열 연결은 느리니 주의하라 

#### 문자열 연결 연산자(+)로 문자열 n개를 잇는 시간은 n^2에 비례한다

```java
pubilc String statement() {
    String result = "";
    for(int i=0;i<numItems(); i++)
        result+=lentForItem(i);
    return result;
}
```

- 양쪽 내용을 모두 복사해야 해서 성능이 저하됨

#### String 대신 StringBuilder를 사용하면 성능이 개선된다

```java
public String statement2() {
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for(int i=0;i<numItems(); i++)
        b.append(lineForItem(i));
    return b.toString();
}
```

- 위 연산보다 5.5배~6.5배 빠르다
- 문자 배열을 사용하거나, 문자열을 연결하지 않고 하나씩 처리하는 방법도 고려한다.

---

### [item 64] 객체는 인터페이스를 사용해 참조하라 

#### 적합한 인터페이스가 있다면 매개변수 뿐 아니라 반환값,변수,필드를 전부 인터페이스 타입으로 선언하라

- 객체의 실제 클래스를 사용해야 할 상황은 '오직' 생성자로 생성할 때 뿐
- 프로그램이 훨씬 유연해진다

#### 주의 사항

- 원래의 클래스가 인터페이스의 일반 규약 이외 특별한 가능을 제공하며, 주변 코드가 이 기능에 기대어 동작한다면 클래스도 같은 기능 제공 필요
- 적합한 인터페이스가 없다면 당연히 클래스로 참조해야 한다
  - 클래스 계층구조 중 필요한 기능을 만족하는 가장 덜 구체적인(상위의) 클래스를 타입으로 사용한다
  - String, BigInteger같은 값 클래스
  - 클래스 기반으로 작성된 프레임워크가 제공하는 객체들
  - 인터페이스에는 없는 특별한 메서드를 제공하는 클래스들

---

### [item 65] 리플렉션보다는 인터페이스를 사용하라

#### 리플렉션(java.lang.reflect)

- 프로그램에서 임의의 클래스에 접근 가능
- 컴파일 당시 존재하지 않던 클래스도 이용할 수 있다
- Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있고, 이 인스턴스들로 클래스 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다
- 인스턴스를 이용, 각각에 연결된 실제 생성자,메서드,필드를 조작할 수 있다
  - 예시: Method.invoke
    - 어떤 클래스의 어떤 객체가 가진 어떤 메서드라도 호출할 수 있게 해준다

#### 리플렉션 이용시 단점

- 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다
  - 예외 검사 등... 
  - 존재하지 않거나 접근 불가능한 메서드를 호출하려 하면 런타임 오류 발생
- 리플렉션을 이용하면 코드가 지저분해지고 장황해진다
- 성능이 떨어진다
  - 리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다

#### 리플렉션은 아주 제한된 형태로만 사용한다

- 리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자
- 런타임에 존재하지 않을 수도 있는 다른 클래스,메서드,필드와의 의존성을 관리할 때 적합하다
  - 버전이 여러 개 존재하는 외부 패키지를 다룰 때 유용함
    - 가장 오래 된 버전만을 지원하도록 컴파일한 후, 이후 버전의 클래스와 메서드 등은 리플렉션으로 접근
    - 새로운 클래스나 메서드가 런타임에 반드시 존재하지 않을 수 있다는 사실을 반드시 감안해야 한다

---

### [item 66] 네이티브 메서드는 신중히 사용하라

#### 자바 네이티브 인터페이스(JNI, Java Native Interface)

- 자바 프로그램이 네이티브 메서드를 호출하는 기술
  - 네이티브 메서드: C/C++같은 네이티브 프로그래밍 언어로 작성한 메서드

##### 쓰임새

- 레지스트리 같은 플랫폼 특화 기능
- 네이티브 코드로 작성된 기존 라이브러리를 사용
  - 레거시 데이터를 사용하는 레거시 라이브러리
- 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역만 따로 네이티브 언어로 작성

##### 단점

- 네이티브 언어가 안전하지 않으므로, 네이티브 메서드를 사용하는 애플리케이션도 메모리 훼손 오류로부터 더이상 안전하지 않다.
- 자바보다 플랫폼을 많이 타기 때문에 이식성이 낮고, 디버깅도 어렵다
- 주의하지 않으면 속도가 오히려 느려진다
  - 자바 코드와 네이티브 코드의 경계를 넘나들 떄마다 비용이 추가된다
- 가비지 컬렉터가 네이티브 메모리는 자동 회수하지 못하고, 심지어 추적도 할 수 없다
- 네이티브 메서드와 자바 코드 사이의 '접착 코드(glue code)'를 작성해야 하는데, 이는 귀찮은 작업이고 가독성도 떨어진다.

#### 성능 개선 목적으로 네이티브 메서드 사용은 거의 권장하지 않는다

- 자바 초기 시절(자바3 이전)이라면 이야기가 다르지만, 대부분의 작업에서 지금의 자바는 다른 플랫폼에 견줄만한 성능을 보인다
- 하지만 GNU 다중 정밀 연산 라이브러리(GMP)등을 이용한 고성능의 다중 정밀 연산이 필요한 자바 프로그래머라면, 네이티브 메서드를 통해 GMP를 사용하는 것을 고려하라

---

### [item 67] 최적화는 신중히 하라

#### 최적화 관련 격언

> (맹목적인 어리석음을 포함해) 그 어떤 핑계보다 효율성이라는 이름 아래 행해진 컴퓨팅 죄악이 더 많다(심지어 효율을 높이지도 못하면서). - 윌리엄 울프

> (전체의 97% 정도인) 자그마한 효율성은 모두 잊자. 섣부른 최적화가 만악의 근원이다. - 도널드 크누스

>최적화를 할 떄는 다음 두 규칙을 따르라.
>
>첫 번째, 하지 마라.
>
>두 번째, (전문가 한정) 아직 하지 마라. 다시 말해, 완전히 명백하고 최적화되지 않은 해법을 찾을 때까지는 하지 마라.

#### 빠른 프로그램보다는 좋은 프로그램을 작성하라

- 최적화는 좋은 결과보다는 해로운 결과로 이어지기 쉽고, 섣불리 진행하면 더 그렇다.
- 좋은 프로그램은 정보 은닉 원칙을 따르므로 개별 구성요소의 내부를 독립적으로 설계할 수 있다
  - 시스템의 나머지에 영향을 주지 않고도 각 요소를 다시 설계할 수 있다

#### 설계 단계에서 성능을 고려하는 방법

- 구현상의 문제는 나중에 최적화를 통해 해결 가능하나, 아키텍처의 결함이 성능을 제한하는 상황에서는 시스템 전체를 다시 작성해야 한다

##### 성능을 제한하는 설계를 피하라

- 컴포넌트끼리/외부 시스템과의 소통 방식같은 설계 요소는 완성후 변경이 어렵거나 불가능할 수 있으며, 동시에 시스템 성능을 심각하게 제한할 수 있다

##### API 설계시 성능에 주는 영향을 고려한다

- public 타입을 가변으로 만들면(내부 데이터를 변경할 수 있게 만들면) 불필요한 방어적 복사를 수없이 유발할 수 있다
- 하지만 성능을 위해 API를 왜곡하는 것은 매우 안 좋은 생각이다

##### 각각의 최적화 시도 전후로 성능을 측정하라

- 프로파일링 도구
- jmh
  - 자바 코드의 상세한 성능을 알기 쉽게 보여주는 마이크로 벤치마킹 프레임워크


---

### [item 68] 일반적으로 통용되는 명명 규칙을 따르라

- 자바 플랫폼은 명명 규칙이 잘 정립되어 있으며, 그중 많은 것이 자바 언어 명세에 기술되어 있다
- 철자와 문법, 두 범주로 나눈 수 있다

#### 철자 규칙

- 클래스,인터페이스,메서드,필드,타입 변수의 이름을 다룬다

| 식별자 타입         | 예                                                   |
| ------------------- | ---------------------------------------------------- |
| 패키지와 모듈       | org.junit.jupiter.api<br />com.google.common.collect |
| 클래스와 인터페이스 | Stream,FutureTask,LinkedHashMap                      |
| 메서드와 필드       | remove,groupingBy,getCrc                             |
| 상수 필드           | MIN_VALUE, NEGATIVE_INFINITY                         |
| 지역 변수           | i, demon, houseNum                                   |
| 타입 매개변수       | T, E, K, V, X, R, U, V, T1, T2                       |

#### 문법 규칙

- 더 유연하고, 논란도 많다
- 객체를 생성할 수 있는 클래스(열거 타입 포함)의 이름은 보통 단수 명사나 명사구 사용
- 객체를 생성할 수 없는 클래스의 이름은 보통 복수형 명사
- 인터페이스 이름은 클래스와 똑같이 짓거나 able, ible로 끝나는 현용사
- 애너테이션은 지배적 규칙 없음
- 어떤 동작을 수행하는 메서드의 이름은 동사나(목적어를 포함한) 동사구로 짓는다
- boolean값을 반환하는 메서드라면 보통 is나 has로 시작하고, 명사나 명사구 형용사로 기능하는 아무 단어나 구로 끝나도록 짓는다
- 반환 타입이 boolean이 아니거나 해당 인스턴스의 속성을 반환하는 메서드의 이름은 보통 명사, 명사구 혹은 get으로 시작하는 동사구로 짓는다

##### 꼭 알아뒤야 할 특별한 메서드 이름

- 객체의 타입을 바꿔서, 다른 타입의 또 다른 객체를 반환하는 인스턴스 메서드의 이름은 보통 toType 형태로 짓는다
  - toString, toArray
- 객체의 내용을 다른 뷰로 보여주는 메서드의 이름은 asType 형태로 짓는다
  - asList
- 객체의 값을 기본 타입 값으로 반환하는 메서드의 이름은 보통 typeValue 형태로 짓는다
  - intValue
- 정적 팩터리의 이름은 다양하지만 from,of,valueOf,instance,getInstance,newInstance,getType,newType을 흔하게 사용한다. 