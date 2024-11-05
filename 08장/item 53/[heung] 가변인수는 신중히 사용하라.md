Effective Java의  53번째 아이템 "가변인수는 신중히 사용하라"를 읽고 정리한 내용을 포스팅합니다.

<br>

## 1. 가변인수 메서드

가변이수(varargs) 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다. 

인수가 1개 이상이어야 할 때도 있을 것이다. 그땐 다음과 같이 구현하지 않도록 주의해야 한다.

```java
static int min(int... args) {
  if (args.length == 0) {
    throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
  }
  int min = args[0];
  for (int i=1; i<args.length; i++) {
    if (args[i] < min) {
      min = args[i];
    }
  }
  return min;
}
```

이 방식에는 문제가 몇 개 있는데, 가장 심각한 문제는 인수를 넣지않고 호출하면 컴파일타임이 아닌 런타임에 실패한다는 점이다. 또 다른 문제로는 args 유효성 검사를 직접 해줘야 해서 코드도 지저분해진다.

훨씬 나은 방법이 있는데, 다음 코드처럼 매개변수를 2개 받도록 하면 된다. 이러면 앞서의 문제가 말끔히 사라진다. 

```java
static int min(int firstArg, int... remainingArgs) {
  int min = firstArg;
  for (int arg : remainingArgs) {
    if (arg < min) min = arg;
  }
  return min;
}
```

<br>

## 2. 가변인수 메서드 주의사항

성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다. 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문이다. 다행히 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 멋진 패턴이 있다. 바로, 다중정의를 이용하는 것이다. 

예를 들어 foo라는 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 해보자. 그렇다면 다음처럼 인수가 0개인 것부터 4개인 것까지, 총 5개를 다중정의하고, 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하도록 하는 것이다. 

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int... rest) {}
```

실제로 EnumSet 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다. 

<br>

## 3. 핵심 정리

* 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다.
* 메서드를 정의할 때 필수 매개변수는 가변인 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자. 
