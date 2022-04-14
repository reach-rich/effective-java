## 1. 들어가기

Singleton이 무엇일까요?

용어의 의미와 함께 어떻게 사용하는지 예시를 통해 알아봅시다.

## 2. Singleton

### 2-1. 의미

<br>
Singleton은 GoF 디자인 패턴 중 Singleton 패턴에서 유래했는데,

Wikipedia에서 Singleton 패턴에 대해서 찾아보면 다음과 같이 명시되어 있습니다.

> the singleton pattern is a software design pattern that restricts the instantiation of a class to one "single" instance.

그리고 이를 해석하면 다음과 같습니다.

> Singleton 패턴은 클래스의 "인스턴스를 하나로 제한"하는 소프트웨어 디자인 패턴

### 2-2. 예시

<br>
이번에는 Singleton을 만드는 방법에 대해서 알아볼까요?

Singleton을 만드는 방법은 세 가지가 있습니다.

<br>
 1\. public static final 필드 방식
<hr>

```java
  public class Printer {
    public static final Printer INSTANCE = new Printer();

    private Printer() {
      ...
    }
  }
```

해당 방법은 생성자의 접근 제한자를 private으로 두어 외부에서 객체를 생성하지 못하도록 막고,

static 키워드를 통해 프로그램 초기화 시에 단 한번만 객체가 생성되도록 하는 방법입니다.

하지만, Reflection API를 통해 클라이언트에서 private 생성자를 호출할 수 있는 예외가 있습니다.

```java
/* Main.class */
  public static void main(String[] args) 
    throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
      Printer printer = Printer.INSTANCE;
      Constructor<Printer> constructor = (Constructor<Printer>) printer.getClass().getDeclaredConstructor();
      constructor.setAccessible(true);
      Printer printer2 = constructor.newInstance();
      System.out.println(printer);
      System.out.println(printer2);
  }
```

* 결과

<img src="{{ site.url }}/assets/img/effective_java/item3/reflection.png" width="40%" height="30%" />

> * Reflection API
>
> 클래스의 정보에 접근할 수 있게 해주는 JAVA API

그래서 Reflection API 공격을 방어하기 위해서는

```java
  public class Printer {
    public static final Printer INSTANCE = new Printer();

    private Printer() {
      if(Objects.nonNull(INSTANCE)) {
        throw new RuntimeException();
      }
      ...
    }
  }
```

이런식으로 생성자에서 두 번째 객체가 생성되려할 때 예외를 던지면 됩니다.

public static final 필드 방식의 장점은 해당 클래스가 Singleton임이 API에 명백히 드러난다는 점과

한줄로 작성 가능하기 때문에 간결하다는 장점이 있습니다.

<br>
 2\. 정적 팩터리 방식
<hr>

```java
  public class Printer {
    private static final Printer INSTANCE = new Printer();

    private Printer() {
      ...
    }
    
    public static Printer getInstance() {
      return INSTANCE;
    }
  }
```

해당 방법은 정적 팩터리 메서드를 통해 Singleton으로 만든 방법입니다.

getInstance를 통해 항상 같은 객체를 반환하므로

단 하나의 인스턴스를 보장하지만,

public static final 필드 방식과 동일하게 Reflection API 예외가 있고

동일하게 생성자에서 두 번째 객체가 생성되려할 때 예외를 던지면 됩니다.

정적 팩터리 방식의 장점은 원할 때, API를 바꾸지 않고도 Singleton이 아니게 변경할 수 있습니다.

```java
  public class Printer {
    private static final Printer INSTANCE = new Printer();

    private Printer() {
      ...
    }
    
    public static Printer getInstance() {
      if(Thread.currentThread().getId() % 2 == 0) {
          return INSTANCE;
      } else {
          return new Printer();
      }
    }
  }
```

해당 예시는 첫 번째 예시를 Singleton이 아니게 변경한 예로

Thread의 아이디가 짝수이면 프로그램 초기화 시에 만들어진 객체를,

홀수이면 새로운 객체를 반환하도록 만든 예시입니다.

그 밖에, 정적 팩터리 메서드는 Generic Singleton으로 만들 수 있다는 점과

정적 팩터리 메서드 참조를 Supplier로 사용할 수 있다는 장점이 있습니다.

<br>
 3\. 열거 타입 방식
<hr>

세 번째 방법은 enum을 통해 Singleton을 만드는 다소 생소한 방법입니다.

```java
  public enum Printer {
    INSTANCE;
  }
```

public static final 필드 방식과 비슷하지만, 

더 간결하고, 추가 노력없이 직렬화할 수 있으며 Reflection 공격에도 새로운 인스턴스가 만들어지는 것을 막아줍니다.

하지만, 만들려는 Singleton이 enum 외의 클래스를 상속해야한다면 사용할 수 없습니다.

### 2-3. 장점 

<br>
Singleton을 사용하면 어떤 장점이 있을까요?

<br>
 1\. 단 하나의 인스턴스만 존재하기 때문에 메모리 낭비를 방지할 수 있다.
<hr>

최초 한번의 new 연산자를 통해서 고정된 메모리 영역을 사용하기에

추후 해당 객체에 접근할 때 메모리 낭비를 방지할 수 있습니다.

<br>
 2\. 속도 측면에서 장점이 있다.
<hr>

이미 생성된 인스턴스를 활용하기에 생성 비용이 없어 속도 측면에서 장점이 있습니다.

<br>
 3\. 다른 클래스 간 데이터 공유가 쉽다는 장점이 있다.
<hr>

Singleton은 전역으로 사용되는 인스턴스이기에

다른 클래스의 인스턴스들이 접근하여 사용할 수 있습니다.

### 2-4. 단점

<br>
그렇다면 Singleton의 단점은 무엇일까요?

<br>
 1\. 테스트하기 어렵다.
<hr>

Singleton은 자원을 공유하고 있기에 격리된 환경에서 수행되려면

매번 인스턴스의 상태를 초기화해줘야 하는 단점이 있습니다.

<br>
 2\. 동시성 문제가 발생한다.
<hr>

여러 인스턴스가 하나의 Singleton 인스턴스를 바라보기에

Multi-Thread 환경에서 각 Thread들이 서로 다른 인스턴스를 생성할 위험이 있습니다.

## 3. 정리

이번 포스트에서는 Singleton의 의미와 함께 장단점, Singleton을 생성하는 3가지 방법을 알아보았습니다. 

그 중, 열거 타입 방식의 Singleton은 간결하지만,

직렬화할 수 있고 Reflection 공격을 막아주기 때문에

대부분의 상황에서 사용하기에 적합한 방법입니다.

하지만, Singleton이 enum 외의 클래스를 상속해야 한다면

다른 방법을 사용해야 합니다.

## 4. 참고

* <a href="https://tecoble.techcourse.co.kr/post/2020-11-07-singleton/" target="_blank">https://tecoble.techcourse.co.kr/post/2020-11-07-singleton/</a>

* <a href="https://jeong-pro.tistory.com/86" target="_blank">https://jeong-pro.tistory.com/86</a>