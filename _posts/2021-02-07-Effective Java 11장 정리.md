---
layout: post
title: 210207 Effective Java 11장 정리
tags:
  - Effective Java
---

## 11. 동시성

### [item78] 공유 중인 가변 데이터는 동기화해 사용하라

- 가변 데이터는 단일 스레드에서만 사용하도록 한다

##### 사실상 불변(effectively immutable)

- 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다
  - 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어갈 수 있다
  - 안전 발행(safe publication): 다른 스레드에 이런 객체를 건넴

##### synchronized

- 해당 메서드나 블록을 한 번에 한 스레드씩 수행하도록 보장

#### 동기화 = 배타적 실행 + 스레드 사이의 안정적인 통신

- 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도
  - 한 객체가 일관된 상태를 가지고 생성, 이 객체에 접근하는 메서드는 그 객체에 락을 건다. 락을 건 메서드는 객체의 상태를 확인하고 필요하다면 수정한다(객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다)
  - 동기화를 제대로 사용한다면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없다
- 한 스레드가 만든 변화를 다른 스레드에서 확인하도록 함
  - 일관성이 깨진 상태를 볼 수 없게 함
  - 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다
- long과 double 외의 변수를 읽고 쓰는 동작은 원자적(atomic)
  - 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다
  - "성능을 높이려면 원자적 데이터를 읽고 쓸 때는 동기화하지 말아야겠다"?
    - 자바 언어 명세는 스레드가 필드를 읽을 떄 항상 '수정이 완전히 반영된 값'을 얻는다고 보장한다
    - 하지만 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다.
      - 한 스레드가 만든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 자바의 메모리 모델 때문이다.

#### Thread.stop은 사용하지 말라

- 공유중인 가변 데이터를 비록 원자적으로 읽고 쓸 수 있을지라도 동기화에 실패한다면 나쁜 결과로 이어진다
- Thread.stop 메서드는 안전하지 않아 이미 오래전에 사용 자제 API로 지정되었다.

```java
/* 잘못된 코드 */
public class StopThread {
    private static boolean stopRequested;
    
    public static void main(String[] args)
        throws InterruptedException {
       	Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested)
                i++;
            /* 다음과 같이 최적화될 수 있다
            if(!stopRequested)
            	while(true) i++;
            */
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
        // 메인 스레드가 1초 후 stopRequested를 true로 설정하면서 반복문을 빠져 나올 것으로 예상한다 => 하지만 무한루프를 돌게 된다
    }
}

/* 올바른 코드 */
public class Stopthread {
    private static boolean stopRequested;
    
    // stopRequested 필드를 동기화해 접근한다
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }
    
    public static void main(String[] args)
    throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested)
                i++;
            });
        backgroundThread.start();
        
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

- 쓰기 메서드와 읽기 메서드를 모두 동기화해야 동작을 보장할 수 있다

- volatile 필드로 선언하면 동기화를 생략할 수 있다

  - ```java
    public class Stopthread {
        private static volatile boolean stopRequested;
        
        public static void main(String[] args)
        throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while(!stopRequested)
                    i++;
                });
            backgroundThread.start();
            
            TimeUnit.SECONDS.sleep(1);
            stopRequested = true;
        }
    }
    ```

  - volatile 한정자는 배타적 수행과는 관계없으나, 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다

#### volatile 한정자 사용시 주의점

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

- 이 메서드는 매번 고유한 값을 반환할 의도로 만들어졌다
- 이 메서드의 상태는 nextSerialNumber라는 단 하나의 필드로 결정된다
  - 원자적으로 접근할 수 있고, 어떤 값이든 허용한다
  - 따라서 굳이 동기화하지 않더라도 불변식을 보호할 수 있어 보인다?
    - 하지만 동기화 없이는 올바로 동작하지 않는다!
- ++연산자는 코드상으로는 하나지만 실제로는 필드에 두번 접근한다
  - 값을 읽고, 새로운 값을 저장
  - 두 접근 사이에 값을 읽어가게 되면 첫번째 스레드와 똑같은 값을 돌려받게 된다(안전 실패`safety failure`, 프로그램이 잘못된 결과를 계산해냄)
- generateSerialNumber 메서드에 synchronized 한정자를 붙이고, nextSerialNumber에서는 volatile을 제거한다

#### java.util.concurrent.atomic 패키지의 AtomicLong

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static int generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

- 락 없이도(lock-free) 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다
- volatile은 동기화의 두 효과 중 통신 쪽만 지원하지만, 이 패키지는 원자성(배타적 실행)까지 지원한다

---

### [item79] 과도한 동기화는 피하라

- 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작응 낳기도 한다
- 기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것이다
  - 오래 걸리는 작업이라면 동기화 영역 바깥으로 옮기는 방법을 찾아본다

#### 동기화 메서드/동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다

- 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안 된다
- 외계인 메서드(alien method): 이 메서드가 무슨 일을 할 지 알지 못하며 통제할 수도 없다
  - 동기화된 영역이 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수 있다

#### 예시) 동기화 블록 안에서 외계인 메서드 호출

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set);}
    
    private final List<SetObserver<E>> observers = new ArrayList<>();
    
    public void addObserver(SetObserver<E> observer){
        synchronized(observers){
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SetObserver<E> observer){
        synchronized(observers){
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        synchronized(observers){
            for(SetObserver<E> observer : observers)
                observer.addad(this, element);
        }
    }
    
    @Override public boolean add(E element){
        boolean added = super.add(element);
        if(added)
            notifyElementAdded(element);
        return added;
    }
    
    @Override public boolean addAll(Collection<? extends E> c){
        boolean result = false;
        for(E element : c)
            result |= add(element);
        return result;
    }
}
```

