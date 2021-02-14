---

layout: post
title: 210214 Effective Java 12장 정리
tags:
  - Effective Java
---

## 12. 직렬화

- 객체 직렬화
  - 자바가 객체를 바이트 스트림으로 인코딩하고(직렬화)
  - 그 바이트 스트림으로부터 다시 객체를 재구성(역직렬화)
  - 직렬화된 객체는 다른 VM에 전송하거나, 디스크에 저장한 후 나중에 역직렬화할 수 있다

### [item85] 자바 직렬화의 대안을 찾으라

#### 직렬화의 보안 문제

##### 공격 범위가 너무 넓고, 지속적으로 더 넓어지고 있다

- ObjectInputStream의 readObject 메서드를 호출하면서 객체 그래프가 역직렬화된다
  - readObject 메서드는(Serializable 인터페이스를 구현했다면) 클래스패스 안의 거의 모든 타입의 객체를 만들어낼 수 있는 생성자
  - 바이트 스트림을 역직렬화하는 과정에서 이 메서드는 그 타입들 안의 모든 코드를 수행할 수 있다
    - 해당 타입들의 코드 전체가 공격 범위에 들어간다
    - 자바 표준 라이브러리, 아파치 커먼즈 컬렉션, 애플리케이션 자신의 클래스들....
- 신뢰할 수 없는 스트림을 역직렬화하면 원격 코드 실행, 서비스 거부 등의 공격으로 이어질 수 있다

##### 가젯(gadget)

- 역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드
- 여러 가젯을 함께 사용하여 가젯 체인 구성 가능
  - 공격자가 기반 하드웨어의 네이티브 코드를 마음대로 실행할 수 있는 아주 강력한 가젯 체인이 발견되기도 함

#### 역직렬화 폭탄

```java
static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    
    for(int i=0;i<100;i++){
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo");
        s1.add(t1); s1.add(t2);
        s2.add(t1); s2.add(t2);
        s1=t1;
        s2=t2;
    }
    return serialize(root);
}
```

- 역직렬화에 시간이 오래 걸리는 짧은 스트림을 역직렬화하는 것으로 서비스 거부 공격에 노출
- 위 코드의 객체 그래프는 201개의 HashSet 인스턴스로 구성, 각각은 3개 이하의 객체 참조를 가짐
  - HashSet 인스턴스를 역직렬화하려면 그 원소들의 해시코드를 계산해야 한다
  - hashCode 메서드를 2^100번 넘게 호출해야 한다
  - 이 코드는 단 몇개의 객체만 생성해도 스택 깊이 제한에 걸리게 된다

#### 직렬화 위험을 회피하는 가장 좋은 방법은 아무것도 역직렬화하지 않는 것읻다

- 자바 직렬화를 써야 하는 이유는 거의 없다
  - 객체<->바이트 시퀀스를 변환해주는 다른 매커니즘이 많이 있다
- 신뢰할 수 없는 데이터는 절대 역직렬화하지 않아야 한다
- 객체 역직렬화 필터링 사용(java.io.ObjectInputFilter, 자바9)
  - 데이터 스트림이 역직렬화 되기 전에 필터를 설치
  - 클래스 단위로 특정 클래스를 받아들이거나 거부
    - '기본 수용' : 블랙리스트에 기록된 잠재적으로 위험한 클래스들을 거부한다
      - 이미 알려진 위험으로부터만 보호 가능하다
    - '기본 거부' : 화이트리스트에 기록된 안전하다고 알려진 클래스들만 수용(추천)
  - 메모리를 과하게 사용하거나 객체 그래프가 너무 깊어지는 사태로부터 보호한다
    - 직렬화 폭탄은 걸러낼 수 없다

##### 크로스-플랫폼 구조화된 데이터 표현

- 자바 직렬화보다 훨씬 간단하다
- 시스템을 밑바닥부터 설계한다면 이 대안을 사용해야 한다
- 임의 객체 그래프를 자동으로 직렬화/역직렬화하지 않는다
  - 속성-값 쌍의 집합으로 구성된 간단하고 구조화된 데이터 객체를 사용한다
  - 기본 타입 몇 개와 배열 타입만 지원한다
