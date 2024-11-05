# ordinal 메서드 대신 인스턴스 필드를 사용하라

### 1. ordinal 메서드
- 대부분의 열거 타입 상수는 하나의 정수값에 대응
- 열거 타입은 해당 상수가 열거 타입에서 몇번째 위치인지를 반환하는 ordinal 메서드 제공

```java
//ordinal을 잘못 사용한 예 (따라하지 말 것)
public enum Ensemble{
  SOLO, DUET, TRIO, ... ;
  
  public int numberOfMusictians() {
    retrun ordinal() +1;
  }
}
```
- 동작은 하지만 유지보수가 끔 찍 한 코드
- 상수 선언 순서를 바꾸는 순간 오작동!

- 이미 사용중인 정수와 같은 값은 추가할 수 없음
- 꼭 중간에 숫자를 비우지 말고 다 사용해야함 (12를 추가하고 싶다면 11도 꼭 넣어야함)

#
### 2. 인스턴스 필드 사용
- 열거 타입 상수에 연결 된 상수값은 ordinal을 사용하지 말고 인스턴스 필드에 저장하자

```java
public enum Ensemble{
  SOLO(1), DUET(2), TRIO(3), ... ;
  
  private final int numberOfMusictians;
  
  Ensemble(int size) { 
    this.numberOfMusictians = size;
  }
  
  public int numberOfMusictians() {
    retrun numberOfMusictians;
  }
}
```

<br>

- Enum의 API 문서에도 ordinal은 프로그래머가 사용할 일이 없다 나와있다 (EnumSet, EnumMap 같은 자료구조에 쓸 목적으로 설계)
- 그러니 ordinal을 절대 사용하지 말자!