- 관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다

  - 두 경우 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다

  - ```java
    @FunctionalInterface public interface SetObserver<E> {
        // ObservableSet에 원소가 더해지면 호출된다
        void added(ObservableSet<E> set, E element);
    }
    ```

  - 이 인터페이스는 구조적으로 `BiConsumer<ObsevableSet<E>,E>`와 같다

```java
/* 0~23까지 출력 후 관찰자 자신을 구독해지, 조용히 종료하는 프로그램: 정상동작할까? */
public static void main(Stirng[] args){
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    
    set.addObserver( new SetObserver<>(){
        public void added(ObservableSet<Integer> s, Integer e){
            System.out.println(e);
            if(e==23)
                s.removeObserver(this); 
            // s.removeObserver 메서드에 함수 객체 자신을 넘겨야 하기 때문에, 람다 대신 익명 클래스를 사용하였다.
        }
    });
}
```

- 23까지 출력한 후 ConcurrentModificationException을 던진다
  - 관찰자의 added메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문
  - 리스트에서 원소를 제거하려 하는데, 리스트를 순회하는 도중이다(허용되지 않은 동작)
  - notifyElementAdded메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거켜 되돌아와 수정하는 것까지 막지는 못한다.

##### 백그라운드 스레드를 사용

```java
set.addObserver(new SetObserver<>(){
   public void added(ObservableSet<Integer> s, Integer e){
       System.out.println(e);
       if(e==23){
           ExecutorService exec = Executors.newSingleThreadExecuter();
           try{
               exec.submit(() -> s.removeObserver(this)).get();
           } catch (ExecutionException | InterruptedException ex){
               // 다중 캐치(자바 7부터 지원)
               throw new AssertionError(ex);
           } finally {
               exec.shutdown();
           }
       }
   } 
});
```

- 예외는 발생하지 않으나 교착상태에 빠지게 된다
  - 백그라운드 스레드가 s.removeObserver를 호출하면
  - 관찰자를 잠그려 시도하지만 락을 얻을 수 없다
  - 메인 스레드가 이미 락을 쥐고 있지만, 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리는 중이다
- 실제 시스템에서도 동기화된 영역 안에서 외계인 메서드를 호출하여 교착상태에 빠지는 사례는 자주 있다

##### 불변식이 임시로 깨진 경우

- 자바 언어의 락은 재진입을 허용하므로 교착상태에 빠지지는 않는다
- 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공한다
  - 그 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행중인데도 말이다!
- 재진입 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답 불가(교착 상태)가 될 상황을 안전 실패(데이터 훼손)로 변모시킬 수도 있다

