## 단위 테스트
작은 코드 단위를 독립적으로 검증하는 테스트로, 검증 속도가 빠르고 안정적이다
- 작은 코드 단위라는 것은 클래스나 메서드 단위
- 테스트 케이스를 세분화하자
	- 성공하는 해피 케이스
	- 예외가 발생하는 실패 케이스
- 경계값을 테스트하는 것이 중요하다
	- 범위, 구간, 날짜 등등

### 테스트하기 어려운 구간은 구분 및 분리하자
- 관측 시 마다 다른 값에 의존하게 되는 코드들
	- 날짜나 시간, 랜덤 값, 전역 변수나 함수, 사용자 입력 등등
- 외부 세계에 영향을 주는 코드들
	- DB 기록, 메시지 발송 등등

예를 들면 시간별로 결과가 달라지기 때문에 테스트하기가 어려운 코드
```java
public Order createOrder() {  
	LocalDateTime now = LocalDateTime.now();  
	LocalTime localTime = now.toLocalTime();  
  
	if (localTime.isBefore(SHOP_OPEN_TIME) || localTime.isAfter(SHOP_CLOSE_TIME)) {  
		throw new IllegalStateException("주문 가능 시간이 아닙니다. 관리자에게 문의하세요.");  
	}
}
```

시간을 외부에서 받는 것으로 분리
```java
public Order createOrder(LocalDateTime now) {
	LocalTime localTime = now.toLocalTime();  
  
	if (localTime.isBefore(SHOP_OPEN_TIME) || localTime.isAfter(SHOP_CLOSE_TIME)) {  
		throw new IllegalStateException("주문 가능 시간이 아닙니다. 관리자에게 문의하세요.");  
	}
}
```
