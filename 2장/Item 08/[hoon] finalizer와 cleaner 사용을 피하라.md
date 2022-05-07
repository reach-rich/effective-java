## 1. 들어가기

Java에서는 finalizer와 cleaner라는 객체 소멸자를 제공합니다.

하지만, Java를 사용하면서 객체 소멸자를 따로 사용해본 경험은 없습니다.

그래서 이번 포스트는 객체 소멸자의 사용법과 

이번 Item에서는 왜 객체 소멸자의 사용을 지양하는지 알아봅시다.

## 2. Java 객체 소멸자 사용법

### 2-1. finalizer

```java
  class Test {
    @Override
    protected void finalize() throws Throwable {
        // clean 작업 수행
        System.out.println("clean");
    }
  }
```

finalizer 사용법은 매우 간단합니다.

Object 클래스는 finalizer 메서드를 가지고 있는데 모든 클래스는 Object 클래스를 상속 받기 때문에

finalizer를 오버라이딩해서 사용할 수 있습니다.

### 2-2. cleaner

```java
  import java.lang.ref.Cleaner;

  class Test {
    private static final Cleaner CLEANER = Cleaner.create();
    private final Cleaner.Cleanable cleanable;
    private final CleanData cleanData;

    Test() {
      this.cleanData = new CleanData();
      this.cleanable = CLEANER.register(this, cleanData);
    }

    private static class CleanData implements Runnable {
      @Override
      public void run() {
          // clean 작업 수행
          System.out.println("clean");
      }
    }
  }
```

cleaner의 사용법은 finalizer에 비해 복잡합니다.

Cleaner와 스레드 객체를 생성한 후,

Cleaner.register 메서드로 스레드 객체를 등록해 사용합니다.

## 3. Java 객체 소멸자의 단점

이번 Item에서는 왜 Java 객체 소멸자의 사용을 지양할까요?

단점을 통해 알아봅시다.

### 3-1. 즉시 수행된다는 보장이 없다. 
<hr>

반복문을 수행하면서 객체를 생성했다가 해제시키는 예시를 보겠습니다.

```java
  static class Item {
    private int id;

    Item(int id) {
      this.id = id;
    }

    @Override
    protected void finalize() throws Throwable {
      System.out.println(id + "번 객체 finalize() 실행");
    }
  }

  public static void main(String[] args) throws InterruptedException {
    for(int num=1; num<=5; num++) {
      Item item = new Item(num);

      item = null;
      System.gc();
    }
    Thread.sleep(100L);
  }
```

```
  1번 객체 finalize() 실행
  5번 객체 finalize() 실행
  4번 객체 finalize() 실행
  3번 객체 finalize() 실행
  2번 객체 finalize() 실행
```

분명 1번부터 5번까지의 객체를 차례대로 생성했다가 해제시켰는데

1-5-4-3-2번 객체 순으로 finalize() 동작이 이루어진걸로 보아 

finalize() 수행이 즉시 수행된다는 보장이 없음을 알 수 있습니다.

### 3-2. 인스턴스 반납을 지연시킬 수 있다.
<hr>

finalizer 스레드는 다른 애플리케이션 스레드보다 우선 순위가 낮아서 언제 실행될 지 모릅니다.

즉, finalizer 스레드보다 우선 순위가 높은 스레드가 계속 들어온다면 계속 뒤로 미뤄진다는 의미입니다.

따라서, 스레드가 계속 들어오는 대기 큐 내부에 finalizer 스레드가 있고,

그 작업을 처리하지 못해 무한정 대기한다면

계속 쌓이다가 결국 OutOfMemoryException이 발생할 수 있습니다.

### 3-3. 아예 실행되지 않는 경우도 있다.
<hr>

Java는 finalizer와 cleaner의 수행 시점 뿐만 아니라 수행 여부조차 보장하지 않습니다.

그래서 접근할 수 없는 일부 객체에 대한 종료 작업을 전혀 수행하지 못한 채 프로그램이 종료될 수 있습니다.

따라서, 프로그램의 상태를 변경하는 작업에서는 절대 finalizer나 cleaner 내부에 작성해서는 안됩니다.

추가로, finalizer와 cleaner를 수행하기 위해 

System.gc 메서드나 System.runFinalization 메서드을 사용할 수 있는데

이 또한 실행 가능성을 높여줄 수 있으나 보장해주지 않습니다.

### 3-4. 실행 시 발생하는 오류의 탐색이 어렵다.
<hr>

이 단점은 finalizer만 해당하는 단점입니다.

finalizer 동작 중 발생한 예외는 무시되고, 처리할 작업이 남았더라도 그 순간 종료됩니다.

이렇게 되면 해당 객체는 자칫 마무리가 되지 않은 상태로 남을 수 있고,

다른 스레드가 해당 객체를 사용하면 어떻게 동작할 지 예측할 수 없습니다.

### 3-5. 심각한 성능 문제도 동반한다.
<hr>

이 책의 저자는 간단한 AutoCloseable 객체를 생성하고 GC로 수거해가기까지의 성능 테스트를 했습니다.

try-with-resources구문으로 만들었을 경우, 12ns 걸리지만

