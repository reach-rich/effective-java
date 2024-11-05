# 적시에 방어적 복사본을 만들라

자바는 안전한 언어이기 때문에 C, C++ 에서 흔히 보이는 버퍼 오버런, 배열 오버런, 와일드포인터 와 같은 메모리 충돌 오류에서 안전하다.  
하지만 아무리 자바라고 해도 다른 클래스로부터의 침범을 아무런 노력 없이 막을 수 있는 건 아니기 때문에  
__클라이언트가 불변식을 얼마든지 깨뜨릴 수 있다 가정하고 방어적으로 프로그래밍 해야한다__

#
### 1. 내부 수정을 허락하는 경우
어떤 객체든 허락 없이 외부에서 수정하는 일은 불가능하다.  
하지만 주의를 기울이지 않으면 내부 수정을 허락하는 경우가 생긴다.

```java
import java.util.Date;

class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if(start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + " after " + end);
        }
        this.start = start;
        this.end = end;
    }
 
 public Date start() { 
    return start; 
 }
 
 public Date end() { 
    return end; 
 }

} 
```
- 클래스가 불변처럼 보일 수 있지만 Date가 가변이기 때문에 불변식이 깨질 수 있음

```java
Date start = new Date();
Date end = new Date();

Period p = new Period(start, end);
end.setYear(78); // p 내부 수정 ㅠ0ㅠ
```

- 자바 8 이후에서는 Date 대신 불변인 Instant 사용으로 해결 가능 (LocalDateTime, ZonedDateTime 도 가능)

> 🎯 __Date는 낡은 API이므로 더이상 사용하지 말자__

#
### 2. 내부 수정을 방지하는 방법
새로운 코드에서는 Date와 같이 내부 수정이 가능한 코드를 작성하지 않으면 되지만, 기존에 사용한 코드들이 있을 수 있다.  
그런 코드들에는 외부 공격으로부터 인스턴스의 내부를 보호하기 위한 방어적 복사를 해주자.

- 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사하기

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    // 유효성 검사 전에 복사해야 한다. 
    if(start.compareTo(end) > 0) {
        throw new IllegalArgumentException(start + " after " + end);
    }
}
```
- 매개변수의 유효성을 검사하지 전에 방어적 복사본을 만들고 복사본으로 유효성 검사를 진행 
  - 멀티스레드 환경에서 유형성 검사 전에 복사를 수행 시 다른 스레드가 원본 객체를 수정할 위험이 있음 (TOCTOU 공격)

- 매개 변수가 제3자에 의해 확장될 수 있는 타입이라면 clone 사용 시 악위적인 하위 클래스의 인스턴스를 반활할 수 있기 때문에 clone으로 복사 금지

<br>

이렇게 공격을 막아도 아직 Period 인스턴스는 공격 가능하기 때문에 __가변 필드의 방어적 복사본을 반환해__ 아래의 문제를 해결해야 한다.

```java
Date start = new Date();
Date end = new Date();

Period p = new Period(start, end);
p.end.setYear(78); // 또 p 내부 수정 ㅠ0ㅠ
```

```java
public Date start() {
  return new Date(start.getTime());
}

public Date end() {
  return new Date(end.getTime());
}
```
- 완볃한 불변의 상태!!
- 생성자와 달리 접근자 메서드에서는 방어적 복사에 clone을 사용 가능 (신뢰할 수 있는 하위 객체이기 때문)
- 하지만 그래도 일반적으로 생성자나 정적 팩터리를 사용하는 것이 좋음


#
### 3. 방어적 복사의 이유

- 매개변수를 방어적으로 복사하는 목적이 불변 객체를 만들기 위함은 아님
- 변경 될 수 있는 객체라면 임의로 변경되어도 그 클래스가 문제 없이 동작할지를 판단하고, 문제가 발생할 수 있다면 복사본을 만들어 저장해야 함
- 안심할 수 없다면 방어적 복사본을 반환하라!
- 호출자가 컴포넌트 내부를 수정하지 않으리라 확신한다면 방어적 복사를 생략하되, 수정하지 말 것을 명확히 문서화 하라!


