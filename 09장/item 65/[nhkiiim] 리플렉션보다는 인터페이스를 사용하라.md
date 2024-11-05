## 리플렉션보다는 인터페이스를 사용하라

- 리플렉션 기능 (java.lang.reflect)을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다.
- Class 객체가 주어지면 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있다
- 인스턴스에서 그 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다
- 심지어 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수도 있다
  

> https://tjdtls690.github.io/studycontents/java/2023-01-27-reflection01/

#
### 🫒 리플렉션의 단점

- 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있어서 컴파일타임 타입 검사 및 예외 검사를 이용할 수 없다
  - 런타임 오류가 되어버림
 
- 리플렉션을 이용하면 코드가 지저분하고 장황해진다
- 성능이 떨어진다 (훨씬 느리다 - int 반환 기중 11배 느림) 


#
### 🫒 리플렉션 주의해서 사용하기
- 코드 분석 도구나 의존관계 주입 프레임워크처럼 리플렉션을 사용해야하는 복잡한 어플리케이션이 몇가지 있다
  - (하지만 이들 마저 지양하고 있음)
- 리플렉션은 아주 제한된 형태로만 사용해야함

  <br>

> ex) 컴파일타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 적절한 인터페이스나 상위 클래스를 이용할 수 있을 것
- 리플렉션은 인스턴스 생성에만 쓰고, 만들어진 인스턴스는 인터페이스나 사우이 클래스로 참조해 사용하자

```java
public static void main(String[] args) {
    // 클래스 이름을 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) // 비검사 형변환
            Class.forName(args[0]);
    } catch (ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }
    
    // 생성자를 얻는다.
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }
    
    // 집합의 인스턴스를 만든다.
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalArgumentException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스턴스화할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Set을 구현하지 않은 클래스입니다.");
    }
    
    // 생성한 집합을 사용한다.
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg) {
    Sytstem.err.println(msg);
    System.exit(1);
}
```

__위의 예시에서 볼 수 있는 리플렉션의 단점__

- 런타임에 최대 6가지의 예외를 던질 수 있다
- 클래스 이름만으로 인스턴스를 생성하기 위해 무려 25줄이나 되는 코드를 작성했다


#
### 🫒 마무리

- 드물긴 하지만 리플렉션은 런타임에 존재하지 않을 수고 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다
- 여러개 존재하는 외부 패키지를 다룰 때 유용하다 (가동 가능한 최소한의 환경, 가장 오래된 버전, 만을 준비해 컴파일한 후 리플렉션으로 접근)

> 특수 시스템을 개발할 때 필요한 강력한 기능이지만 단점도 많다. 컴파일타임에 알 수 없는 클래스를 사용할 경우 사용하되,
>  생성한 객체를 이용할 때는 적절한 인터페이스나 상위클래스로 형변환해 사용하자
