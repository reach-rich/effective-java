### 🔎 가변인수

**: 가변인수(varargs) 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있음**

<br>

**가변인수 호출 시**

1. 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네줌
2. 입력받은 `int` 인수들의 합을 계산해주는 가변인수 메서드

<br>

**✏ #01 예제소스 | 간단한 가변인수 활용 예**

```java
static int sum(int... args) {
    int sum = 0;
    for (int arg : args)
        sum += arg;
    return sum;
}
```

>`sum(1, 2, 3)`은 `sum()`은 0을 돌려준다

<br>

- 인수가 1개 이상이어야 할 때도 있음

- 최솟값을 찾는 메서드인데 인수를 0개만 받을 수도 있도록 설계하는 건 좋지 않음

- 인수 개수는 런타임에(자동 생성된) 배열의 길이로 알 수 있음

<br>

**✏ #02 예제소스 | 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예**

```java
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
		if (args[i]) < min)
            min = args[i];
        return min;
}
```

>인수를 0개만 넣어 호출하면 (컴파일타임이 아닌) 런타임에 실패
>
>코드도 지저분함
>
>`args`유효성 검사를 명시적으로 해야 하고, `min`의 초깃값을 `Integer.MAX_VALUE`로 설정하지 않고는 (더 명료한) `for-each`문도 사용 불가

<br>

더 나은 방법으로는 매개변수를 2개 받도록 하는 것

**✏ #03 예제소스 | 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법** 

```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    returm min;
}
```

>첫번째로 평범한 매개변수를 받고, 가변인수는 두번 째로 받으면 위의 문제 해결 가능

<br>

- 가변인수는 인수 개수가 정해지지 않았을 때 아주 유용함
- `printf`는 가변인수와 한 묶음으로 자바에 도입되었고, 핵심 리플렉션기능도 재정비 되었음
- `printf`와 리플렉션 모두 가변인수의 수혜를 받고있음

<br>

**❗ 가변인수 사용 시 주의할 점**

- 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화
- 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 멋진 패턴이 있음

<br>

**📝 예시 | 메서드 호출의 95%가 인수를 3개 이하로 사용하는 경우**

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int... rest) {}
```

>다음처럼 인수가 0개인 것부터 4개인 것까지, 총 5개를 다중정의
>
>마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하는 것
>
>따라서 메서드 호출 중 단 5%만이 배열을 생성

**대다수의 성능 최적화와 마찬가지로 이 기법도 보통 때는 별 이득이 없지만, 꼭 필요한 특수 상황에서는 좋음!**

<br>

**📚 EnumSet**

- `EnumSet`의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다
- `EnumSet`은 비트필드를 대체하면서 성능가지 유지해야 하므로 아주 적절하게 활용한 예

<br>

---

<br>

### 📌 핵심정리

**인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다**

**메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자**
