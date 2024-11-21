## 예외의 상세 메시지에 실패 관련 정보를 담으라

<br>

사후 분석을 위해 실패 순간의 상황을 정확히 포착해 예외의 상세 메시지에 담아야 한다. 

실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다. 

예를 들어 IndexOutOfBoundsException의 상세 메시지는 범위의 최솟값과 최댓값, 그리고 그 범위를 벗어났다는 인덱스의 값을 담아야 한다.

<br>

관련 데이터를 모두 담아야 하지만 장황할 필요는 없다. 

예외의 상세 메시지와 최종 사용자에게 보여줄 오류 메시지를 혼동해서는 안 된다. 
최종 사용자에게는 친절한 안내 메시지를 보여줘야 하는 반면, 예외 메시지는 가독성보다는 담긴 내용이 더 중요하다. 

<br>

실패를 적절히 포착하려면 필요한 정보를 예외 생성자에서 모두 받아서 상세 메시지까지 미리 생성해놓는 방법도 괜찮다. 
```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
  // 실패를 포착하는 상세 메시지를 생성한다.
  super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));

  this.lowerBound = lowerBound;
  this.upperBound = upperBound;
  this.index = index;
}
```

<br>

예외는 실패와 관련한 정보를 얻을 수 있는 접근자 메서드를 제공하는 것이 좋다. 

<br>
