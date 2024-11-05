## 1. 들어가기

Java의 라이브러리는 close 메서드를 호출해 직접 닫아줘야 하는 자원들이 많습니다.

대표적으로 InputStream, OutputStream, java.sql.Connection 등이 있습니다.

하지만, 클라이언트에서 해당 자원 닫기를 놓치기 쉬워 성능 문제로 이어지곤 합니다.

그렇다면 어떤 방법으로 이를 해결할 수 있는지 알아봅시다.

## 2. try-finally 구문

전통적인 방법으로 try-finally 구문을 사용했습니다.

### 2-1. 사용법

먼저, try-finally 구문의 사용법에 대해 알아봅시다.

```java
    InputStream input = new FileInputStream("test.txt");
    try {
        // 예외가 발생할만한 코드
    } finally {
        input.close();
    }
```

try-finally 구문의 사용법은 매우 간단합니다.

try 문 내부에 예외가 발생할만한 코드를 넣고, finally 문에서는 사용한 자원들을 닫아주면 끝입니다.

너무 쉽죠?

### 2-2. 문제점

간단하고 아무 문제 없어 보이는 try-finally 구문에는 문제점이 있습니다.

예시를 통해 알아보겠습니다.

```java
/* Exception */
    public class FirstException extends Exception {}
    public class SecondException extends Exception {}

/* Resource */
    public class TestResource implements AutoCloseable {
        public void doSomething() throws FirstException {
            System.out.println("Do Something");
            throw new FirstException();
        }

        @Override
        public void close() throws SecondException {
            System.out.println("Close");
            throw new SecondException();
        }
    }

/* Client */
    TestResource resource = new TestResource();
    try {
        resource.doSomething();
    } finally {
        resource.close();
    }
```

doSomething 메서드는 FirstException을 발생시키고,

close 메서드는 SecondException을 발생시킵니다.

이를 실행하면 어떤 결과가 나올까요?

분명, 많은 분들이 FirstException 예외가 발생한 후, SecondException 발생하리라 짐작할 것입니다.

하지만 실제 결과는 다음과 같습니다.

```
    Do Something
    Close
    Exception in thread "main" Main$SecondException
        at Main$TestResource.close(Main.java:19)
        at Main.main(Main.java:28)
```

"Do Something"이 출력 값으로 나온 것을 보아 분명, doSomething 메서드를 호출했는데

FirstException 예외는 출력되지 않고 SecondException 예외만 출력된 것을 볼 수 있습니다.

이는 SecondException 예외가 FirstException 예외를 완전히 집어삼켜버린 문제로

스택 추적 내역에 FirstException 예외가 남지 않아

실제 시스템에서의 디버깅을 몹시 어렵게 하는 문제점이 있습니다.

그럼 도저히 방법은 없는걸까요?

이를 보완하기 위해 Java 7의 try-with-resources 구문을 사용하기 시작했습니다.

## 3. try-with-resources 구문

### 3-1. 사용법

먼저, try-with-resources 구문의 사용법을 알아봅시다.

```java
    try(InputStream input = new FileInputStream("test.txt")) {
        // 예외가 발생할만한 코드
    }
```

코드를 살펴보면 try-finally 구문에 비해 코드량이 눈에 띄게 줄은 것을 볼 수 있습니다.

try-with-resources 구문은 매개변수로 닫아야 하는 자원을 넣어주면

try 문 내부의 코드를 모두 실행한 후 자동으로 close 메서드를 실행해 자원을 닫아줍니다.

이 때, close 메서드는 AutoCloseable 인터페이스의 close 메서드를 재정의해서 사용합니다.

### 3-2. try-finally 구문의 문제점과 비교

try-finally에서 발생한 SecondException 예외가 FirstException 예외를 집어삼킨 문제점은

try-with-resources 구문에서는 해결했을까요?

예시를 통해 알아봅시다.

```java
/* Exception */
    public class FirstException extends Exception {}
    public class SecondException extends Exception {}

/* Resource */
    public class TestResource implements AutoCloseable {
        public void doSomething() throws FirstException {
            System.out.println("Do Something");
            throw new FirstException();
        }

        @Override
        public void close() throws SecondException {
            System.out.println("Close");
            throw new SecondException();
        }
    }

/* Client */
    try(TestResource resource = new TestResource()) {
        resource.doSomething();
    }
```

Exception과 Resource의 코드는 try-finally 구문에서 사용한 예시와 동일하고

Client 코드만 try-with-resources 구문으로 변경했습니다.

```
    Do Something
    Close
    Exception in thread "main" Main$FirstException
        at Main$TestResource.doSomething(Main.java:13)
        at Main.main(Main.java:26)
        Suppressed: Main$SecondException
            at Main$TestResource.close(Main.java:19)
            at Main.main(Main.java:25)
```

결과를 보면 try-finally 구문과 달리 가장 먼저 발생한 FirstException 예외가 최상단에 출력되고

아래에 숨겨졌다는 Suppressed 키워드를 통해 SecondException 예외가 출력되는 것을 볼 수 있습니다.

## 4. 정리

이번 포스트에서는 사용한 자원을 닫는 2가지 방법을 알아보았습니다.

그 중, 전통적 방법인 try-finally 구문은 여러 예외가 발생했을 때,

마지막 예외가 다른 예외들을 집어삼키는 문제점이 있었습니다.

이는 Java 7의 try-with-resources 구문을 통해 보완할 수 있었습니다.

try-with-resources 구문은 try-finally 구문보다 간결하고,

시스템 디버깅이 쉬워 예외를 쉽게 찾을 수 있는 장점이 있기 때문에

사용한 자원을 닫고 싶을 때는 try-finally 구문보다 try-with-resources 구문을 사용해야 합니다.