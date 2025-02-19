# section 4. 컴포넌트 스캔
 @Bean이나 XML을 통해 설정 정보를 직접 개발자가 작성하는 방법은 누락의 위험성, 관리가 어려워짐 등의 문제 발생 가능성이 높다. 특히 사용하는 스프링 빈이 많을수록 더더욱 그렇다.

이런 문제를 해결해주는 방법이 바로 스프링의 **컴포넌트 스캔**이다.

## ✔︎ 컴포넌트 스캔
컴포넌트 스캔이란 말 그대로 컴포넌트들을 스캔하여 스프링 빈으로 등록해주는 작업을 말한다.
```java
@Configuration  
@ComponentScan
public class AutoAppConfig {
}
```
컴포넌트 스캔을 사용할 새로운 설정 파일을 생성했다. `@ComponentScan`이 추가되어 있으며, 기존과는 다르게 이제 자동으로 빈을 만들고 의존성을 주입해주기 때문에 클래스 내부 내용이 비어 있음을 확인할 수 있다.

그리고 빈으로 등록하길 원하는 클래스들을 스캔 대상으로 만들어주기 위해 클래스에 `@Component`를 붙이는 작업이 필요하다.
```java
// Component 어노테이션 추가
@Component  
public class MemberServiceImpl implements MemberService {  
  
	private final MemberRepository memberRepository;  
  
	@Autowired  
	public MemberServiceImpl(MemberRepository memberRepository) {  
		this.memberRepository = memberRepository;  
	}  
  
}
```

한 가지 더 주의해야 할 점은 의존관계 주입 문제인데, 기존에는 의존관계 역시 직접 메서드를 통해 명시해주는 방식이었다. 

하지만 이제는 이런 설정 정보가 없기 때문에 의존관계 주입 역시 클래스 내부에서 처리하도록 해야 한다. 여기서 사용되는 것이 `@Autowired`이다.

## ✔︎ @ComponentScan과 @Autowired
두 어노테이션의 특징을 정리해보자
### @ComponentScan
- `@ComponentScan`은 `@Component`가 붙은 모든 클래스를 스프링 컨테이너에 스프링 빈으로 등록한다.
- 이 때 등록되는 스프링 빈의 이름은 클래스명에서 맨 앞글자만 소문자로 바꾸어 사용한다.
	- 원한다면 스프링 빈의 이름을 직접 지정할 수 있다 → `@Component(“myMemberService“)`
### @Autowired
- 생성자에 `@Autowired`가 붙어있다면, 스프링 컨테이너는 자동으로 파라미터에 해당되는 스프링 빈을 찾아서 주입해준다.
- 해당 여부를 판단하는 기본 조회 전략은 타입이 같은 빈을 찾는 것이다.
- 타입이 같은 빈이 여러 가지일 경우에는 충돌이 나기 때문에 NoUniqueBeanDefinitionException 예외가 발생한다.
	- 이런 경우에는 `@primary`를 사용하여 여러 빈들 중에 우선 순위를 주입할 수 있다
	- 또 빈 이름을 명시하여 주입할 수 있다

## ✔︎ 스캔의 탐색 위치와 대상
프로젝트의 모든 클래스를 대상으로 컴포넌트 스캔을 하려면 시간이 오래 걸리기 때문에, 필요한 부분만 탐색할 수 있도록 스캔 시작 위치를 설정할 수 있다.
```java
@ComponentScan(
         basePackages = "hello.core"
         )
```
- basePackages를 통해 탐색 패키지를 지정할 수 있다. 해당 패키지부터 모든 하위 패키지를 탐색하게 된다.
- 시작 위치를 여러 개 지정하는 것도 가능하다.
- 만일 따로 시작 패키지 위치를 저장하지 않는다면, `@ComponentScan`이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.

일반적으로 권장되는 방법은 따로 패키지 위치를 설정하지 않고, 설정 정보 클래스를 프로젝트 최상단에 두고 사용하는 것이다. 이렇게 되면 하위 패키지들도 모두 자동으로 컴포넌트 스캔의 대상이 되기 때문이다.

