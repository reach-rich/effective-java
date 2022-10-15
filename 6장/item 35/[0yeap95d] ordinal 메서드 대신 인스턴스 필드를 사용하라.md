### ordinal 메서드 대신 인스턴스 필드를 사용하라

<br>

**✏ #01 예제소스 | ordinal을 잘못 사용한 예**

```java
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
	
	pulic int numberOfMusicians() { return ordinal() + 1; }
}
```

> 상수 선언 순서 바꾸는 순간 `numberOfMusicians`가 오작동
>
> 이미 사용중인 정수와 값이 같은 상수 추가할 수 없음
>
> 값을 중간에 비워둘 수도 없음

<br>

**열거타입 상수에 연결된 값은 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장해라**

<br>

**✏ #02 예제소스**

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), NONET(9), DECTET(10), TRIPLE_QUARTET(12);
    
    private final int numberOfMusicians = size; 
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

>`ordinal`메서드는 `EnumSet`과 `EnumMap` 같이 열거 타입기반의 범용 자료 구조에 쓸 목적으로 설계되었다
>
>이런 용도가 아니라면 `ordinal` 메서드는 절대 사용하지 마라
