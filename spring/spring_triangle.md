# Spring Triangle (스프링 핵심 기술 중 3가지)
스프링 삼각형이란 스프링의 근간을 이루는 핵심 개념중 다음 3가지를 아울러 이루는 말로, `관점 지향 프로그래밍`, `의존성 주입`, `추상화 서비스`를 의미한다.
## Inversion of Control / Dependency Injection (IoC / DI) : 제어 역전 / 의존성 주입
### 제어의 역전
#### 누가 메시지를 시작하는가?
- 일반적으로 프로그래밍에서는 프로그래머가 작성한 프로그램이 외부 라이브러리의 코드를 호출한다. 마치 우리가 스프링 라이브러리의 어떠한
코드를 호출해 개발을 하는 것을 생각하면 이해가 쉽다.
- 하지만 이와 반대로 외부 라이브러리인 스프링이 우리가 작성한 코드를 호출하는 것을 생각해 보자. 이것이 바로 제어 역전 디자인 패턴이다.
- 따라서 제어 역전은 스프링 프레임워크에만 국한된 개념이 아니며, 소프트웨어 전반에서 통용되는 개념이다.

#### 이러한 제어의 역전은 프레임워크와 라이브러리를 차별화하는 핵심 요소이기도 하다.
- 라이브러리는 일반적으로 호출하여 사용할 수 있는 클래스들의 집합 &rarr; 작업을 수행한 후 사용자에게 그 결과를 반환한다.
- 프레임워크는 그보다 더 많은 동작이 있는 추상적인 디자인을 구현한다 &rarr; 이를 사용하기 위해서는 특정 구현들이 필요하고
그것들은 사용하는 쪽에서 작성된다. 그리고 프레임워크는 설정에 따라 이런 구현체들을 호출하여 사용한다.

#### 앞서 제어 역전이 호출 제어권이 반전된다고 정리했는데, 그렇다면 스프링에서는 코드의 여러가지 부분 중에서 어떠한 것의 제어권을 역전해서 호출하는 것일까?
- 우리는 변경이 용이한 어플리케이션을 위해 인터페이스를 이용해 다형성을 가지는 코드를 작성한다. 그리고 이러한 방식은
  필연적으로 인터페이스의 구현체가 존재해야만 한다.
- 이것은 이를 사용하는 코드가 해당 인터페이스와 그 구현체에 종속적이라는 의미와도 같다.
- 이런 종속성 문제를 해결하기 위해 스프링은 이런 구현체 모듈을 주입하는 식으로 작동한다.

&rarr; 이를 생각해보면 우리가 작성한 어떠한 객체가 필요로 하는 의존성을 스프링 프레임워크가 외부에서 주입해주는 것.
즉 객체가 스스로 구현체를 관리하지 않고 스프링으로 제어가 넘어가는 것이다.

그리고 이러한 방법을 일반적인 의미로 사용되는 제어 역전이라는 용어 이름이 아닌 의존성 주입이라는 용어로 따로 정의하게 됨.

**즉 `의존성 주입`은 `제어 역전`이라는 디자인 패턴을 구현하기 위한 구체적인 방법인 것이다.**

### 의존성 주입
- 생성자
- setter
- 필드

스프링에서 의존성 주입은 위 방법들로 가능하다. 하지만 될 수 있으면 다음과 같은 이점 때문에 생성자 주입을 권장하고 있다.
 - 불변 객체로 선언 가능하다는 점
 - 테스트가 용이하다는 점
 - 객체 생성 시점에 주입되기 때문에 주입을 빼먹는다던지 하는 실수를 줄일 수 있다는 점 등등
### IoC Container
제어 역전을 구현하고 관리하기 위한 요소로, 스프링에서는 `ApplicationContext`가 IoC Container의 역할을 담당하고 있다.
- `ApplicationContext`는 POJO와 구성 메타데이터를 읽어내고, 이를 바탕으로 빈 객체를 관리하는 역할을 한다.
- 빈 객체와 관련해 기본적인 관리를 해주는 `BeanFactory` 인터페이스에서 조금 더 많은 기능이 추가된 것이 `ApplicationContext`이다.

### OCP와 DIP (개방 폐쇄 원칙과 의존관계 역전 원칙)
OCP는 확장에는 열려 있고, 변경에는 닫혀 있어야 한다는 개방-폐쇄 원칙, DIP는 구현체가 아닌 인터페이스에 의존해야 한다는 의존관계 역전 원칙인데, 스프링에서는 IoC와 DI를 이용해서 이 두가지 원칙을 준수할 수 있게 된다.

스프링을 사용하지 않을 때, 객체가 구현체를 직접 관리하는 예시를 보자. service 계층에서 구현체를 변경하는 상황이 생기면, 다음과 같이 코드의 변경이 일어날 수밖에 없다.
- MemberService가 인터페이스와 동시에 구체 클래스도 의존하고 있다 → DIP 위반
- 코드 내용의 확장(새로운 기능을 위한 구체 클래스 변경)을 위해 변경이 발생한다 → OCP 위반
```java
public class MemberService {
	// private MemberRepository memberRepository = new MemoryMemberRepository();
	private MemberRepository memberRepository = new JdbcMemberRepository();
}
```
따라서 구체 클래스에 관한 의존성을 직접 선언하는 것이 아니고 어떤 누군가가 구현 객체를 대신 생성하고 주입해주면 된다. &rarr; 이것이 스프링의 의존성 주입을 통한 제어 역전으로 가능해진다
```java
@Service
public class MemberService {
	// 인터페이스에만 의존 가능
	private MemberRepository memberRepository;
}
```
```java
@Configuration  
public class AppConfig {  
	// 스프링 컨테이너가 스프링 빈을 관리하면서 객체를 주입해주게 됨
	@Bean  
	public MemberService memberService() {  
	return new MemberServiceImpl(memberRepository());  
	}  
  
	@Bean  
	public MemoryMemberRepository memberRepository() {  
	return new MemoryMemberRepository();  
	}   
}
```
이렇게 객체 생성 및 연결의 역할과 실행하는 역할이 명확히 분리되었다. (관심사의 분리로 SRP 원칙까지 준수 가능해짐)
## Portable Service Abstraction (PSA) : 추상화 서비스
## Aspect-Oriented Programming (AOP) : 관점 지향 프로그래밍

### 참고 문헌
[제어 반전](https://martinfowler.com/bliki/InversionOfControl.html)  
[The IoC Container section](https://docs.spring.io/spring-framework/reference/core/beans.html)  
[DIP in the Wild](https://martinfowler.com/articles/dipInTheWild.html#YouMeanDependencyInversionRight)
김영한님 스프링 핵심 원리 기본편 강의