#### 해결: 외계인 메서드 호출을 동기화 블록 바깥으로 옮긴다

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers){
        snapshot = new ArrayList<>(observers);
    }
    for(SetObserver<E> observer : snapshot)
        observer.added(this,element);
}
```

- 열린 호출(open call): 동기화 영역 바깥에서 호출되는 외계인 메서드	
  - 외계인 메서드는 얼마나 오래 실행될지 알 수 없으나, 동기화 영역 안에서 호출된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야만 한다
  - 실패 방지 효과+동시성 효율 증대
- 또는 자바의 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList를 사용한다
  - 수정할 일은 드물고 순회만 빈번히 일어나는 관찰자 리스트 용도로는 좋다

#### 해결2: CopyOnWriteArrayList를 사용한다

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observeer) {
    observer.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for(SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

##### 자바의 동기화 비용(성능)

- 멀티코어가 일반화된 오늘날, 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다
- 가상머신의 코드 최적화를 제한한다

#### 가변 클래스를 작성하려면

- 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 한다
- 동기화를 내부에서 수행해 스레드 안전한 클래스로 만든다
  - 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만
  - 락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 동원해 동시성을 높여줄 수 있다
- 예시) StringBuffer 클래스: 항상 단일 스레드에서 사용되었음에도 내부적으로 동기화 수행
  - StringBuilder 클래스 등장(동기화하지 않은 StringBuffer 클래스)
- 예시2) java.util.Random은 동기화하지 않는 버전인 java.util.ThreadLocalRandom으로 대체

---

### [item80] 스레드보다는 실행자,태스크,스트림을 애용하라

```java
// 실행자 프레임워크: 인터페이스 기반의 유연한 태스크 실행 기능
ExecutorService exec = Executors.newSingleThreadExecutor();
// 실행자에 실행할 태스크를 넘기는 방법
exec.execute(runnable);
// 실행자를 종료한다
exec.shutdown();
```

#### 실행자의 다양한 기능들

- 특정 태스크가 완료되기를 기다린다
- 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다린다
- 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드)
- 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용)
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThreadPoolExecutor 이용)

#### 실행자 서비스를 사용하기 까다로운 애플리케이션

- 작은 프로그램이나 가벼운 서버라면 Executors.newCachedThreadPool
  - 무거운 프로덕션 서버에는 적절하지 못하다
  - CachedThreadPool에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행된다
  - 가용한 스레드가 없다면 새로 하나를 생성
  - 서버가 아주 무겁다면 CPU 이용률이 100%로 치닫고, 새로운 태스크가 도착하는 족족 또 다른 스레드를 생성하며 상황을 더욱 악화시킨다
- 무거운 프로덕션 서버에서는 스레드 개수를 고정한 Executors.newFixedThreadPool을 선택
- 또는 완전히 통제할 수 있는 ThreadPoolExecutor를 사용

#### 태스크

- 실행자 프레임워크에서는 작업 단위와 수행 매커니즘이 분리된다
- 작업 단위를 나타낸다
- Runnable / Callable(Callable은 Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있음)
- 태스크를 수행하는 일반적인 메커니즘이 바로 실행자 서비스
  - 원하는 태스크 수행 정책을 선택할 수 있다
  - 변경이 용이하다

#### 포크-조인 태스크

- 자바 7이 되면서 실행자 프레임워크가 포크-조인 태스크를 지원하도록 확장됨
- ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있다
  - ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리한다
  - 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다
  - 모든 스레드가 바쁘게 움직여 CPU를 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성한다.
- 포크-조인에 적합한 형태의 작업이라면 포크-조인 풀을 이용해 만든 병렬 스트림을 이용한다

---

### [item81] wait와 notify보다는 동시성 유틸리티를 애용하라

- wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용한다

##### java.util.concurrent의 세 범주

- 실행자 프레임워크
- 동시성 컬렉션
- 동기화 장치

#### 동시성 컬렉션

- List,Queue,Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션
- 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다
- 동시성 컬렉션에서 동시성을 무력화하는 것은 불가능
  - 여러 메서드를 원자적으로 묶어 호출하는 일 역시 부가능
  - 상태 의존적 수정 메서드들이 추가됨
    - 여러 기본 동작을 하나의 원자적 동작으로 묶음
    - 자바 8에서는 일반 컬렉션 인터페이스에도 디폴트 메서드 형태로 추가됨
    - 예시) Map의 putIfAbsent(key, value) 메서드
      - 주어진 키에 매핑된 값이 아직 없을때만 새 값을 넣는다
      - 기존 값이 있었다면 그 값을 반환하고, 없었다면 null을 반환
- 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다
- 동시성 컬렉션은 동기화한 컬렉션을 낡은 유산으로 만들어 버렸다
  - Collections.synchronizedMap보다는 ConcurrentHashMap을 사용하라
- 컬렉션 인터페이스중 일부는 작업이 성공적으로 완료될때까지 기다리도록(차단되도록) 확장되었다
  - 예시: Queue를 확장한 BlockingQueue의 take 메서드
    - 큐의 첫 번째 원소를 꺼내고, 큐가 비어있다면 새로운 원소가 추가될 떄까지 기다린다 - BlockingQueue는 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다
      - 하나 이상의 생산자 스레드가 작업을 큐에 추가
      - 하나 이상의 소비자 스레드가 큐에 있는 작업을 꺼내 처리
      - ThreadPoolExecutor를 포함한 대부분의 실행자 서비스 구현자에서 사용

#### 동기화 장치

- 스레드가 다른 스레드를 기다릴 수 있게 하고, 서로 작업을 조율할 수 있게 해준다
- CountDownLatch, Semaphore
- CyclicBarrier, Exchanger, Phaser

##### CountDownLatch

- 일회성 장벽
- 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날때까지 기다리게 한다
- 생성자는 int값을 받으며, 이 값이 래치의 countDown 메서드를 몇번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다

##### CountDownLatch 예제) 동시 실행 시간을 재는 간단한 프레임워크

```java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    
    CountDownLatch done = new CountDOwnLatch(concurrency);
    
    for(int i=0;i<concurrency;i++){
        executor.execute(()->{
            // 타이머에게 준비를 마쳤음을 알린다.
            ready.countDown();
            try{
                // 모든 작업자 스레드가 준비될때까지 기다린다
                start.await();
                action.run();
            } catch (InterruptedException e){
                Thread.currentThread().interrupt();
            } finally {
                // 타이머에게 작업을 마쳤음을 알린다
                done.countDown();
            }
        });
    }
    
    ready.await(); // 모든 작업자가 준비될 때까지 기다린다
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자들을 깨운다
    done.await(); // 모든 작업자가 일을 끝마치기를 기다린다
	return System.nanoTime() - startNanos;    
}
```

- 카운트다운 래치 3개
  - ready : 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지
  - start : 통지를 끝낸 작업자 스레드들이 이 래치가 열리기를 기다린다
    - ready.countDown() : 타이머 스레드가 시작 시간을 기록
    - start.countDown() : 기다리던 작업자 스레드들을 깨운다
  - done : 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다
    - 타이머 스레드는 done 래치가 열리자마자 깨어나 종료 시간을 기록한다
  - CyclicBarrier나 Phaser 인스턴스 하나로 대체할 수 있다
    - 더 명료하지만 이해하기는 훨씬 어렵다
- 시간 간격을 잴 때는 항상 System.currentTimeMillis가 아닌 System.nanoTime을 사용하라
  - 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향받지 않는다

##### wait 메서드

```java
synchronized(obj) {
    while(<조건이 충족되지 않았다>)
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다
    ... // 조건이 충족됐을 때의 동작을 수행한다
}
```

- wait 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용하라
  - 반복문 밖에서는 절대 호출하지 말자
- 대기 전 조건을 검사해 조건이 이미 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치

##### 대기 후 조건 검사, 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황

- 스레드가 notify를 호출한 다음 대기중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다
- 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다. 공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다. 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.
- 깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 꺠울 수도 있다
- 대기 중인 스레드가 (드물게) notify 없이도 깨어나는 경우가 있다. 허위 각성(spurious wakeup)이라는 현상이다
- notify와 notifyAll을 선택하는 문제
  - notify는 스레드 하나만 꺠우며, notifyAll은 모든 스레드를 깨운다
  - 일반적으로 언제나 notifyAll을 사용하라
  - 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 notify를 사용해 최적화
    - notify를 사용한다면 응답 불가 상태에 빠지지 않도록 각별히 주의한다

---

### [item82] 스레드 안전성 수준을 문서화하라

- 메서드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않는다
  - 그 메서드가 스레드 안전하다고 믿기 어렵다

#### 스레드 안전성 수준

##### 불변(immutable)

- 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화도 필요 없다
- String, Long, bigInteger가 대표적이다

##### 무조건적인 스레드 안전(unconditionally thread-safe)

- 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다
- AtomicLong, ConcurrentHashMap이 여기에 속한다

##### 조건부 스레드 안전(conditionally thread-safe)

- 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다
- Collections.synchronized 래퍼 메서드가 반환한 컬렉션들이 여기 속한다
  - 이 컬렉션들이 반환한 반복자는 외부에서 동기화해야 한다

##### 스레드 안전하지 않음(not thread-safe)

- 이 클래스의 인스턴스는 수정될 수 있다
- 동시에 사용하려면 각각의(혹은 일련의) 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다
- ArrayList, HashMap과 같은 기본 컬렉션

##### 스레드 적대적(thread-hostile)

- 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다
- 일반적으로 정적 데이터를 아무 동기화 없이 수정한다
- 이런 클래스를 고의로 작성하는 사람은 없겠으나, 동시성을 고려하지 않고 작성하다 보면 우연히 만들어질 수 있다
- 일반적으로 문제를 고쳐 배포하거나 사용 자제 API로 지정한다

#### 조건부 스레드 안전한 클래스는 주의해서 문서화해야 한다

- 어떤 순서로 호출할 떄 외부 동기화가 필요한지?
- 그 순서로 호출하려면 어떤 락 혹은 어떤 락들을 얻어야 하는지?

#### 클래스가 외부에서 사용할 수 있는 락을 제공하면

- 클라이언트에서 일련의 메서드 호출을 원자적으로 수행할 수 있다
  - 내부에서 처리하는 고성능 동시성 제어 메커니증과 혼용할 수 없게 된다
  - 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격을 수행할수도 있다

##### 서비스 거부 공격 막기: 비공개 락 객체

```java
private final Object lock = new Object();
// lock 객체를 final로 선언하여 락 필드가 교체되는 것을 막는다

