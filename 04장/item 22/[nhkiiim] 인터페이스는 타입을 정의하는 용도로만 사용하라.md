# 인터페이스는 타입을 정의하는 용도로만 사용하라

### 1. 인터페이스의 용도를 기억해 
- 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할

- 클래스가 어떤 인터페이스를 구현한다? 인스턴스로 뭘 할 수 있는지 클라이언트에게 말하는 것!
- 오직 이 용도로만 사용하도록 하자

#
### 2. 상수 인터페이스 (사용 금지!!!!!!!!)
- 인터페이스의 용도와 다른 인터페이스!
- 메서드 없이 static final 필드로만 가득 차있음

- 이 상수들을 사용하려는 클래스에서 정규화된 이름을 쓰는 것을 피하고자 구현하곤 함

```java
public interface PhysicalConstants{
  static final double KIMNAHYEON_N1 = 159;
  static final double KIMNAHYEON_N2 = 980514;
  static final double KIMNAHYEON_N3 = 444.444;
}
```

- 이것은 인터페이스를 잘못 사용한 예!
- 사용자에게 혼란을 주고 내부 구현을 노출시키는 나쁜 예!
- final이 아닌 클래스가 상수 인터페이스를 구현해버리면 모든 하위클래스의 이름 공간이 상수들로 오염됨

- 절대 따라하지 마시오

<br>

- __상수 제대로 쓰기__
  - 클래스나 인터페이스 자체에 추가하라!
  - PhysicalConstants.KIMNAHYEON_N1 이렇게 클래스 이름까지 명시해서 써라
  - 자주 사용하는 상수라면 static import 해도 좋다

```java
public class PhysicalConstants{
  private PhysicalConstants(){} //인스턴스화 방지
  
  public static final double KIMNAHYEON_N1 = 159;
  public static final double KIMNAHYEON_N2= 980514;
  public static final double KIMNAHYEON_N3 = 444.444;
  
}  
```
