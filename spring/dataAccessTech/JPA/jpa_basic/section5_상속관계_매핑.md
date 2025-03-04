# section5. 상속관계 매핑
RDB가 상속 관계를 지원하는 것은 아니지만, 슈퍼타입-서브타입 모델링 기법을 객체의 상속과 유사하게 볼 수 있다.

그렇기 때문에 상속관계 매핑이라고 함은 객체의 상속관계와 DB의 슈퍼타입-서브타입 관계를 매핑하는 것을 의미

기본적으로 엔티티 객체를 상속하여 선언하면 한 테이블에 모두 들어가는 전략이 적용된다.
## ✔︎ 매핑 전략
### 조인 전략
부모와 자식 객체를 각각의 테이블로 변환하는 전략으로 가장 정규화가 잘 되어있는 전략
- 저장공간을 효율적으로 사용 가능
<img width="805" alt="Image" src="https://github.com/user-attachments/assets/6d953ec4-9f5c-41a0-a6de-7473a13af0f6" />
- `@Inheritance(strategy=InheritanceType.JOINED)`으로 사용
- 외래 키 참조 무결성 제약조건이 사용 가능해짐
- 객체 값 저장 시에는 JPA가 두 테이블에 모두 INSERT 쿼리를 날려 주고, 조회 시에는 조인을 통해 값을 가져온다
	- 즉, 쿼리가 많이 필요하고, 복잡하다는 단점이 있음
- 슈퍼타입 ITEM에서 저장된 행이 어떤 서브타입인지 값을 담고 있는 부분이 DTYPE 컬럼
	- `@DiscriminatorColumn(name=“DTYPE”)`
	- `@DiscriminatorValue(“XXX”)` → 자식 클래스에서 컬럼에 저장될 값을 변경 가능 (디폴트는 entity명)

### 단일 테이블 전략
부모와 자식 객체를 하나의 테이블에 모두 집어넣는 전략. 단일 테이블 전략에서는 DTYPE이 필수로 필요
- 그렇기 때문에 `@DiscriminatorColumn`를 붙여주지 않아도 DTYPE이 추가되어 생성된다.
<img width="699" alt="Image" src="https://github.com/user-attachments/assets/a2e83bcb-f8ab-4edd-bbea-ba397a47e535" />
- `@Inheritance(strategy=InheritanceType.SINGLE_TABLE)`으로 사용
- 테이블이 1개이기 때문에 저장 시 INSERT문 한 번으로 가능, 조회 시 조인 필요 없음
	- 성능상 이점이 있으나, 테이블이 너무 커지는 경우 오히려 성능이 나빠질 수도 있다
- 매핑이 필요 없는 컬럼의 경우 null이 되기 때문에 주의 필요 

### 구현 클래스마다 테이블을 생성 전략
부모 객체의 정보를 포함하는 자식 객체 테이블을 각각 따로 생성하는 전략. (권장x)
<img width="805" alt="Image" src="https://github.com/user-attachments/assets/58021a3a-61b6-4cc3-acf8-b6ed02abea86" />
- `@Inheritance(strategy=InheritanceType.TABLE_PER_CLASS)`으로 사용
- 만들어진 서브타입 테이블들을 함께 조회할 일이 있다면 UNION을 사용하기 때문에 성능이 좋지 않다
	- 서브타입들 사이에 연관 관계가 없기 때문에 통합해서 관리하는 것이 어려워짐

## ✔︎ @MappedSuperclass
공통 매핑 정보가 필요할 때 사용(누가 수정했는지, 언제 수정했는지 등등..)
- DB에서는 테이블이 완전히 나눠져 있지만, 객체에서는 공통 매핑 정보를 상속받아서 쓰고 싶은 경우 사용
```java
@MappedSuperclass
public abstract class BaseEntity {  
	private String createBy;  
	private LocalDateTime createDate;  
}
```

```java
@Entity  
public class Item extends BaseEntity {  
	@Id  
	@GeneratedValue(strategy = GenerationType.IDENTITY)  
	private Long id;  
	private String name;  
	private Integer price;  
	private Integer quantity;  
}
```

- 상속관계 매핑이 아님 주의
- `@MappedSuperclass` 클래스는 엔티티 아님 주의
- 직접 생성해서 사용하지 않으므로, 추상 클래스 권장됨