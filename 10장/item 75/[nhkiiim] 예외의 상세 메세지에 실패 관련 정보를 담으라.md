## 예외의 상세 메세지에 실패 관련 정보를 담으라

- 예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 stack trace 정보를 자동으로 출력
- 스택 추적은 예외 객체의 `toString` 메서드를 호출해 얻는 문자열로 보통 에외 클래스의 이름 뒤에 상세메세지가 붙는 형태
- 실패 원인을 분석할 수 있는 유일한 정보인 경우가 많다
- __`toString` 메서드에 최대한 많은 정보를 담아 반환하자!__

#
### 예외 상세 메세지 내용
- 실패 순간을 포착하기 위해 예외에 관련된 모든 매개변수와 필드의 값을 실패 메시지에 담자
- IOBE은 범위와 최소 최대값을 출력할 수 있다
- __다만 보안에 관련된 정보는 주의헤서 다뤄야한다__
- 소스코드를 통해 알 수 있는 내용은 필요 없다
- 간결하게 작성하되, 핵심 내용을 모두 담아라
- 예외의 상세 메세지는 최종 사용자(고객)에게 보여줄 메세지와는 다르다는 것을 명심해라 (500에러입니다 하면 안됨 ㄱ-)


#
### 상세한 메세지를 담도록 IndexOutOfBoundsException 수정하기
```java
/**
 * IndexOutOfBoundsException을 생성한다.
 *
 * @param lowerBound 인덱스의 최솟값
 * @param upperBound 인덱스의 최댓값 + 1
 * @param index 인덱스의 실젯값
 */
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
   // 실패를 포착하는 상세 메시지를 생성한다.
   super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
   
   // 프로그램에서 이용할 수 있도록 실패 정보를 저장해둔다.
   this.lowerBound = lowerBound;
   this.upperBoudn = upperBound;
   this.index = index;
}
```