finalizer를 사용한 경우, 550ns가 걸려 50배 정도의 성능이 떨어졌고

cleaner를 사용한 경우, 66ns로 5배 정도의 성능이 떨어졌습니다.

만약, 복잡한 AutoCloseable 객체일 경우, 이보다 더 한 성능 문제로 이어질 수 있습니다.

### 3-6. 심각한 보안 문제를 일으킬 수 있다.
<hr>

이 단점은 finalizer만 해당하는 단점입니다.

finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있습니다.

예시를 통해 알아보겠습니다.

```java
  public class Main {
    static Normal obj;

    static class Normal {
      Integer value;
      Normal(int value) {
        if(value <= 0) {
          throw new IllegalArgumentException();
        }

        this.value = value;
      }
      @Override
      protected void finalize() throws Throwable {}
    }

    static class Attack extends Normal {
        public Attack(int value) {
          super(value);
        }

        @Override
        protected void finalize() throws Throwable {
          obj = this;
          this.value = -1;
        }
    }

    public static void main(String[] args) throws InterruptedException {
      obj = new Attack(1);
      obj = null;
      System.gc();

      Thread.sleep(1000L);
      System.out.println(obj.value);
    }
  }
```

```
  -1
```

Normal 클래스는 양수인 value 값만 가지는 인스턴스를 생성합니다.

Attack 클래스는 Normal 클래스의 악용을 위해 Normal 클래스를 상속한 클래스입니다.

예시에서는 양수 value 값을 가진 Normal 인스턴스를 생성하고 

obj에 null을 넣으면서 GC의 대상이 되도록 만듭니다.

원래 Normal 객체의 finalizer는 아무 동작을 하지 않지만,

Normal 객체를 악용하기 위해 만든 Attack 객체의 finalizer는 obj를 다시 되살릴 뿐만 아니라,

그 객체가 음수인 value 값을 가지도록 만듭니다.

이는 매우 심각한 보안 문제로 자칫하면 프로그램 다운을 초래할 수 있습니다.

그래서 이를 막기 위해 finalizer를 final로 선언하여 finalizer 공격을 사전에 방어해야 합니다.

## 4. finalizer와 cleaner의 대체 방법

그렇다면 파일이나 스레드 등 종료해야 할 자원을 담고 있는 클래스에서

finalizer와 cleaner를 대신해줄 방법이 어떤게 있을까요?

바로, AutoCloseable이 있습니다.

AutoCloseable은 close 메서드를 통해 자동으로 객체를 소멸시키는 인터페이스입니다.

try-with-resources 구문에서는 AutoCloseable을 구현해야 사용할 수 있고,

try 내부 로직이 끝나면 자동으로 객체를 회수하는 역할을 합니다.

```java
  class Item implements AutoCloseable {
    @Override
    public void close() throws Exception {
        // clean 작업
    }
  }
```

## 5. finalizer와 cleaner의 사용처

이쯤이면 이런 생각이 듭니다.

> 쓰지 말라고 할꺼면 도대체 왜 만든거야?

finalizer, cleaner를 어떻게 활용하면 좋을까요?

첫 번째, 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할로 사용할 수 있습니다.

finalizer와 cleaner가 즉시 호출된다는 보장은 없지만,

자원 회수를 늦게라도 해주는 것이 안해주는 것보다 낫기 때문입니다.

```java
/* Cleaner 안전망 */
  import java.lang.ref.Cleaner;

  public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    private final Cleaner.Cleanable cleanable;
    private final State state;

    public Room(int numJunkPiles) {
      state = new State(numJunkPiles);
      cleanable = cleaner.register(this, state);
    }

    private static class State implements Runnable {
      int numJunkPiles;
      
      State(int numJunkPiles) {
        this.numJunkPiles = numJunkPiles;
      }

      // close 메서드나 cleaner가 호출
      @Override
      public void run() {
        numJunkPiles = 0;
      }
    }

    @Override
    public void close() throws Exception {
      cleanable.clean();
    }
  }
```

두 번째, 네이티브 피어와 연결된 객체에서 사용할 수 있습니다.

> 네이티브 피어
>
> 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체

네이티브 피어는 자바 객체가 아니기 때문에 GC는 그 존재를 알지 못합니다.

그렇기 때문에 finalizer나 cleaner를 통해 처리할 수 있습니다.

단, 성능 저하를 감당할 수 없거나, 자원을 즉시 회수해야 한다면 close 메서드를 사용해야 합니다.

## 6. 정리

이번 포스트에서는 finalizer와 cleaner의 단점을 통해 둘의 사용을 지양해야하는 이유를 알게 되었고,

그 대체 방법으로 AutoCloseable에 대해 알아보았습니다.

finalizer와 cleaner의 단독적 사용은 지양하지만,

안전망 또는 네이티브 자원 회수 역할에서는 효과가 있으므로

두 역할인 경우에는 사용해도 괜찮습니다.

하지만, finalizer와 cleaner는 성능 저하의 요인이 되기 때문에 

성능에 민감한 프로그램인 경우에는 사용을 지양하는게 좋을 것 같습니다.