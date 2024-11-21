## 1. 들어가기

다음은 예외를 잡지 못해 프로그램이 실패할 때 나타나는 Java 시스템의 스택 추적 정보입니다.

```java
  Exception in thread "main" java.lang.NoClassDefFoundError: org/springframework/boot/SpringApplication
    at com.sample.SampleApplication.main(SampleApplication.java:12)
  Caused by: java.lang.ClassNotFoundException: org.springframework.boot.SpringApplication
    at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:641)
    at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:188)
    at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:520)
    ... 1 more
```

이번에는 이 스택 추적에 대해 자세히 알아보겠습니다.

## 2. Java의 스택 추적

스택 추적은 예외 객체의 `toString` 메서드를 호출해 얻는 문자열로 보통은 예외 클래스 이름 뒤에 상세 메시지가 붙는 형태입니다.

이 정보는 실패 원인을 분석해야 하는 프로그래머 혹은 사이트 신뢰성 엔지니어(SRE)가 얻을 수 있는 유일한 정보인 경우가 많고, 더구나 그 실패를 재현하기 어려운 경우
더 중요한 정보입니다.

즉, 사후 분석을 위해 실패 순간의 상황을 정확히 포착해 예외 상세 메시지에 담아야 합니다.

```java
/* IndexOutOfBoundsException */
  public IndexOutOfBoundsException(int index) {
    super("Index out of range: " + index);
  }
```

하지만 그렇다고 장황하게 작성할 필요는 없습니다.

스택 추적에는 예외가 발생한 파일 이름과 줄 번호는 물론 스택에서 호출한 다른 메서드들의 파일 이름과 줄번호까지 정확히 기록되어 있습니다.

그러므로 문서와 소스 코드에서 얻을 수 있는 정보는 길게 늘어놔봐야 군더더기가 될 뿐입니다.

## 3. 주의해야 할 점

스택 추적은 주의해야 할 점이 있습니다.

바로 예외의 상세 메시지와 사용자에게 보여줄 오류 메시지를 혼동해서는 안된다는 점인데요.

사용자에게는 친절한 안내 메시지를 보여줘야 하는 반면, 예외 메시지는 문제를 분석해야 할 프로그래머와 SRE 엔지니어가 보는 것이기에 가독성보다는 담긴 내용이 훨씬 중요합니다.