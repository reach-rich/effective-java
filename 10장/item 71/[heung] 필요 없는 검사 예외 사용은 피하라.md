## 필요 없는 검사 예외 사용은 피하라

검사 예외를 제대로 활용하면 API와 프로그램의 질을 높일 수 있다.
발생한 문제를 프로그래머가 처리하여 안정성을 높이게끔 해준다. 

<br>

하지만, 검사 예외를 과하게 사용하면 오히려 쓰기 불편한 API가 된다. 
API를 제대로 사용해도 발생할 수 있는 예외이거나, 프로그래머가 의미 있는 조치를 취할 수 있는 경우라면 이 정도 부담은 받아들일 수 있을 것이다.
그러나 둘 중 어디에도 해당되지 않는다면 비검사 예외를 사용하는게 좋다.

<br>

검사 예외가 프로그래머에게 지우는 부담은 메서드가 단 하나의 검사 예외만 던질 때가 특히 크다.
오직 그 예외 하나 때문에 try-catch 블록을 추가해야 하기 때문이다.
이런 상황에서는 검사 예외를 던지지 않는 방법이 있는지 고민해봐야 한다.

<br>

검사 예외를 회피하는 가장 쉬운 방법은 적절한 결과 타입을 담은 옵셔널을 반환하면 된다.

<br>

또 다른 방법은 검사 예외를 던지는 메서드를 2개로 쪼개 비검사 예외로 바꿀 수 있다. 
이 방식에서 첫 번째 메서드는 예외가 던져질지 여부를 boolean 값으로 반환한다.

<br>

```java
try {
  obj.action(args);
} catch (TheCheckedException e) {
  ... // 예외 상황에 대처한다.
}
```

<br>

리팩터링하면 다음처럼 된다.
```java
if (obj.actionPermitted(args)) {
  obj.action(args);
} else {
  ... // 예외 상황에 대처한다.
}
```
이 리팩터링을 모든 상황에 적용할 수는 없지만 적용할 수 있다면 더 쓰기 편한 API를 제공할 수 있다.

<br>

그러나 주의할 점이 있이 있는데, actionPermitted는 상태 검사 메서드에 해당하므로 동시에 접근하거나 외부 요인에 의해 상태가 변할 수 있다면 이 리팩터링은 적합하지 않다.
또한 actionPermitted가 action 메서드이 일부를 중복 수행한다면 성능에서 손해이니 적절하지 않을 수 있다.
