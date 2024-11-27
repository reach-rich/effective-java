## 예외를 무시하지 말라

<br>

API 설계자가 메서드 선언에 예외를 명시하는 까닭은, 그 메서드를 사용할 때 적절한 조치를 취해달라고 말하는 것이다. 
API 설계자의 목소리를 흘려버리지 말자. 
```java
// 의심스러운 코드
try {

} catch (SomeException e) {
}
```
예외는 문제 상황을 잘 대처하기 위해 존재하는데 catch 블록을 비워두면 예외가 존재할 이유가 없어진다. 

<br>

물론 예외를 무시해야 할 때도 있다. 예를 들어 FileInputStream을 받을 때가 그렇다. 
파일의 상태를 변경하지 않았으니 복구할 것이 없으며, 필요하나 정보는 이미 다 읽었다는 뜻이니 남은 작업을 중단할 이유도 없다.

예외를 무시하기로 했다면 catch 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔놓도록 하자. 
```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4;
try {
  numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException } ExecutionException ignored) {
  // 기본값을 사용한다(색상 수를 최소화하면 좋지만, 필수는 아니다)
}
```
<br>
