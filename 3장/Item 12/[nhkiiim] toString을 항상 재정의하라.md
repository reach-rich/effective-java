# toString을 항상 재정의하라

### 1. toString은 재정의하는게 국룰.(진지)
- Object의 기본 toString 메서드가 우리가 작선한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다 (당연한 말)
- 이 메서드는 단순히 클래스 이름을 16진수로 변환한 해시코드를 반환하게 된다
- toString 일반 규약에 따르면 `간결하면서도 사람이 읽기 쉬운 형태의 유익한 정보`를 반환해야 함

- toString의 규약 : 모든 하위 클래스에서 메서드를 재정의하라 -> 새겨 들으시오!!!!!!!!!

<br> 

- 앞의 아이템인 equals와 hashCode를 재정희 하는 것 만큼 중요하진 않음 (하지만 얘네는 노잼)
- toString은 디버깅도 쉬워지고 직관적인 변화를 가져와서 좋다
- toString 메서드는 println, +, assert 구문에 넘길 때, 디버거가 객체 출력할 때 자동으로 호출 됨
- 우리들이 직접 호출하지 않아도 어디선가는 쓰일 수 있음 (로깅할 때도 굳 쓰레기 메세지 시러~)

- 객체 출력할 때 {나현=PhoneNumber@abdb} 보다 {나현=010-1111-2222} 이게 더 행복하지?
- 근데 막 전화번호책처럼 몇만개 가지고 있거나 Tread[main,5,main]과 같이 요약 정보를 담아야 한다면 좀 무리가 있다


#
### 2. toString 재정의 시 고려해 보세용
- toString의 반환값을 문서화하기
  - 아주 중요한 선택이다 (very important choice)
  - 전화번호나 행렬같은 값은 문서화 하기를 권함 (CSV파일로 바꾸면 유용)

- 포맷을 명시하기로 했다면 면시한 포맷에 맞는 문자열과 객페를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해줘도 좋다 ^0^
  - BigInteger, BigDecimal과 대부분의 기본 타입 클래스가 많이 따르는 방식
  - 단점 : 포맷을 한번 명시하면 평생 그 포맷에 얽매이게 된다
  - 개발자들이 그 포맷에 맞춰서 파싱하고 새로운 객체를 만들기 때문!!
  - 어쩌면 명시하지 않는게 유연성을 위해 좋을지도..
  
- 아무튼 포맷을 명시하든 아니든 의도를 명확하게 밝히자
  - 명시하려면 아주 정확하게 해야한다!
  - 설명도 자세하게 적어준다 (포맷 안적으면 나중에 포맷 바꿔도 개발자 책임 푸핰!)

#
### 3. 주의 사항
- 포맷 명시와 상관 없이 toString이 반환값에 포함된 정보를 얻어올 수 있는 API를 제공하자!
- 정보를 모르면 toString의 반환값을 파싱해야한다.. (진짜로 필요하다)

- 성능도 나빠지고 불필요한 작업이기에 향후 시스템이 망가지는 원인이 될 수 있음


#
### 4. 재정의 할 필요 없는 경우도 있긴 허다~!! 
- 정적 유틸리티 클래스는 toString을 제공할 이유가 없음
- 열거타입도 필요 없음 (자바가 제공)
- 하지만 하위 클래스가 공유해야 할 문자열 표현이 있는 추상 클래스라면 재정의 


