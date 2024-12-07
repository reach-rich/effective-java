## 1. 동기화

Java에서 동기화를 사용하려면 synchronized 키워드를 사용할 수 있습니다.

동기화된 메서드나 블록은 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 같게 합니다.

반대로 말하면 여러 스레드가 존재하는 멀티 스레드 프로그램에서 동기화하지 않으면

객체 상태가 일관되지 않아 어떤 결과로 이어질지 알 수 없어 위험합니다.

## 2. 원자성(atomic)

언어 명세상 long과 double 외 변수를 읽고 쓰는 동작은 원자성을 가지고 있습니다.

즉, 여러 스레드가 같은 변수를 동기화 없이 수정하더라도 

항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어온다는 것을 보장합니다.

하지만 Java 언어 명세는 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된' 값을 얻는다고 보장하지만,

한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않습니다.

따라서 동기화는 스레드 사이의 안정적인 통신을 위해 꼭 필요합니다.

## 3. 동기화에 실패한 예시

가변 데이터를 원자적으로 읽고 쓸 수 있을지라도 동기화에 실패하면 처참한 결과로 이어질 수 있습니다.

```java
    public class StopThread {
        private static boolean stopRequested;

        public static void main(String[] args) throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested)
                    i++;
            });
            backgroundThread.start();

            TimeUnit.SECONDS.sleep(1);
            stopRequested = true;
        }
    }
```

해당 예시를 보면 `TimeUnit.SECONDS.sleep(1);`에 의해 1초 이후에 종료될 것처럼 보입니다.

하지만 실제로 실행해보면 반복문을 빠져나오지 못하고 무한히 수행합니다.

해당 원인은 동기화에 있습니다.

동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 보게 될 지 모르기 때문입니다.

동기화하지 않은 위 코드는 가상머신이 다음과 같이 최적화를 수행할 수 있습니다.

```java
// 기존 코드
while (!stopRequested)
    i++;

// 가상머신이 최적화한 코드 (호이스팅)
if (!stopRequested)
    while (true)
        i++;
```

그 결과 프로그램은 응답 불가 상태가 되어 더 이상 진전이 없어집니다.

## 4. 동기화를 적용하여 문제 해결

이 문제를 해결하기 위해 다음과 같이 동기화 한다면 1초 후 정상적으로 종료되도록 해결할 수 있습니다.

```java
    public class StopThread {
        private static boolean stopRequested;

        private static synchronized void requestStop() {
            stopRequested = true;
        }

        private static synchronized boolean stopRequested() {
            return stopRequested;
        }

        public static void main(String[] args) throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested)
                    i++;
            });
            backgroundThread.start();

            TimeUnit.SECONDS.sleep(1);
            requestStop();
        }
    }
```

여기서 쓰기 메서드(requestStop)와 읽기 메서드(stopRequested) 모두 동기화했음에 주목해야 합니다.

둘 중 하나만 적용한다면 동작을 보장하지 않습니다.

## 5. volatile

여기서 `volatile` 키워드를 사용하면 동기화 비용을 줄이면서 속도 최적화를 이룰 수 있습니다.

아래와 같이 stopRequest 필드를 `volatile` 키워드를 적용하면

메인 메모리 영역을 참조하게 되므로 동기화를 생략할 수 있습니다.

```java
    public class StopThread {
        private static volatile boolean stopRequested;

        public static void main(String[] args) throws InterruptedException {
            Thread backgroundThread = new Thread(() -> {
                int i = 0;
                while (!stopRequested)
                    i++;
            });
            backgroundThread.start();

            TimeUnit.SECONDS.sleep(1);
            stopRequested = true;
        }
    }
```

하지만 `volatile` 키워드는 주의해서 사용해야 합니다.

예를 들면 다음의 일련번호 생성 코드입니다.

```java
    private static volatile int nextSerialNumber = 0;

    public static int generateSerialNumber() {
        return nextSerialNumber++;
    }
```

해당 코드는 매번 고유한 값을 반환할 의도로 만들어졌습니다.

하지만 멀티 스레드 프로그램인 경우 해당 코드는 정상적으로 동작하지 않습니다.

그 이유는 증가 연산자(++) 때문입니다.

증가 연산자는 코드 상으로는 하나지만 실제로는 nextSerialNumber 필드에 두 번 접근합니다.

그러므로 두 접근 사이 다른 스레드가 값을 읽어가는 경우, 같은 값을 받게 되는 문제가 생길 수 있습니다.

그래서 아래와 같이 동기화를 적용하면 문제 없이 동작합니다.

주의해야 할 점은 메서드에 `synchronized`를 적용하면 `volatile` 키워드를 제거해야 합니다.

```java
    private static int nextSerialNumber = 0;

    public static synchronized int generateSerialNumber() {
        return nextSerialNumber++;
    }
```

## 6. 락-프리 동기화

volatile에 이어 또 하나의 대안을 소개합니다.

`java.util.concurrent.atomic` 패키지의 `AtomicLong`을 사용할 수 있습니다.

앞서 소개한 generateSerialNumber 같은 기능에 적합합니다.

```java
    private static final AtomicLong nextSerialNum = new AtomicLong();

    public static long generateSerialNumber() {
        return nextSerialNum.getAndIncrement();
    }
```

## 7. 정리

이번에 언급한 문제를 회피하는 가장 좋은 방법은 애초에 가변 데이터를 공유하지 않는 것입니다.

즉, 가변 데이터는 단일 스레드 환경에서만 사용하도록 합니다.

이 정책을 받아들였다면 문서에 남겨 유지보수 과정에서도 계속 지켜지도록 해야 합니다.

또한 사용하려는 프레임워크나 라이브러리를 깊이 이해하는 것도 중요합니다.

외부 코드가 인지 못한 스레드를 수행하여 문제가 되는 경우가 있기 때문입니다.