public void foo() {
    synchronized(lock){
        ...
    }
}
```

- synchronized 메서드 대신 비공개 락 객체를 사용한다
  - 락 객체를 동기화 대상 객체 안으로 캡슐화
- 비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용할 수 있다
- 비공개 락 객체는 상속용으로 설계한 클래스에 특히 잘 맞는다

---

### [item83] 지연 초기화는 신중히 사용하라

#### 지연 초기화(lazy initialization)

- 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
- 값이 전혀 쓰이지 않으면 초기화도 전혀 일어나지 않는다
- 클래스/인스턴스 생성시의 초기화 비용은 줄지만 그 대신 지연 초기화하는 필드에 접근하는 비용은 커진다
- 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 클 때 사용

#### 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다

##### 일반적인 초기화

```java
private final FieldType field = computeFieldValue();
```

##### 인스턴스 필드의 지연 초기화 - synchronized 방식

```java
private FieldType field;

private synchronized FieldType getField() {
    if(field == null)
        field = computeFieldValue();
    return field;
}
```

- 지연 초기화가 초기화 순환성을 깨뜨릴 것 같을때, synchronized를 단 접근자를 사용한다

##### 정적 필드의 지연 초기화 - 지연 초기화 홀더 클래스

```java
private staitc class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field;}

