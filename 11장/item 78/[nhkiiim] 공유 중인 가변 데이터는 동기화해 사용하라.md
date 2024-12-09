## 공유 중인 가변 데이터는 동기화해 사용하라

#
### 동기화란?
> synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다. (동기화)

- 다른 스레드가 변경 중이어서 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 것
  - 락을 걸어서 다른 스레드가 보지 못하게 막는 것 (일관성이 깨진 상태를 볼 수 없음)
- 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해주는 것

#
### 동기화가 필요한 이유
- long, double 외의 변수를 읽고 쓰는 동작은 원자적
- 여러 스레드가 한 변수를 수정해도 항상 정상적으로 저장한 값을 온전히 읽어옴을 보장
- 하지만 '수정이 완전히 반영된' 값은 보장해주지만 해당 값이 다른 스레드에게 보이는가는 보장해주지 않음

> __동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다__

#### 기본형 타입이 원자적인 이유  
> JVM은 데이터를 4바이트(32비트) 단위로 처리한다.
> 
> int 보다 작은 타입 : 하나의 명령어로 처리되기 때문에, 한 스레드로만 처리된다(작업의 최소 단위).
> long, double : 8 바이트를 넘어가기 때문에 여러 스레드가 개입될 여지가 생겨 원자적이라 할 수 없다.

#
### 다른 스레드를 멈추는 법

- 우선 Thread.stop은 사용하지 말자 : 데이터가 훼손될 수 있다 (자바 11에서부터는 제거됨)

```java
// boolean 필드를 활용한 잘못된 코드
public class StopThread {
	private static boolean stopRequested;
    
  public static void main(String[] args) throws InterruptedException {
    Thread th = new Thread(() -> {
        int i = 0;
          while (!stopRequested) {
            i++;
          }
      });
      th.start();
      
      TimeUnit.SECONDS.sleep(1);
      stopRequested = true;
  }
}
```
```java
// JVM의 최적화된 코드
if (!stopRequested)  // 끌어올리기(최적화 기능) : 응답 불가 상태가 됌
	while (true)
    	i++;
```
- `stopRequested` 필드가 동기화 되지 않아 무한루프에 빠진다
> JVM의 최적화된 코드에서 메인 스레드가 수정한 true 값이 반영되기 이전 그 짧은 시간 사이에 stopRequested 필드를 읽어와 버렸다면,
> if문이 이미 false 가 되었기 때문에 응답 불가 상태가 된다

#
### 🐶 올바르게 종료하는 방법 : synchronized
```java
public class StopThread {
	private static boolean stopRequested;
  private static synchronized void requestStop() { // 쓰기
     stopRequested = true;
  }

  private static synchronized boolean stopRequested() { // 읽기
    return stopRequested;
  }

  public static void main(String[] args) throws InterruptedException {
    Thread th = new Thread(() -> {
        int i = 0;
          while (!stopRequested()) {
            i++;
          }
      });
      th.start();
      
      TimeUnit.SECONDS.sleep(1);
      requestStop();
  }
}
```
- 쓰기 메서드와 읽기 메서드를 모두 동기화해 스레드가 1초 뒤 종료됨

#
### 🐶 속도가 더 빠른 대안 : volatile
- stopRequested 필드를 volatile로 선언하기
- volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게됨을 보장

> volatile 이란, 공유 변수를 캐시가 아닌 메인 메모리에 저장하겠다라고 명시하는 키워드이다.
> 참고로 해당 키워드가 선언된 변수가 있는 코드는 최적화되지 않는다.

```java
public class StopThread {
	private static volatile boolean stopRequested;
    ...
}
```

- 하지만 멀티 스레드 환경에서, 여러 스레드가 쓰기 연산을 하는 경우에는 적합하지 않다

> 참고 내용 : https://velog.io/@semi-cloud/Java-아이템-78-공유-중인-가변-데이터는-동기화해-사용하라

```java
private static volatile int nextNubmer = 0;

public static int generateNumber() {
	return nextNumber++;
}
```
1. 스레드1이 메인 메모리 변수 값을 읽어오고, 1 증가시킨다.
2. 스레드2가 스레드1이 변경시킨 값이 메인 메모리에 반영 되기 전에 변수 값을 읽어오고, 1 증가시킨다.
3. 둘 다 변수 0 일때의 상태에서 1 을 증가시켰으므로 2 가 아닌 1 이 최종적으로 반영되버린다.

> 이처럼 volatile은 공유 블록에 한번에 하나의 스레드만 접근하는 것을 막지 못한다.  

> 따라서 이런 경우, volatile 을 제거하고 synchronized 를 붙여서 동시에 호출해도 서로 간섭하지 않는 상태에서 이전 호출이 변경한 값을 읽어올 수 있다.
> 앞선 StopThread 예제에서 volatile 사용이 가능했던 이유는, 오직 스레드 하나만 stopRequested 공유 변수에 쓰기 연산을 하고 있었기 때문이다.

#
### 🐶 락-프리 안전한 클래스 : AtomicLong
- `java.util.concurrent.atomic` 패키지의 AtomicLong 을 사용하면 멀티 스레딩 환경에서 락 없이도 안전하게 사용할 수 있다.

private static final AtomicLong nextNum = new AtomicLong();

public static long generateNumber() {
	return nextNum.getAndIncrement();

