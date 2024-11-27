## 예외를 무시하지 말라
> 너무 뻔한 조언 같지만 어기는 사람들이 많다!!  
> 메서드 선언에 예외를 선언하는 것은 메서드 사용 시 적절한 조치를 취하라는 것이다.  
> 예외를 무시하기가 쉬워서 그냥 넘어가는 경우가 많다.. ㅠㅠ

### 예외 무시하지 마..
- try catch문을 사용하면서 catch 블록에서 아무 일도 하지 않고 무시한다
- catch 블록을 비워두면 예외가 존재하는 이유가 없다

  
### 무시해야하는 경우
- FileInputStream을 닫을 때와 같은 경우가 있다
- 파일의 상태를 변경하지 않았으니 복구할 것이 없고, 정보를 모두 읽어 작업을 중단할 이유도 없기 때문!
- 파일을 닫지 못했다는 사실을 로그로 남겨두는 정도도 좋은 생각이다

### 무시할거라면..
- catch블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔두자!
```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4;
try {
  numColors = f.get(1L, TimeUtil.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
  // 기본값을 사용한다
}
```

> 검사/비검사 예외에 모두 적용되는 사항들이며, 예외를 무시해서 오류를 내재한 채 동작하게 하는 것보다 무시하지 않고 바깥으로 전파하는 것 만으로도 디버깅 정보를 남길 수 있다는 걸 기억하라
  
