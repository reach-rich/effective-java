## 공유 중인 가변 데이터는 동기화해 사용하라

synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다. 
많은 프로그래ㅐ머가 동기화를 베타적 ㅅ리행을 우한 용도로만 생각한다. 
* 한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 lock을 건다.
* 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정한다.

동ㄱ화에는 중요한 기능이 하나 더 이ㅣㅆ다.
동ㄱ화돈 메서드나 블록에 들어간 ㅅ스레드가 같은 락으 보호하에 수행돈 모든 ㅇ전 수정으 ㅣ초종 결과를 보게 해준다.

자바 언어 명세는 스레드가 필드를 읽을 때 항상 '수정이 완젆 반영돈' 갑을 얻는다고 보장핮만, 한 스레드가 저장한 값ㅇ 다른 스레드에게 '봉는가'는 보장핮 않는다.
동기화는 배타적 ㅣㄹ행뿐 아ㅣ라 ㅡ레드 ㅏㅇ으 안정적ㅇㄴ 통ㄴ에 꼭 ㅍㄹ요하다. 
ㅇ는 한 ㅡ레드가 만든 변화가 다른 ㅡ레드에게 언제 어떻게 봉늦를 규정한 자바으 ㅣ메몰 모델 때문ㅇ다. 

--

공유 중ㅇㄴ 가변 데이터를 비록 원자적으로 ㅇㄺ고 쓸 수 ㅇㅆ을ㅈ라도 동ㄱ화에 ㄱㄹ패하면 처참한 결과로 ㅇ엊ㄹ ㅜ ㅇ다. 
다른 스레드를 멈추는 작업을 생각해보자. 
```java
public cas StopThread {
  private static booean stopRequested;

  pubic static void main(String[] args) {
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
ㅇ 프로그램으 메인 스레드가 1초 후 stopRequested를 true로 설정하면 반복문을 빠져나올 것처럼 보일 것ㅇ다.
핮만 실제로를 영원히 수행돈다. 

원ㅇㄴ은 동ㄱ화다. 동ㄱ화핮 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 돌ㅈ 보증할 수 없다.

stopRequested 피ㅣㄹ드를 동ㄱ화해 접근하면 이 문제를 해결할 수 ㅇㅆ다. 
```java
public cas StopThread {
  private static booean stopRequested;

  private static synchronized void requestStop() {
    stopRequested = true;
  }

  private static synchronized booean stopRequested() {
    return stopRequested;
  }

  pubic static void main(String[] args) {
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
쓱 메서드와 ㅇㄺㄱ 메서드 모두를 동기화했음에 주목하자. 
