# section3. 엔티티 매핑
## ✔︎ 객체와 테이블 매핑
### @Entity
- JPA가 관리하는 객체로 만들기 위해서는 클래스 명 위에 `@Entity`를 붙여서 표시한다.
- 매핑된 클래스는 기본 생성자가 필수이고, DB에 저장할 필드(컬럼)는 final이 붙어서는 안 된다.
- 기본적으로는 클래스 명이 곧 DB의 테이블 명이 되지만, 테이블명과 클래스명이 다를 경우 name이라는 옵션을 사용해서 이름을 맞춰 줄 수 있다.

## ✔︎ 필드와 컬럼 매핑
![필드_컬럼](https://github.com/user-attachments/assets/3fdc2716-33e8-44c0-9b4e-bb991bdd0b0d)
- 자바 객체의 필드와 DB의 컬럼을 매핑하는 여러 어노테이션들이 존재한다.
- `@Temporal`로 날짜 타입을 매핑할 수 있는데, db는 날짜. 시간, 둘다 포함된 것을 다 구분하기 때문에 매핑할 정보가 필요하다 
	- 자바 8 이상이라면 DateTime을 쓰냐, Date 타입을 쓰냐에 따라서 알아서 만들어주기 때문에 아예 어노테이션 자체를 생략 가능. 하지만 만약에 어노테이션을 쓰는 경우라면 꼭 매핑할 값을 넣어 주자
- enum 타입을 매핑할 때 사용하는 `@Enumerated`는 사용할 때 꼭 String으로 설정하자. (`value = EnumType.String `사용)
	- 기본값은 Ordinal임을 주의
	- Ordinal을 사용할 경우 무언가 enum 타입 내에 추가되어 순서가 바뀌거나 하는 경우 매칭이 이상해져 문제가 되기 쉽다.

### @Column
![컬럼](https://github.com/user-attachments/assets/12800e92-dfb3-414c-bf58-32f5542c8862)

## ✔︎ 기본 키 매핑
기본 키를 매핑에 사용하는 어노테이션은 `@Id`와` @GeneratedValue`가 있다.

### @Id
- pk를 나타내는 필드에 적용한다.
- 직접 아이디를 할당해서 사용하는 경우에는 이것만 사용

### @GeneratedValue
- pk를 자동으로 생성하도록 하는 경우 사용
- 어떤 전략으로 아이디를 생성할지 4개의 옵션이 있다. (AUTO는 DB 방언에 따라 디폴트 값을 따라감)
#### IDENTITY
- 기본 키 생성을 DB에 위임
- MySQL, PostgreSQL 등등에서 사용
	- AUTO_INCREMENT (MySQL)
- DB에 INSERT를 실행 한 이후에 ID 값을 알 수 있다는 특징이 있다.
	- 따라서 해당 전략을 사용하는 경우 persist만 해도 실제 INSERT 쿼리가 실행됨
#### SEQUENCE
- 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트인 데이터베이스 시퀀스를 사용
- `@SequenceGenerator`를 사용하여 테이블마다 따로 시퀀스를 둘 수 있음
- persist를 할 때, 시퀀스를 호출해 다음 값을 가져와서 키로 설정함
	- IDNTITY 전략과는 다르게 이 때 INSERT 쿼리는 실행되지 않는다
- `allocationSize` 옵션을 이용하여 한 번 시퀀스를 호출할 때 값을 몇 개씩 가져올지 설정 가능
	- 단 이렇게 미리 가져온 값은 서버가 종료되면 사라진다는 점 유의
#### TABLE
- 키 생성 전용 테이블을 만들어서 데이터베이스 시퀀스처럼 사용하는 전략
- 모든 DB에 구애받지 않고 적용 가능
- SEQUNCE처럼 `allocationSize`을 이용하여 동일하게 값을 몇 개씩 가져올지 설정 가능