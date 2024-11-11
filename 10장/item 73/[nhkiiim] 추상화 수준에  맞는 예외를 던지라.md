## 추상화 수준에  맞는 예외를 던지라
> 수행하려는 일과 관련없는 예외가 나오면 당황스럽다~!  
> 메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해버릴 경우 종종 일어나는 일인데 내부 구현방식을 드러내 API를 오염시킨다.  
> __상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던지자__

#
### 예외 번역
```java
try {
	... // 저수준 추상화를 이용
} catch (LowerLevelException e) {
	throw new HigherLevelException();
}
```

> AbstractSequentialList 예시
```java
public E get(int index) {
	ListIterator<E> i = listIterator(index);
	try {
		return i.next();
	} catch (NoSuchElementException e) {
		throw new IndexOutOfBoundsException("인덱스: " + index);
	}
}
```

#
### 예외 연쇄
- 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄(exception chaining) 사용하는게 좋다
- 예외 연쇄란 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식
- 별도의 접근자 메서드 (Throwable의 getCause 메서드)를 통해 언제든 꺼내볼 수 있다.
- 대부분의 표준예외는 예외 연쇄용 생성자를 가지고 있다

```java
  try {
  // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
  // 저수준 예외를 고수준 예외에 실어 보낸다
  throw new HigherLevelException(cause);
}

// 예외 연쇄용 생성자
class HigherLevelException extends Exception {
  HigherLevelException(Throwable cause) {
    super(cause);
  }
}
```

> __무턱대고 예외를 전파하는 것 보다 예외 번역이 우수한 방법이지만 그렇다고 남용해서는 안된다!__
- 가능하다면 저수준 메서드가 반드시 성공하도록하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다.
- 때로는 상위 계층의 메서드의 매개변수 값을 아래 계층 메서드에 건네기 전에 미리 검사하는 방법으로 예방할 수 있다,

> 차선책 : 아래 계층에서의 예외를 피할 수 없다면 상위 계층에서 조용히 처리하여 문제를 API 호출자에게까지 전달하지 않는 방법도 있다 -> logging 활용!!
