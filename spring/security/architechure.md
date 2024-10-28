# Architecture :: Spring Security [🔗](https://docs.spring.io/spring-security/reference/servlet/architecture.html)

## Review of Filters

Spring Security은 서블릿 필터를 기반으로 서블릿을 서포트하므로, 아키텍쳐를 살펴보기 전 필터에 대해 간단히 정리해보자.

- 클라이언트에게 요청이 오면 HttpServletRequest를 처리하기 위한 필터 체인이 만들어지는데, 내부에는 필터 객체들과 서블릿이 포함되어 있다.
- 서블릿은 하나의 요청과 응답을 처리할 수 있지만, 필터는 한 개 이상 적용할 수 있고, 필터 체인 내부에서 여러 개의 필터를 거쳐서 처리가 이루어진다.

## **DelegatingFilterProxy**

- 서블릿 컨테이너와 스프링 사이의 다리 역할을 하는 필터 구현체
- 서블릿 컨테이너와 스프링의 라이프사이클이 다를 수 밖에 없어 서블릿 컨테이너는 bean을 인식하지 못하는데, 해당 프록시 필터 객체를 통해 작업을 내부적으로 bean으로 등록된 필터에 위임할 수 있다.

## **FilterChainProxy**

- 스프링 시큐리티에서 제공하는 특수한 필터로, 시큐리티 필터 체인으로 들어가기 위한 첫번째 지점
- `filterChainProxy`는 bean이기 때문에 `DelegatingFilterProxy`로 래핑되어 있다.

## **SecurityFilterChain**

- 한 마디로 `securityFilterChain`은 시큐리티 필터들의 목록이다
- 어떤 요청이 들어오면 어떤 시큐리티 필터를 태워야 하는지를 처리해야 하기 때문에 `FilterChainProxy`가 `SecurityFilterChain`을 사용한다.


> #### 💡 `SecurityFilterChain`도 bean 객체인데 왜 바로 `DelegatingFilterProxy`로 추가를 하지 않았는지?  
> → 필터라는 것은 모든 필터가 다 적용되거나 하는 것이 아니므로, 디버깅과 같은 상황을 고려하여 모든 필터의 출발점이 있는 편이 낫다. 어떠한 문제가 생겼을 때 FilterChainProxy부터 디버깅을 시작한다든지 할 수 있다.  
> → 또 filterChainProxy에서 필수적인 작업들을 실행하도록 할 수도 있다.

- `filterChainProxy`는 `RequestMatcher`를 사용해서 유연하게 필터 체인을 호출할 수도 있다.

![image](https://github.com/user-attachments/assets/e2ebeccd-a67e-4b5a-88b1-1b9b25f86a68)

- 해당 그림에서 `filterChainProxy` 는 어떤 `securityFilterChain` 를 사용할지를 결정한다.
- 일치하는 첫 번째 필터체인만을 선택하는데, 예를 들어 `/api/message/` 라는 리퀘스트 요청이 들어오면, 0번 필터체인과, n번 필터체인 둘 다 패턴에는 일치하지만, 첫 번째 필터체인인 0번 체인으로 들어가게 된다.
- 쭉 필터 체인을 찾으면서 내려오다가, 일치하는 패턴이 없으면 n번 필터 체인이 실행되는 식이다.
- 필터 체인 내부적으로 존재하는 필터의 개수는 제각각으로 각 필터 체인별로 독립적이다. 0개가 될 수도 있다.

## Security Filters

- 필터들은 `filterChainProxy` 에 삽입되며, 다양한 용도로 사용된다.
- 올바르게 작동하기 위해 특정한 순서대로 실행되는 것을 보장하며, 예를 들어 인증 수행 필터가 인가 필터보다 먼저 호출되는 식이다.
- 일반적으로는 필터의 진행 순서를 알 필요는 없으나, `FilterOrderRegistration` 클래스에서 확인할 수 있다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(Customizer.withDefaults())
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .formLogin(Customizer.withDefaults());
        return http.build();
    }

}
```

위와 같은 코드 구성은 다음과 같은 필터를 생성한다.

| Filter | 설명 | 메서드 |
| --- | --- | --- |
| CsrfFilter | csrf 공격으로부터 보호하는 기능을 하는 필터 | csrf |
| UsernamePasswordAuthenticationFilter | 사용자 아이디와 비밀번호를 인증하는 기능의 필터 | formLogin |
| BasicAuthenticationFilter | UsernamePasswordAuthenticationToken 토큰 생성 | httpBasic |
| AuthorizationFilter | HttpServletRequests에 대한 권한 제공 | authorizeHttpRequests |

## 사용자 정의 필터 추가하기

```java
public class TenantFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;

        String tenantId = request.getHeader("X-Tenant-Id"); (1) 리퀘스트 헤더에서 테넌트ID를 가져온다
        boolean hasAccess = isUserAllowed(tenantId); (2) 사용자에게 테넌트 ID 엑세스 권한이 있는지 확인한다
        if (hasAccess) {
            filterChain.doFilter(request, response); (3) 권한이 있는 경우, 필터 체인에 있는 나머지 필터를 호출한다
            return;
        }
        throw new AccessDeniedException("Access denied"); (4) 권한이 없는 경우 예외를 발생시킨다
    }

}

