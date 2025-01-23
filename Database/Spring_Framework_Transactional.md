# Spring Framework @Transactional
## @Transactional
스프링 프레임워크에서 트랜잭션을 구성하는 방법으로는 XML 기반 방식과 어노테이션 사용 방식이 있는데, 그 중 어노테이션 사용 방식에 대해 정리해보자
```
// the service class that we want to make transactional
@Transactional
public class DefaultFooService implements FooService {

	@Override
	public Foo getFoo(String fooName) {
		// ...
	}

	@Override
	public Foo getFoo(String fooName, String barName) {
		// ...
	}

	@Override
	public void insertFoo(Foo foo) {
		// ...
	}

	@Override
	public void updateFoo(Foo foo) {
		// ...
	}
}
```
위와 같이 어노테이션을 클래스 수준에 붙여서 사용하는 경우, 트랜잭션 로직이 클래스의 모든 하위 메서드와 서브 클래스들에 적용되며, 원한다면 클래스 수준이 아니라 각 메서드에 개별적으로 어노테이션을 추가할 수도 있다.
### 주의할 점
#### 클래스 수준 사용에서
- 자식 클래스에 `@Transactional` 설정을 사용하는 경우, 해당 설정은 조상 클래스들에는 영향을 끼치지 않는다.
	- 따라서 만일 부모 클래스의 메서드를 트랜잭션을 적용하여 사용하고자 한다면, 자식 클래스에서 오버라이드를 해야 함
#### 메서드 수준 사용에서 : visibility와 관련하여
- 프록시 기반으로 동작하는 Spring의 트랜잭션 관리 특성 상, `@Transactional`는 일반적으로 `public` 메서드에 적용되지만, Spring 6.0부터는 `protected`와 `package-visible`인 메서드도 트랜잭션을 적용할 수 있도록 변경되었다.
	- 다만 인터페이스 기반 프록시를 이용해 트랜잭션을 적용하기 위해서는 타겟 메서드가 `public`이어야 한다는 점에 유의하자
	- 즉, CGLIB 프록시로 생성되는 경우에만 `protected`와 `package-visible` 수준 메서드에 트랜잭션을 걸 수 있다는 것이다
#### 클래스 레벨과 메서드 레벨에 동시에 어노테이션을 붙이는 경우
`@Transactional`을 클래스에도 붙이고 메서드에도 동시에 붙일 수 있는데, 메서드 레벨에 붙어 있는 트랜잭션 설정이 클래스 레벨의 설정보다 우선순위를 가진다.

따라서 아래 예시에서 `getFoo()`는 readOnly = true가 그대로 적용되지만, `updateFoo()`에 적용되는 설정은 readOnly = false이다.
```
@Transactional(readOnly = true)
public class DefaultFooService implements FooService {

	public Foo getFoo(String fooName) {
		// ...
	}

	// these settings have precedence for this method
	@Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
	public void updateFoo(Foo foo) {
		// ...
	}
}
```
### @Transactional만으로 트랜잭션이 동작하는 것이 아니다!
트랜잭션 어노테이션은 자바 코드 이곳저곳에 부착이 가능하다. 하지만 단순히 해당 어노테이션만으로 트랜잭션이 활성화 되는 것은 아님

왜냐하면 해당 어노테이션의 역할은 단지 런타임 시점에 트랜잭션 프록시 bean을 구성하기 위한 메타데이터를 나타내는 것 뿐이기 때문이다.

따라서 spring이 트랜잭션 프록시를 만들고 관리하도록 transaction management를 활성화하는 설정들이 필요하다. (transaction-manager과 같은 것들)

→ spring boot로 만든 프로젝트에서 jpa와 같은 data 관련 의존성을 당겨 오는 경우 마치`@EnableTransactionManagement`를 사용한 것처럼 자동적으로 관련 옵션들을 활성화시켜주기 때문에, 사용자가 따로 설정을 신경쓰지 않고 `@Transactional`을 붙이기만 해도 괜찮은 것

### 인터페이스에 @Transactional을 사용하는 것은 권장되지 않는다
공식 문서에서는 인터페이스에 어노테이션 메서드를 추가하는 것보다는 구체 클래스의 메서드에 어노테이션을 추가할 것을 권장하고 있으며, 주요 이유는 자바에서 어노테이션은 인터페이스에서 상속되지 않는다는 사실 때문이다.
따라서 아래와 같은 현상들이 발생할 수 있다.
1. 어노테이션을 인터페이스에만 붙여둔다면, 실제로 구현 클래스의 메서드에 잘 적용되지 않을 위험성이 있다.
2. 특히 AspectJ 모드에서 인터페이스에 붙은 어노테이션이 잘 작동하지 않을 수 있는데 왜냐하면 **대상 메서드의 바이트코드를 조작할 때, 해당 메서드에 트랜잭션 어노테이션이 달려 있다는 사실을 알 수 없기 때문이다.**

> **AspectJ**
> 스프링 AOP를 위한 프레임워크로, 바이트코드를 직접 수정하는 방식으로 AOP 코드를 생성한다.
> 
> 일반적으로 프록시 객체를 통해 AOP를 구현하는 방법과는 다르게, 바이트코드가 실행되기 전에 코드를 위빙해버리는 방식이다. 여기에서 바이트코드를 변경하여 AOP 로직을 넣는 것

### @Transactional Settings
 `@Transactional`의 디폴트 설정은 다음과 같다.
 - 전파 수준 : `PROPAGATION_REQUIRED` (기존 트랜잭션이 있는 경우에는 참여, 없는 경우 생성)
 - 격리 수준 : `ISOLATION_DEFAULT` (DB의 기본 격리 수준으로 설정)
 - 읽기 쓰기 기능이 둘 다 가능 (not read only)
 - 타임아웃 시간은 기본 설정 시간을 따라간다, 만일 타임아웃 관련 설정이 없는 경우에는 사용하지 않음
 - 런타임 예외가 발생 시에만 롤백

spring 6.2부터는 기본 롤백 규칙을 변경하여 사용할 수 있도록 지원하기 때문에, 런타임 예외 뿐 아니라 모든 예외가 발생했을 때 롤백하도록 설정이 가능하다.