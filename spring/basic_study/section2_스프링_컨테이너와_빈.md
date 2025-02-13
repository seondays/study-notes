# section 2. 스프링 컨테이너 & 스프링 빈
## ✔︎ 스프링 컨테이너
 `ApplicationContext`가 바로 스프링 컨테이너 역할을 하는 타입으로, 앞서 언급한 DI를 진행해주는 주체이다.
```java
// 자바 설정 클래스 기반으로 스프링 컨테이너 생성
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(설정 정보 클래스.class);
```
`@Configuration`이 붙은 설정 정보 클래스들을 확인해서 `@Bean` 메서드들을 모두 호출하고, 그 결과들을 스프링 컨테이너에 스프링 빈으로 등록한다.

이렇게 등록한 객체들을 스프링 빈으로 관리하게 되고, 사용 시에는 컨테이너에서 스프링 빈들을 꺼내서 사용하게 된다. 이후 스프링 컨테이너는 설정 정보를 참고하여 생성된 빈들을 가지고 의존관계 주입 작업을 진행한다.

## ✔︎ BeanFactory & ApplicationContext
### **BeanFactory**
- 스프링 컨테이너의 최상위 인터페이스로, 스프링 빈을 조회 및 관리하는 역할을 한다

### **ApplicationContext**
![applicationContext](https://github.com/user-attachments/assets/f7c35c2d-7ce0-467d-b7a1-0a779b745d28)
- `BeanFactory`를 상속받는 인터페이스
- `BeanFactory`와 비교하여 애플리케이션을 개발하기 위한 부가 기능들이 많이 포함되었다
	- 국제화 기능 → 접속하는 국가에 맞게 출력을 변경할 수 있음
	- 환경변수 지원 → 개발, 운영 환경을 구분해서 처리할 수 있도록 해줌
	- 이벤트 작업 → 이벤트 리스너를 통한 이벤트 발행 & 구독 모델 처리를 지원함
	- 리소스 조회 편의성 → 여러가지 루트의 리소스를 편리하게 조회할 수 있음
- 이러한 편리한 부가 기능들을 지원하기 때문에 `BeanFactory`가 아닌 `ApplicationContext`가 주로 사용되며, 대부분 일반적으로 스프링 컨테이너라고 하면 `ApplicationContext`를 의미하게 된다.

## ✔︎ 스프링 빈
스프링 컨테이너에서 빈을 꺼내서 사용도 가능한데, 기본적으로는 `getBean(빈이름, 타입)` 형태로 조회하지만 이름 없이 타입만으로도 조회 가능하다. (같은 타입의 빈이 여러개라면 이름과 함께 찾아야만 한다)
```java
AnnotationConfigApplicationContext ac = new
 AnnotationConfigApplicationContext(AppConfig.class);

ac.getBean("beanName", beanName.class);
ac.getBean(beanName.class);
```
중요한 것은 부모-자식 상속 관계에 있는 빈들을 조회하는 경우, 부모 빈을 조회하면 자식 빈들이 모두 함께 조회된다는 것인데, 따라서 `Object` 타입으로 조회하면 모든 스프링 빈을 조회할 수 있다. (`getBeansOfType()`)

## ✔︎ 스프링은 다양한 설정 형식을 지원한다
이러한 스프링 설정 정보는 자바 코드로 작성할 수 있을 뿐 아니라, XML으로도 가능하고 원한다면 커스텀 방법으로도 가능하다. 이러한 다양한 설정 형식 지원이 가능한 이유는 **빈 설정 메타정보를 추상화**한 `BeanDefinition`이라는 인터페이스가 존재하기 때문!

스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성하는데, 빈을 생성하기 위해 이 설정 파일이 어떤 방법으로 구현되었는지 전혀 몰라도 된다.

오직 추상화 인터페이스인 `BeanDefinition`에만 의존하고 이것만을 확인하면 되기 때문이다.
![ConfigContext](https://github.com/user-attachments/assets/b53fc257-dd17-4fa0-99ca-4d9a947675f4)
- 보다시피 설정 정보를 작성하는 방식이 무엇이든 간에, 이것을 `BeanDefinition`으로만 만들어 낸다면 읽어내는 데 전혀 문제가 없는 것