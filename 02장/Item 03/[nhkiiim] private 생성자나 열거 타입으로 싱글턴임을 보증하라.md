# private 생성자나 열거 타입으로 싱글턴임을 보증하라

### 1. 싱글턴
- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- ex) 함수 같은 무상태 객체
- ex) 설계상 유일해야하는 시스템 컴포넌트

       무상태 객체 : 함수에 인자로 전달 된 데이터를 변경해도 무상태, 불변성이 유지되어야 함

__클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려움!__

<br> `🧐: (왜????)-> 인스턴스가 하나만 생성되니까 테스트할 때 분리 불가능, 테스트 할거면 할 때마다 초기화 해야됨`


</span>
 

#
### 2. 싱글턴을 만드는 방식
- 두 방식 모두 생성자는 private으로 감춰두고 인스턴스에 접근할 수 있는 유일한 방법으로 public static 멤버 만들어둠 
 
`🧐: 두 방식의 정확한 차이는 ..? -> 필드와 메서드의 차이`

### (1) public static final 필드 방식의 싱글턴
```java
public class Elvis{
  public static final Elvis instance = new Elvis();
  private Elvis(){...}
  
  public void leaveTheBuilding(){...}
}
```
- private 생성자는 인스턴스 초기화 시 한번만 호출
- AccessiblrObject.setAccessible 이라는 리플렉션 API 아이템 쓰면 private 생성자를 호출 가능하다고 함
  - 이거까지 방어하려면 두번째 객체 생성시 예외 던지자
  
- 장점) 해당 클래스가 싱글턴인게 API에 명백히 드러남 
- 장점) 간결함

#

### (2) 정적 팩터리 메서드 방식의 싱글턴
```java
public class Elvis{
  private static final Elvis instance = new Elvis();
  private Elvis(){...}
  public static Elvis getInstance {return instance}
  
  public void leaveTheBuilding() // 빌딩은 왜 자꾸 더남 엘비스가 뭐지
}
```
- Elvis.getInstance가 항상 같은 객체의 참조를 반환
- 리플렉션을 통한 예외는 동일

- 장점) API를 바꾸지 않고 싱글턴이 아니게 변경 가능 `🧐: 여기서 말하는 API라는게 정확히 뭐죠..? -> 메서드 정도로 이해하기 -> 밖에서 사용자가 봤을 때 메서드 차제는 바뀌지 않음`
  - 유일한 인스턴스만 반환하다 맘 바뀌면 다 다른거 주게 바꿀 수 있음
- 장점) 원한다면! 정적 팩터리를 제네릭 싱글턴 패턴으로 만들기 가능 `🧐: 먼소리조 1 -> 진흥이오빠거 참조`
- 장점) 정적 팩터리의 메서드 참조를 공급자로 이용 가능 `🧐: 먼소리조 2 -> 자바가 기본적으로 제공해주는 supplier 함수형 인터페이스 찾아보기`
<br>

__이런 장점들이 필요하지 않다면 public 필드 방식을 쓰도록__

#

### # 두 방식의 싱글턴 클래스를 직렬화하기 `🧐: 먼소리조 3`
- 단순히 Serialize를 구현한다고 선언하는거로 ㄴ부족해
- 모든 인스턴스를 일시적(transient)이라 선언하고 readResolve 메서드를 제공해야함 `🧐: 다 먼소리인지.. -> 역직렬화 시 readResolve 자동 호출`

<br>

__이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어짐__

      인스턴스 직렬화 : https://pathas.tistory.com/151

<br>

- 싱글턴임을 보장해주는 메서드
```java
private Object readResolve(){
//진짜 엘비스를 반환하고 가짜 엘비스는 가비지 컬랙터에게 버리기
  return instance;
 }
```

#

### (3) 열거 타입 방식의 싱글턴 - 바람직한 방법 `🧐: 이해 필요 -> enum 자체가 상수 만드는거니까 싱글턴이랑 같은 효과를 `
```java
private enum Elvis{
  instance;
  
  public void leaveTheBuilding() {...}
 }
}
```
- public 방식과 유사하지만 더 간결한 방식
- 추가적인 노력 없이 직렬화 가능
- 복잡한 직렬화 상확이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 걸 막아줌

<br>

__부자연스러워 보일 순 있지만 대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법!__

- 단 만들려는 싱글턴 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용 불가 (열거 타입이 다른 인터페이스픞 구현하도록 선언할 순 있음)