@Bean
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        // ...
        .addFilterBefore(new TenantFilter(), AuthorizationFilter.class); 
    return http.build();
}
```

- 여기서 `Filter`를 구현하는 대신에 요청당 한 번만 호출되는 필터 클래스인 `OncePerRequestFilter`를 구현할 수도 있다.
- 필터를 만들고 나서는 필터 체인 내부에 등록해주어야 하는데 위치를 지정할 수 있다. 인증 필터 이후에 `TenantFilter` 필터가 호출되도록 예시에서는 등록하였다.
- 필터를 `@Component` 등으로 bean으로 관리하게 되는 경우에는 주의가 필요하다. 이렇게 되면 스프링 부트가 해당 필터를 서블릿 컨테이너에 등록하게 되면서 서블릿 필터 체인에서 한 번, 시큐리티에서 한 번 이렇게 두 번 필터가 호출 될 수 있기 때문이다.
    - 이를 방지하기 위해 아래와 같은 코드를 등록할 수 있다.

    ```java
    @Bean
    public FilterRegistrationBean<TenantFilter> tenantFilterRegistration(TenantFilter filter) {
        FilterRegistrationBean<TenantFilter> registration = new FilterRegistrationBean<>(filter);
        registration.setEnabled(false);
        return registration;
    }
    ```


## 예외 다루기

- `ExceptionTranslationFilter` 는 필터 체인 내에서 발생하는  [`AccessDeniedException`](https://docs.spring.io/spring-security/site/docs/6.3.3/api/org/springframework/security/access/AccessDeniedException.html) 와 [`AuthenticationException`](https://docs.spring.io/spring-security/site/docs/6.3.3/api//org/springframework/security/core/AuthenticationException.html)  예외를 HTTP 응답으로 바꿔주며, 필터 체인의 체인 중 하나로 들어가 있다.
- 즉 이 두 가지 예외가 아니면 `ExceptionTranslationFilter` 는 아무런 작업도 수행하지 않는다.

### AccessDeniedException

- 인증 된 사용자가 특정 리소스에 접근할 권한이 없는 경우 발생하는 예외
- 403 Forbidden

### AuthenticationException

- 인증되지 않은 사용자가 보호된 리소스에 접근하려 하는 경우 발생하는 예외
- 401 Unauthorized 혹은 인증 페이지(로그인 페이지)로 리다이렉트

![image2](https://github.com/user-attachments/assets/b184875c-7429-41f0-b19a-1fbac63994e8)

- 기본적으로 흐름에서 `ExceptionTranslationFilter` 필터를 만나면 나머지 필터를 계속 진행한다.
- 만약 이후에  `AuthenticationException` 가 발생하는 경우…
    - `SecurityContextHolder`를 비운다.
    - 인증 성공시에 `HttpServletRequest`를 재사용할 수 있도록 저장한다.
    - `AuthenticationEntryPoint` 를 통해 자격 증명을 요청한다. (예를 들면 로그인 페이지로 리다이렉션)
- `AccessDeniedException` 발생시에는 접근이 금지되고 해당 예외를 처리할 핸들러가 실행된다.

## 재인증 시 기존 Request 저장하기

`AuthenticationException` 예외가 발생하여 인증 과정 페이지로 리다이렉션이 일어나는 경우, 인증이 성공한 후 다시 이어서 요청을 진행할 수 있도록 요청 정보의 저장이 필요하다. 이를 위해 시큐리티는 `RequestCache`를 이용하여 `HttpServletRequest`를 저장한다.

- 인증이 성공한 후, [`RequestCacheAwareFilter`](https://docs.spring.io/spring-security/reference/servlet/architecture.html#requestcacheawarefilter)가 `RequestCache` 에서 저장된 `HttpServletRequest` 를 가져와서 이전 요청을 이어서 진행한다.
- 디폴트로 `HttpSessionRequestCache`이 사용되기 때문에, 다르게 설정해주지 않는다면 기본적으로 세션에 캐시 정보를 저장하게 된다.
- [`NullRequestCache`](https://docs.spring.io/spring-security/site/docs/6.3.4/api/org/springframework/security/web/savedrequest/NullRequestCache.html) 구현체를 사용하여 캐시 기능을 아예 끌 수도 있다.

## Logging

기본적으로 시큐리티와 관련된 로깅은 DEBUG과 TRACE 레벨로 제공된다. 시큐리티는 요청이 거부된 경우 보안 문제 때문에 응답 본문에 세부 사항을 추가하지 않기 때문에, 이유를 파악할 때 로깅 기능을 유용하게 사용할 수 있다.