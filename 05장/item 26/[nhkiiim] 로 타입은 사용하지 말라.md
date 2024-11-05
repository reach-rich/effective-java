# 로 타입은 사용하지 말라

### 1. 제네릭 클래스 (제네릭 인터페이스)

- 클래스와 인터페이스 선업 타입에 매개변수가 쓰이면 이를 제네릭 클래스 혹은 제네릭 인터페이스라고 함
- List 인터페이스는 원소의 타입을 나타내는 타입 매개변수 E를 받는다 -> List<E>
- 제네릭 클래스와 제네릭 인터페이스를 통틀어서 `제네릭 타입` 이라고 한다.
  
- 매개변수화 타입
  - 각각의 제네릭 타입은 일련의 매개변수화 타입을 정의
  - 클래스 이름 <실제 타입 매개변수> 
  - ex) List<String>

- 제네릭 타입을 하나 정의하면 그에 딸린 로 타입도  함께 정의

#
### 2. 로 타입 (raw type) 
- 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때
- List<E>의 로 타입은 List!
- 로 타입은 타입 선언에서 제네릭 타입 정보가 전부 지워진 것 처럼 도ㅇ작
- 제네릭이 도래하기 전 코드와 호환되게 해주는 궁여지책(꾀)

<br>

- __제네릭 지원 전__
  
``` java
private final Collection stamps = ...;
stamps.add(new Coin(...));
```

- 이 코드를 사용하면 도장이든 동전이든 넣으면 실행 됨 (컴파일러가 모호한 경고 메세지를 보여주긴 함)
- 오류는 가능한 한 발생 즉시 컴파일 전에 발견해줘야함
- 이 예에서는 런타임시에나 오류를 발견할 수 있음

```java
for (Iterator i = stamps.iterator(); i.hasNext();) {
      Stamp stamp = (Stamp) i.next(); // ClassCastException을 던진다.
      stamp.cancel();
}
```
  
- __타입 안전성 확보__

``` java
private final Collection stamps = ...;
stamps.add(new Coin(...));
```

- 이렇게 선언하면 컴파일러는 Stamp의 인스턴스만 넣어야함을 인지
- 컴파일러는 원소를 꺼내는 모든 곳에서 보이지 않는 형변환을 추가해 절대 실패하지 않음을 보장한다
  
#
### 3. 로타입을 쓰지 마라
- 로타입을 쓰면 제네렉이 안겨주는 안전성과 표현력을 모두 일게 됨!
- 이런 친구를 왜 만들어서 위험하게 하는가 싶겠지만 호환성 떄문에 어쩔 수 없다
- 자바가 제네릭을 받아들이기까지 10년이 걸려서 제너릭 없는 코드가 세상을 덮어버림 (ㅇㅋ)

#
### 4. 써도 되는 매개변수화 타입
- List\<Object> 처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다!
- List는 제네릭을 아예 빼서 안되지만 List\<Object> 처럼 모든게 다 와도 돼는 가능하다

#
### 5. 와일드카드 타입
  
```java
static int numElementsInCommon(Set s1, Set s2) {
    int result = 0;
    for (Object o1 : s1)
      if (s2.contains(o1)) result++;
    return result;
}
```
- 이 메서드는 동작 하지만 로 타입을 사용해 안전하지 않다
  
<br>
  
- 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않다면 물음표를 사용하라 ???
- Set<?> 처럼 사용하면 어떤 타입이라고 담을 수 있는 가장 범용적인 상태가 된다

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

- 와일드카드 타입은 안전하고 로타입은 안전하지 않다!
- 와일드카드 타입을 쓰면 로 타입과 다르게 컴파일러가 제 할일을 한다

#
### 6. 로타입을 써도 되는 예외
__1) class 리터럴에는 로 타입을 써야한다__

- 자바는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다
  - OK : List.class, String[].class, int.class 
  - NO : List.class, List<?>.class

<br>

__2) instanceof 연산자에는 차라라 로 타입을 써라__
- instanceof 연산자는 비한정적 와일드카드 타입(=> ?를 의밈함) 이외의 매개변수화 타입에는 적용할 수 없다
 
```java
if (o instanceof Set) {
    Set<?> s = (Set<?>) o;
    ...
}
```
  
