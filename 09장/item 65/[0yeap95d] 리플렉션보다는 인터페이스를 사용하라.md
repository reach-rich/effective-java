### 📝 리플렉션의 기능

- 임의의 클래스에 접근할 수 있음
- 클래스 객체가 주어지면 그 클래스의 생성자, 메서드, 필드 인스턴스를 가져올 수 있음
- 가져온 인스턴스로 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등 가져올 수 있음
- 실제 생성자, 메서드, 필드를 조작할 수 있음
- 컴파일 당시 존재하지 않던 클래스도 이용할 수 있음

<br>

### 📝 리플렉션의 단점

- 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없음
- 리플렉션을 이용하면 코드가 지저분하고 장황해짐
- 리플렉션을 통한 메서드 호출은 일반 메서드보다 훨씬 느려서 성능도 떨어짐

<br>**✔ 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있음**

<br>

### 🔎 리플렉션 사용예시

리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조

**✏ #01 예제소스 | 리플렉션으로 생성하고 인터페이스로 참조해 활용한다**

```java
public static void main(Stringp[] args) {
    // 클래스 이름을 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) // 비검사 형변환!
            Class.forName(args[0]);
    } catch (ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }
    
    // 생성자를 얻는다.
    Constructor<> extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }
    
    // 집합의 인스턴스를 만든다.
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        ...
    } catch (InstantiationEception e) {
        ...
    } catch (InvocationTargetException e) {
        ...
    } catch (ClassCastException e) {
        ...
    }
    
    // 생성한 집합을 사용한다.
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
}
```

> - 런타임에 총 여섯 가지나 되는 예외를 던질 수 있음
>   - 리플렉션 없이 생성했다면 컴파일타임에 잡아낼 수 있는 예외들
>   - ReflectiveOperationException을 잡도록 하여 코드 길이를 줄일 수도 있음 (자바7부터 지원)
> - 클래스 이름만으로 인스턴스를 생성해내기 위해 25줄이나 되는 코드를 작성
>   - 리플렉션이 아니라면 생성자 호출 한 줄

<br>

**✔리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합**

- 버전이 여러 개 존재하는 외부 패키지를 다룰 때 유용
- 가장 오래된 버전만을 지원하도록 컴파일 한 후, 이후 버전의 클래스와 메서드 등은 리플렉션으로 접근하는 방식
- 같은 목적을 이룰 수 있는 대체 수단을 이용하거나 기능을 줄여 동작하는 등의 조치가 필요

<br>

### 📌 핵심정리

**리플렉션은 복잡한 특수 시스템을 개발할 때 필요한 강력한 기능이지만, 단점도 많다**

**컴파일타임에는 알 수 없는 클래스를 사용하는 프로그램을 작성한다면 리플렉션을 사용해야 할 것이다**

**단, 되도록 객체 생성에만 사용하고, 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스로 형변환해 사용해야 한다**

<br>