- JSON, 프로토콜 버퍼(Protocol Buffers / protobuf)
  - JSON
    - 브라우저와 서버의 통신용으로 설계
    - 자바스크립트용
    - 텍스트 기반이라 사람이 읽을 수 있다
  - [프로토콜 버퍼](https://developers.google.com/protocol-buffers)
    - C++
    - 구글이 서버 사이 데이터 교환,저장을 위해 설계
    - 이진 표현으로 효율이 높다
    - 문서를 위한 스키마(타입)을 제공, 올바로 쓰도록 강요
    - 사람이 읽을 수 있는 텍스트 표현(pbtxt)도 지원한다

### [item86] Serializable을 구현할지는 신중히 결정하라

- 어떤 클래스의 인스턴스를 직렬화?
  - 클래스 선언에 `implements Serializable`을 덧붙인다

#### Serializable을 구현하면 릴리스한 뒤에는 수정하기 어렵다

- 클래스가 Serializable을 구현하면 직렬화된 바이트 스트림 인코딩(직렬화 형태)도 하나의 공개 API가 된다
  - 이 클래스가 널리 퍼지는 경우 그 직렬화 형태도 영원히 지원해야 한다
  - 기본 직렬화 형태에서는 클래스의 private과 package-private 인스턴스 필드들마저 API로 공개되는 꼴이 된다(캡슐화가 깨진다)
- 클래스 내부 구현을 손보면 원래의 직렬화 형태와 달라지게 된다
  - 한쪽은 구버전 인스턴스를 직렬화, 다른 쪽은 신버전 클래스로 역직렬화한다면 실패를 맛보게 된다
    - ObjectOutputStream.putFields <-> ObjectInputStream.readFields
  - 직렬화 가능 클래스를 만들고자 한다면, 길게 보고 감당할 수 있을 만큼 고품질의 직렬화 형태도 주의해서 함께 설계해야 한다

#### Serializable을 구현하면 버그와 보안 구멍이 생길 위험이 높아진다

- 객체는 생성자를 사용해 만드는게 기본
  - 직렬화는 언어의 기본 매커니즘을 우회하는 객체 생성 기법
  - 역직렬화는 일반 생성자의 문제가 그대로 적용되는 '숨은 생성자'
  - 이 생성자는 전면에 드러나지 않는다
    - "생성자에서 구축한 불변식을 모두 보장해야 하고, 생성 도중 공격자가 객체 내부를 들여다볼 수 없도록 해야 한다"
    - 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다

#### Serializable을 구현하면 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다

- 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화할 수 있는지
  - 그 반대도 가능한지 검사
  - 테스트해야 할 양이 직렬화 가능 클래스의 수와 릴리스 횟수에 비례해 증가
- 양방향 직렬화, 역직렬화가 모두 성공하고, 원래의 객체를 충실히 복제해내는지 반드시 확인해야 한다

#### Serializable 구현 여부는 가볍게 결정할 사안이 아니다

- 객체를 전송하거나 저장할 때 자바 직렬화를 이용하는 프레임워크용으로 만든 클래스
- Serializable을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스
- '값' 클래스는 Serializable 구현, '동작' 클래스는 구현X

#### 상속용으로 설계된 클래스는 대부분 Serializable을 구현하면 안 되며, 인터페이스도 대부분 Serializable을 확장해서는 안 된다

- 클래스를 확장하거나, 그런 인터페이스를 구현하는 이에게 커다란 부담을 지우게 된다
- Serializable을 구현한 클래스만 지원하는 프레임워크를 사용하는 상황이라면 다른 방도가 없다

##### 인스턴스 필드가 직렬화와 확장이 모두 가능하다면

- 인스턴스 필드 값 중 불변식을 보장해야 할 게 있다면 반드시 하위 클래스에서 finalize 메서드를 자신이 재정의하면서 final로 선언하면 된다

  - finalizer 공격을 당할 수 있음

- 인스턴스 필드 중 기본값(정수형은 0, boolean은 false, 객체 참조 타입은 null)으로 초기화되면 위배되는 불변식이 있다면

  - 다음 readObjectNoData 메서드를 반드시 추가

  - ```java
    private void readObjectNoData() throws InvalidObjectException {
        throw new InvalidObjectException("스트림 데이터가 필요합니다");
    }
    ```

  - 기존 직렬화 가능 클래스에 직렬화 가능 상위 클래스를 추가하는 드문 경우를 위한 메서드

#### 내부 클래스는 직렬화를 구현하지 말아야 한다

- 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다
- 내부 클래스에 대한 기본 직렬화 형태는 분명하지 않다

### [item87] 커스텀 직렬화 형태를 고려해보라

#### 먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라

- 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다
- 객체의 물리적 표현 == 논리적 내용 => 기본 직렬화 형태도 무방

##### 기본 직렬화 형태가 적합하더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공

##### 객체의 물리적 표현과 논리적 표현이 다를 때

- 공개 API가 현재의 내부 표현 방식에 영구히 묶인다

- 너무 많은 공간을 차지할 수 있다
- 시간이 너무 많이 걸릴 수 있다
- 스택 오버플로를 일으킬 수 있다

##### 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략해야 한다

##### 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다

##### 어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자

### [item88] readObject 메서드는 방어적으로 작성하라

#### 필요하다면 매개변수를 방어적으로 복사하라

- readObject 메서드는 실질적으로는 또 다른 public 생성자이기 때문에 생성자와 똑같은 수준으로 주의를 기울여야한다.
- readObject 메서드에서 인수가 유효한지 검사해야하고 필요하다면 방어적으로 복사하라
- readObject에서 이 작업을 제대로 하지 못하면 공격자는 쉽게 클래스의 불변식을 깨뜨릴 수 있다

#### 역직렬화시, 불변식을 만족하는 유효성 검사를 해야한다

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```

- 위의 if문 추가로 허용되지 않는 Period 인스턴스가 생성되는 일을 막을 수 있지만, 아직도 미묘한 문제가 숨어있다.
- 정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들 수 있다.

#### 가변 공격을 막기 위해서는 방어적 복사본을 만들어야 한다

```java
public class MutablePeriod {
    //Period 인스턴스
    public final Period period;
    
    //시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;
    //종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;
    
    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectArrayOutputStream out = new ObjectArrayOutputStream(bos);
            
            //유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));
            
            /**
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고
             */
            byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
            bos.write(ref); // 시작 start 필드 참조 추가
            ref[4] = 4; //참조 #4
            bos.write(ref); // 종료(end) 필드 참조 추가
            
            // Period 역직렬화 후 Date 참조를 훔친다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
    
/* 공격 코드 */
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;
    
    //시간 되돌리기
    pEnd.setYear(78);
    System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
    
    //60년대로 회귀
    pEnd.setYear(60);
    System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1969
}
```

- Period 인스턴스는 불변식을 유지한 채 생성됐지만 의도적으로 내부의 값을 수정할 수 있었다.
  - 변경할 수 있는 Period 인스턴스를 획득한 공격자는 인스턴스가 불변이라고 가정하는 클래스에 넘겨 엄청난 보안 문제를 일으킬 수 있다.

- 이 문제의 근원은 **Period의 readObject메서드가 방어적 복사를 충분히 하지 않은 데 있다.**
  - 객체를 직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 방어적으로 복사해야 한다.
  - 따라서 `readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사 해야한다.`

#### readObject 메서드에서는 private 가변 요소를 방어 복사하라

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```

- 방어적 복사를 유효성 검사보다 앞서 수행하며, Date의 clone 메서드는 사용하지 않았음
- 두 조치 모두 Period를 공격으로 부터 보호하는데 필요하다
- 또한 final 필드는 방어적 복사가 불가능 하니 주의
- 그래서 이 readObject를 사용하려면 start와 end필드에서 final 한정자를 제거해야 한다

#### 기본 readObject 사용 여부 체크리스트

- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
  - 아니오 -> 커스텀 readObject 메서드를 만들어 유효성 검사와 방어적 복사를 수행
  - 예 -> 기본 readObject 메서드 사용
- 직렬화 프록시 패턴을 사용해도 된다 .
  - 역직렬화를 안전하게 만드는 데 필요한 노력을 경감해 준다. (권장)

### [item89] 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

- 싱글턴 클래스는 선언에 implements Serializable 을 추가하는 순간 더이상 싱글턴이 아니게 된다

#### readResolve로 readObject가 만들어낸 인스턴스를 다른 것으로 대체 가능하다

#### 예시) 잘못된 싱글턴과 도둑 클래스

```java
public class Elvis implements Serializable {
  public static final Elvis INSTANCE = new Elvis();
  private static final long serialVersionUID = 0;
  private String[] favoriteSongs = {"1111", "2222"};

  private Elvis() {

  }

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }

  private Object readResolve() {
    return INSTANCE;
  }
}
/* 도둑 클래스 */
public class ElvisStealer implements Serializable {

  private static final long serialVersionUID = 0;
  static Elvis impersonator;
  private Elvis payload;

  private Object readResolve() {
    // resolve되기 전의 Elvis 인스턴스의 참조를 저장한다
    impersonator = payload;
    // favoriteSongs 필드에 맞는 타입의 객체를 반환한다
    return new String[]{"A Fool Such as I"};
  }

}
```

```java
/* 직렬화의 허점을 이용, 싱글턴 객체를 2개 생성한다 */
public class ElvisImpersonator {
  // 진짜 Elvis 인스턴스로는 만들어 질 수 없는 바이트 스트림
  private static final byte[] serializedForm = new byte[] { (byte) 0xac,
      (byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05, 0x45, 0x6c, 0x76,
      0x69, 0x73, (byte) 0x84, (byte) 0xe6, (byte) 0x93, 0x33,
      (byte) 0xc3, (byte) 0xf4, (byte) 0x8b, 0x32, 0x02, 0x00, 0x01,
      0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76, 0x6f, 0x72, 0x69, 0x74, 0x65,
      0x53, 0x6f, 0x6e, 0x67, 0x73, 0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61,
      0x76, 0x61, 0x2f, 0x6c, 0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a,
      0x65, 0x63, 0x74, 0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45,
      0x6c, 0x76, 0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72,
      0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
      0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64, 0x74,
      0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b, 0x78, 0x70,
      0x71, 0x00, 0x7e, 0x00, 0x02 };

  public static void main(String[] args) {
    // ElvisStealer.impersonator 를 초기화 한 다음
    // 진짜 Elvis를 반환
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;

    elvis.printFavorites();
    impersonator.printFavorites();
  }
}
```

- 위 프로그램은 서로 다른 2개의 Elvis 인스턴스를 생성할 수 있다
  - 11111,22222 / A Fool Such as I
- favoriteSongs 필드를 transient로 선언하여 이 문제를 고칠 수 있으나, Elvis를 원소 하나짜리 열거 타입으로 바꾸는 편이 낫다
  - ElvisStealer 공격으로 보이듯 readResolve 메서드를 사용해 순간적으로 만들어진 역직렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고, 신경을 많이 써야 한다

```java
/* 열거 타입 싱글턴 */
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs = {"Hound Dog", "HeartBreak Hotel"};
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

- final 클래스에서는 readResolve 메서드는 private 이어야 한다
- final이 아닌 클래스에서는
  - private으로 선언하면 하위 클래스에서 사용할 수 없다
  - package-private으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다
  - protected, public으로 선언하면 이를 재정의하지 않은 모든 하위클래스에서 사용할 수 있다
  - protected, public 이면서 하위 클래스에서 재정의하지 않았다면
    - 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킨다

### [item90] 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

#### 직렬화 프록시 패턴

```java
class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    private static class SerializationProxy implements Serializable {
        private static final long serialVersionUID = 2123123123;
        private final Date start;
        private final Date end;

        public SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        /**
         * Deserialize 할 때 호출된다.
         * 오브젝트를 생성한다.
         */
        private Object readResolve() {
            return new Period(start, end);
        }
    }


    /**
     * 이로 인해 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다.
     * 직렬화할 때 호출되는데, 프록시를 반환하게 하고 있다.
     *
     * Serialize할 때 호출된다.
     */
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    /**
     * readObject, writeObject 가 있다면, 기본적으로 Serialization 과정에서
     * ObjectInputStream, ObjectOutputStream이 호출하게 된다.
     * 그 안에 커스텀 로직을 넣어도 된다는 것.
     */
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
        // readObject는 deserialize할 때, 그러니까 오브젝트를 만들 때인데.
        // 이렇게 해두면, 직접 Period로 역직렬화를 할 수 없는 것이다.
        throw new InvalidObjectException("프록시가 필요해요.");
    }
}
```

- 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계, private static으로 선언
  - 이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시
  - 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다
    - 이 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사
- 바깥 클래스와 직렬화 프록시 모두 Serializable을 구현한다

#### 직렬화 프록시 패턴의 장점

- 멤버 필드를 final로 선언할 수 있어, 진정한 불변으로 만들 수 있다
- 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 클래스가 달라도 정상적으로 동작한다

#### 직렬화 프록시 패턴의 단점

- 클라이언트가 마음대로 확장할 수 있는 클래스에는 적용할 수 없다
- 객체 그래프에 순환이 있는 클래스에는 적용할 수 없다
- 방어적 복사보다 상대적 속도가 느리다