스프링 부트를 사용하는 경우에는 `@SpringBootApplication`이 붙은 시작 클래스가 프로젝트 시작 루트에 두는 것이 관례인데, 해당 어노테이션 안에` @ComponentScan`이 들어 있다. (즉, 프로젝트 패키지 전체를 스캔하는 동일한 효과인 것)

## ✔︎ 기본적인 컴포넌트 스캔의 대상들
스프링 컨트롤러, 서비스, 레포지토리에 붙여서 사용하는 어노테이션들 역시 내부에 @Component를 포함하고 있으며, 따라서 컴포넌트 스캔의 대상이 된다.

다만 해당 어노테이션들은 스프링이 특별한 빈으로 인식하고 부가 기능을 수행하도록 돕는다.
- `@Controller` → 해당 클래스가 스프링 MVC 컨트롤러임을 인식
- `@Repository` → 해당 클래스가 스프링 데이터 접근 계층임을 인식, 데이터 계층의 예외를 스프링의 예외로 변환해준다. (이런 예외의 추상화 덕분에 구체적 DB가 달라져도 동일한 예외로 처리가 가능해짐)
- `@Configuration` → 해당 클래스가 스프링 설정 정보를 담고 있음을 인식하고, 스프링 빈을 만들 때 싱글톤 객체로 만들어지도록 프록시를 적용한다.
- `@Service` → 서비스는 특별한 처리는 없으나, 이를 통해 비즈니스 로직이 존재하는 클래스라는 것을 쉽게 알아볼 수 있도록 해준다.

## ✔︎ 컴포넌트 스캔 필터 지정
컴포넌트 스캔 대상을 추가로 지정하거나, 제외할 수 있다.
- `includeFilters` : 대상으로 추가
- `excludeFilters` : 대상에서 제외
```java
@Configuration  
@ComponentScan(  
includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),  
excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)  
)  
static class ComponentFilterAppConfig {  
  
}
```
위 코드에서는 다음과 같은 작업이 진행된다.
- 기본적으로 현재 위치에서부터 `@Component`가 붙은 클래스들을 찾아서 스캔 진행
- 추가적으로 `MyIncludeComponent` 클래스를 스프링 빈으로 등록 (`@Component`가 없더라도 추가로 스캔해서 등록하는 것)
- `@MyExcludeComponent`가 붙은 클래스는 `@Component`가 있더라도 빈 등록에서 제외됨

### FilterType 옵션
추가적으로 `FilterType` 옵션에 대해 간단히 알아보자
```java
// ANNOTATION
includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class)

// ASSIGNABLE_TYPE
includeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = Parent.class)

// ASPECTJ
includeFilters = @Filter(type = FilterType.ASPECTJ, pattern = "com.example.service..*")

// REGEX
includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*ServiceImpl")
```
- ANNOTATION → 디폴트값, 어노테이션에 대해 동작
- ASSIGNABLE_TYPE → 지정 타입과 그 자식 타입에 대해 동작
- ASPECTJ → AspectJ 패턴으로 인식하여 동작
- REGEX → 정규 표현식으로 인식하여 동작
- CUSTOM → TypeFilter 인터페이스를 직접 구현하여 해당 인터페이스에 대해 동작

## ✔︎ 중복 등록과 충돌
빈 충돌 상황을 크게 두 가지로 나눠볼 수 있다.
1. 자동 빈 등록 vs 자동 빈 등록
	- 컴포넌트 스캔 시 이름이 같은 빈이 존재한다면 ConflictingBeanDefinitionException 예외를 발생시킨다.
2. 자동 빈 등록 vs 수동 빈 등록
	- 이 경우에는 수동 빈 등록이 우선권을 가지기 때문에, 수동 빈이 자동 빈을 오버라이딩 하는 상황이 발생한다.
	- 하지만 사실 이러한 동작은 의도적이라기보다는 설정들이 꼬여서 오버라이딩되는 문제가 발생하는 경우가 더 많다.
	- 따라서 최근 스프링 부트에서는 수동 빈 등록과 자동 빈 등록이 충돌이 나면 오류가 발생하는 것이 기본값이다.