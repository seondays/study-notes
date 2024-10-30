# Servlet Authentication Architecture :: Spring Security [🔗](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)

스프링 시큐리티에서 인증과 관련된 아키텍쳐 요소들을 정리합니다.

## **SecurityContextHolder**

인증된 사용자에 대한 세부 정보를 저장하는 곳이다. 시큐리티는 이곳에 값이 포함되어 있다면, 인증된 유저로 인식하고, 유저에 관한 정보가 필요할 경우 해당 값을 사용한다.

![securitycontextholder](https://github.com/user-attachments/assets/c9b22755-95e2-4b72-8805-5ac9919aa578)

### 동작 방식

- 기본적으로 `ThreadLocal`을 이용하여 유저 정보를 저장한다. 따라서 동일한 스레드 내부의 메서드들은 항상 `SecurityContextHolder`에 접근하여 유저 정보를 사용할 수 있다.
- 모든 요청이 완료되면 `FilterChainProxy`가 `SecurityContext`를 초기화한다.
- 만일 멀티스레딩 애플리케이션에 대한 처리가 필요하다면 설정을 변경하여 MODE_GLOBAL이나 MODE_INHERITABLETHREADLOCAL 로 설정해서 사용할 수도 있다.

## SecurityContext

`SecurityContextHolder` 내부에 저장되어 있는 객체로, 인증 객체(`Authentication`)를 포함하고 있다.

## Authentication

`Authentication` 는 시큐리티 내에서 크게 두 가지 용도로 사용된다.

- 사용자가 입력한 credentials을 `AuthenticationManager` 에게 전달하기 위한 객체의 역할
- 현재 인증된 사용자의 정보를 담기 위한 역할. 이는 `SecurityContext` 내부에 저장되어 사용된다.

`Authentication` 객체의 구성 요소는 다음과 같다.

- `Principal` : 사용자를 식별하는 정보. 일반적으로 아이디-비밀번호 기반 인증에서는 주로 `UserDetails` 인터페이스를 구현한 객체가 사용된다.
- `Credentials` : 일반적으로는 비밀번호이다. 유출 방지를 위해 사용자 인증 후 지워진다.
- `Authorities` : 사용자의 권한 정보. `GrantedAuthority` 인터페이스를 사용하여 나타내고, 역할과 범위 정보를 나타낸다.

## GrantedAuthority

`Authentication.getAuthorities()`를 통해 `GrantedAuthority` 객체의 **컬렉션**을 가져올 수 있다. 단일 객체가 아닌 컬렉션이라는 사실에 유의하자.

- 일반적인 아이디-비밀번호 기반 인증에서는 `UserDetailsService`에 의해 로드된다.
- 해당 권한정보는 애플리케이션 전체에 적용되는 권한이다.

## AuthenticationManager

아이디-비밀번호가 담겨 있는 인증 전 상태인 `Authentication` 객체를 전달받아 인증을 수행하는 역할을 한다. 인증이 성공적으로 완료된다면 인증된 `Authentication` 객체를 반환하는데, 이 때 반환되는 객체가 `SecurityContextHolder` 에 세팅된다.

- 꼭 `AuthenticationManager`를 사용하지 않더라도, `SecurityContextHolder` 에 직접 `Authentication` 를 세팅할 수도 있다.
- 가장 일반적인 구현체는 `ProviderManager`이다.

## ProviderManager

![providermanager](https://github.com/user-attachments/assets/af11d6a4-2f4e-490e-a1a6-291268b99978)

`AuthenticationManager` 의 구현체로, 가장 일반적으로 사용되는 구현체이다. 내부적으로 여러 개의 `AuthenticationProvider`를 가지고 있으며, 이 리스트를 쭉 거치면서 인증 프로세스가 진행된다.

각각의 `AuthenticationProvider`마다 처리할 수 있는 인증 방식이 각각 다르기 때문에, 어떤 인증 요청이냐에 따라서 처리하는 `AuthenticationProvider`가 달라진다.

인증 처리가 완료되었다면 그 다음 `AuthenticationProvider` 는 실행되지 않으며, 만일 처리할 수 없는 인증 방식인 경우 `AuthenticationException` 예외가 발생한다.

`ProviderManager`가 해당 인증을 처리할 수 없는 경우 예외 대신 부모 `AuthenticationManager`구현체에게 인증 처리를 위임하도록 설정할 수도 있으며, 여러 `ProviderManager` 인스턴스가 동일한 부모 `AuthenticationManager` 를 공유할 수도 있다.

![providermanagers-parent](https://github.com/user-attachments/assets/25224672-3b26-4067-b74b-2a5b86c04d39)

## AuthenticationProvider

`ProviderManager` 내부에서 특정 유형의 인증을 수행하게 될 객체. 예를 들면 `DaoAuthenticationProvider`는 아이디-비밀번호 기반 인증, `JwtAuthenticationProvider`는 JWT 토큰을 이용한 인증을 담당한다.

## **Request Credentials with AuthenticationEntryPoint**

`AuthenticationEntryPoint`는 일반적으로 인증이 필요한 리소스에 사용자가 접근하려고 할 때 예외가 발생하면, 인증이 필요하다는 것을 알리는 http 응답을 클라이언트에게 반환한다. 만일 클라이언트의 요청에 이미 Credentials(자격 증명)이 포함되어 있다면 그것으로 처리를 진행하기 때문에 `AuthenticationEntryPoint` 가 작동할 필요는 없다.

- 로그인 페이지로 리다이렉션
- WWW-Authenticate 헤더를 사용한 응답
- 그 외 다른 작업을 커스텀 작성해서 사용

## **AbstractAuthenticationProcessingFilter**

해당 클래스는 사용자의 Credentials를 인증하기 위한 필터로 사용된다.

![abstractauthenticationprocessingfilter](https://github.com/user-attachments/assets/bf9606d1-c1c4-4302-964e-9ef625a5848b)

1. 사용자가 Credentials를 제출하면 필터는 이를 바탕으로 `Authentication`(인증 객체)를 생성한다. 해당 인증 객체의 유형은 어떤 인증 방식을 사용하는지에 따라 상이하다.
2. 다음으로 `Authentication`는 `AuthenticationManager`에게 전달되어 인증 프로세스가 진행된다.
3. 인증에 실패하면 `SecurityContextHolder`를 비우고 만일 RememberMe 기능이 설정되어 있다면 `RememberMeServices.loginFail`로직이 실행되지만, 설정되어 있지 않다면 넘어가서 `AuthenticationFailureHandler` 가 동작한다.
    - 일반적으로 `AuthenticationFailureHandler` 는 사용자를 인증 페이지로 다시 리다이렉션한다.
4. 인증에 성공하면 다음 과정이 실행된다.
    - 인증 시 세션을 어떻게 관리할 것인지를 관리하는`SessionAuthenticationStrategy` 가 설정된 전략에 따라 로그인 세션을 처리한다.
    - 인증 성공 후 `Authentication` 객체는 `SecurityContextHolder`에 저장된다.
    - 만일 RememberMe 기능이 설정되어 있는 경우, `RememberMeServices.loginSuccess` 이 실행되어 내부 로직이 실행된다.
    - `ApplicationEventPublisher`가 `InteractiveAuthenticationSuccessEvent`를 발행하여 인증이 성공되었음을 알린다.
    - 마지막으로 `AuthenticationSuccessHandler`가 실행된다.