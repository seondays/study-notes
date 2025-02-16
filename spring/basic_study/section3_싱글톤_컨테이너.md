# section 3. 싱글톤 컨테이너
스프링으로는 주로 웹 어플리케이션을 개발하게 되는데, 웹은 보통 여러 고객이 동시에 요청을 한다.

그런데 만일 모든 요청마다 이를 담당할 service나 controller 객체를 매번 만들어야 한다면? 이는 엄청난 메모리 낭비가 될 것이다.

이를 해결하기 위한 방법 중 하나가 바로 **싱글톤 패턴**을 사용하여 하나의 객체를 만들어 두고 이 객체를 공유해서 사용하는 것이다.

## ✔︎ 싱글톤 패턴
클래스의 인스턴스가 단 1개만 생성되는 것을 보장하는 디자인 패턴
- 따라서 객체 인스턴스가 2개 이상 만들어지지 못하도록 막는 장치가 필요함
	- 다양한 장치가 있지만, 그 중 `private` 생성자를 통해 외부에서 생성자를 호출하지 못하도록 하자
```java
public class Singleton {  

	// static 영역에 존재하는 오직 하나의 인스턴스 객체  
	private static final Singleton instance = new Singleton();  

	// 생성자를 private 수준으로 설정하여 외부에서 생성하지 못하도록 막음
	private Singleton() {}  

	// 사용을 위해서는 미리 만들어진 인스턴스를 이용하도록 한다
	public static Singleton getInstance() {  
		return instance;  
	}  
}
```

객체를 공유해서 사용할 수 있게 하는 장점도 있지만, 싱글톤 패턴에도 문제점이 존재한다.
- 싱글톤 패턴 구현을 위해 부가 코드가 필요
- 인스턴스를 가져와서 사용하기 때문에, 구체 클래스에 의존하는 모습
- 내부의 속성 변경과 초기화가 어려우며, 테스트 역시 어려움

→ 종합해보자면 유연성이 떨어진다는 단점이 존재한다

하지만 스프링 컨테이너는 이러한 문제점들을 해결하면서 스프링 빈을 싱글톤으로 관리하고 있다.

## ✔︎ 싱글톤 컨테이너
스프링 컨테이너는 객체를 딱 하나 생성해서 관리하기 때문에, 싱글톤 패턴 없이도 인스턴스를 싱글톤으로 관리할 수 있게 된다.

즉 스프링 컨테이너는 싱글톤 컨테이너의 역할을 하며, 싱글톤 패턴의 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다. (싱글톤 패턴 자체를 사용하지 않으니까)

스프링 컨테이너에서 기본적으로 스프링 빈을 등록하는 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것은 아니며, 요청 시 마다 새로운 객체를 만들게 할 수도 있다.

## ✔︎ 싱글톤 방식 사용 시 주의할 점 (패턴이든, 싱글톤 컨테이너든 모두 해당)
싱글톤 방식은 하나의 객체를 가지고 여러 클라이언트에서 사용하기 때문에 stateless하게 설계해서 사용해야 한다.
- 상태가 유지되어서는 안 된다는 의미
- 특정 클라이언트에 의존적인 필드 금지
- 특정 클라이언트가 값을 바꿀 수 있는 필드 금지
- 공유 값을 저장할 필드가 필요하다면 객체의 필드 대신 지역변수, 파라미터 등을 사용하자

**→ 따라서 스프링 빈 역시 stateless 해야 한다!**

## ✔︎ @Configuration과 싱글톤
Configuration 클래스를 보면, 빈 생성 과정에서 `memberRepository`가 여러 번 호출된다는 사실을 알 수 있다.
```java
@Configuration  
public class AppConfig {  
  
@Bean  
public MemberService memberService() {  
return new MemberServiceImpl(memberRepository());  
}  
  
@Bean  
public MemoryMemberRepository memberRepository() {  
return new MemoryMemberRepository();  
}  
  
@Bean  
public OrderService orderService() {  
return new OrderServiceImpl(memberRepository(), discountPolicy());  
}  
  
@Bean  
public static DiscountPolicy discountPolicy() {  
return new FixDiscountPolicy();  
}  
}
```
스프링 컨테이너는 빈을 싱글톤으로 관리한다고 했는데, 이렇게 여러 번 호출되어 생성되는 객체는 어떻게 싱글톤으로 관리할 수 있는 것일까?
![appconfig](https://github.com/user-attachments/assets/f9f060ad-9757-4652-8e09-a516f8f6aeea)
스프링 컨테이너는 설정 클래스인 `AppConfig`에 `@Configuration`이 적용되어 있는 경우, 바이트코드를 조작하여 특별한 로직을 추가한 프록시 객체(`AppConfig@CGLIB`)를 만들어 낸다. 그리고 이것을 원본 `AppConfig` 대신 설정 클래스로 사용하게 된다.

해당 프록시 코드는 `@Bean`을 호출할 때 이미 해당 스프링 빈이 존재한다면 존재하는 빈을 리턴하고, 존재하지 않는다면 새로 생성해서 등록하도록 하는 로직이 포함되어 있다. 그래서 이 추가적인 로직을 통해 스프링 빈을 싱글톤으로 관리할 수 있게 되는 것이다.

주의할 점은 만일 설정 클래스에 `@Configuration`이 없고 각 메서드에 `@Bean`만 적용되어 있다면, CGLIB이 작동하지 않게 되어 싱글톤임이 보장되지 않는다는 점이다.