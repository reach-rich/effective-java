# ordinal 메서드 대신 인스턴스 필드를 사용하라

### 1. ordinal 메서드
- 대부분의 열거 타입 상수는 하나의 정수값에 대응
- 열거 타입은 해당 상수가 열거 타입에서 몇번째 위치인지를 반환하는 ordinal 메서드 제공

```java
//ordinal을 잘못 사용한 예
public enum Ensemble{

}
```
