## 가변인수는 신중히 사용하라

가변인수(varargs) 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.  
가변인수 메서드를 호출하면, 인수의 개수와 길이가 같은 배열을 만들고 인수들을 배열에 저장해 가변인수 메서드에 건네준다. 

#
### 가변인수 예시

#### int 인수들의 합을 계산해주는 가변인수 메서드
> `sum(1,2,3)`은 6 반환, `sum()`은 0 반환
```java
static int sum(int... args) {
  int sum = 0;
  for (int arg : args) {
    sum += arg;
  }
  
  return sum;
}
```

<br>

#### 인수가 1개 이상이어야 하는 경우 (별로인 방법)
> 컴파일 타임이 아닌 런타임에 실패하며, 코드도 지저분하게 인수를 1개로 제한
```java
static int min(int... args) {
  if (args.length == 0) {
    throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
  }
  
  int min = args[0];
  
  for (int i = 1; i < args.length; i++) {
    if (args[i] < min) {
      min = args[i];
    }
  }
  
  return min
}
```
<br>

#### 가변인수를 제대로 활용한 인수가 1개 이상인 메서드
> 매개변수 2개를 받이시영 (for-each문도 사용가능)

```java
static int min(int firstArg, int remainingArgs) {
  int min = firstArg;
  
  for (int arg : remainingArgs) {
    if(arg < min) {
      min = arg;
    }
  }
  
  return min
}
```

#
### 다중정의(오버로딩)으로 대체하기
> 가변 인수는 인수 개수가 정해지지 않았을 때 매우 유용하게 사용 가능

> 하지만 성능에 민감한 상황이라면, 호출할 때마다 배열을 새로 할당하고 초기화해 성능에 걸림돌이 될 수 있음

해당 메서드 호출의 95%가 인수를 3개 이상으로 사용한하는 것과 같은 특수 상황에서는 다중정의로 대체해 성능 최적화 가능
 
```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

- EnumSet의 정적 팩터리도 위와 같은 기법을 사용해 열거 타입 집합 생성 비용 최소화


#
### 핵심 정리

인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하지만, __필수 매개변수를 가변인수 앞에 사용하고 성능 문제까지 고려해 사용하자__