```

- 성능 때문에 정적 필드를 지연 초기화 해야 하는 경우

##### 인스턴스 필드 지연 초기화용 이중검사 관용구

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if(result != null){
        return result;
    }
    synchronized(this){
        if(field == null)
            field = computeFieldValue();
        return field;
    }
}
```

- 한번은 동기화 없이 검사 / 두번째는 동기화하여 검사
- 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile로 선언

##### 단일검사 관용구

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if(result == null)
        field = result = computeFieldValue();
    return result;
}   
```

- 반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화해야 할 때

---

### [item84] 프로그램의 동작을 스레드 스케줄러에 기대지 말라

- 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다

#### 견고하고 빠릿하며, 이식성 좋은 프로그램을 작성하는 가장 좋은 방법

- 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 한다
- 실행 준비가 된 스레드들은 맡은 작업을 완료할 때까지 계속 실행되도록 만든다
  - 스레드 스케줄링 정책이 상이한 시스템에서도 동작이 크게 달라지지 않는다
- 실행 가능한 스레드 수와 전체 스레드 수는 구분한다
  - 전체 스레드 수는 훨씬 많을 수 있고, 대기 중인 스레드는 실행 가능하지 않다

##### 실행 가능한 스레드 수를 적게 유지하는 주요 기법

- 각 스레드가 무언가 유용한 작업을 완료한 후에는 다음 일거리가 생길 때까지 대기하도록 하는 것이다
- 스레드는 당장 처리할 작업이 없다면 실행되어서는 안된다
- 스레드는 절대 바쁜 대기(busy waiting) 상태가 되면 안 된다
  - 공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사해서는 안 된다
  - 바쁜 대기는 스레드 스케줄러의 변덕에 취약하며, 프로세서에 큰 부담을 준다

##### 예시) 바쁜 대기 상태가 되는 CountDownLatch 

```java
public class SlowCountDownLatch {
    private int count;
    
    public SlowCountDownLatch(int count){
        if(count<0)
            throw new IllegalArgumentException(count+"<0");
        this.count=count;
    }
    
    public void await(){
        while(true){
            synchronized(this){
                if(count==0) return;
            }
        }
    }
    
    public synchronized void countDown() {
        if(count!=0){
            count--;
        }
    }
}
```

- 약 10배 느리다...

##### Thread.yield는 자제하자

- 테스트할 수단이 없다
- 이식성이 나빠진다

##### 스레드 우선순위는 자바에서 이식성이 가장 나쁜 특성

- 심각한 응답 불가 문제를 스레드 우선순위로 해결하려는 시도는 절대 합리적이지 않다