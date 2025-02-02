# 자바 로깅 라이브러리들에 대한 간략 정리
[How To Do Logging In Java](https://www.marcobehler.com/guides/java-logging) 글을 기반으로 학습 및 정리한 문서입니다.

## Legacy Logging Libraries
### java.util.logging (JUL)
자바의 1.4 버전(2002년)부터 JDK에서 제공하는 자체 로깅 라이브러리이다. 하지만 일관성 없는 API, 번거로운 설정 방식, 좋지 않은 성능, 기능 부족 등의 이유로 잘 사용되지 않고 있다.
```java
java.util.logging.Logger logger =  java.util.logging.Logger.getLogger(this.getClass().getName());

// 일반적으로 익숙한 로그 레벨과는 명칭이 다르다는 점 주의
logger.severe("This is an error message"); // == ERROR
logger.fine("Here is a debug message"); // == DEBUG
```
### Log4j(v1)
2000년대 초반부터 2010년대까지 널리 사용되던 라이브러리.

직관적인 로그 레벨로 명칭이 변경되었고, 다양한 `Appender`를 지원하여 로그를 콘솔 뿐 아니라 여러가지 방법으로 내보낼 수 있었으며, 로그 메시지의 출력을 위한 패턴을 지원해서 포맷팅이 가능했다.

이후 `Log4j2`로 대체되면서 점차 사장됨
```java
// Log4j V1
org.apache.log4j.Logger logger = org.apache.log4j.Logger.getLogger(MyClass.getClass().getName());

logger.info("This is an info message");
logger.error("This is an error message");
logger.debug("Here is a debug message");
```
### Apache Commons Logging (JCL)
역시 2000년대 초반 비슷한 시기에 출시된 라이브러리. 로깅 구현체가 아닌 `로깅 인터페이스`를 제공하는 라이브러리였다는 특징이 있다.

```java
// Apache Commons Logging
org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(MyApp.class);

log.info("This is an info message");
log.error("This is an error message");
log.debug("Here is a debug message");
```
그렇기 때문에 실제 로깅 작업 수행은 다른 로깅 구현체가 담당하며, `Log4j`, `JUL` 등이 필요하다.

#### 이런 방식이 도입된 이유
이렇게 인터페이스를 제공하는 방식이 도입된 이유는 라이브러리를 개발하는 경우 특정 로깅 라이브러리에 종속되는 것을 피하기 위해서였다.
- 만일 라이브러리를 개발해야 하는데 로깅을 `Log4j`을 이용해서 개발했다면? 나의 라이브러리를 사용하는 사용자 역시 `Log4j`를 사용해야 하는 문제점 (종속)
- 하지만 `JCL`을 사용하면, 특정한 로깅 라이브러리에 의존하지 않고 인터페이스만을 사용함으로서 사용자가 원하는 로깅 시스템을 사용하도록 할 수 있다.
#### 문제점
하지만 사용 시 몇몇 문제점들이 존재했기 때문에, 현재는 해당 문제점들을 개선한 `SLF4J`에 의해 대체되었다.
- 런타임에 클래스 로더를 이용하여 동적으로 구현체를 찾는 방식으로 동작했는데, 이것이 문제가 되는 일이 많았음
- API가 유연하지 않았음
- 사용을 위한 설정의 번거로움
## Modern Logging Libraries
### SLF4J & Logback
#### SLF4J (Simple Logging Facade for Java)
`Log4j` 개발자 Ceki Gülcü가 개발하였으며, 다양한 로깅 라이브러리에 대해 파사드, 추상화 역할을 하는 라이브러리.  `Log4j`, `Logback`, `JUL` 등이 구현체로 사용될 수 있다.

`SLF4J`는 런타임 바인딩을 하는 `JCL`과 다르게 컴파일 타임에 특정한 로깅 라이브러리를 사용하도록 바인딩된다는 특징이 있다. 만일 두 개 이상의 바인딩 후보가 있을 경우 무작위로 하나만을 선택한다.

`LoggerFactory.getLogger()`를 통해서 `Logger`를 가져오는 부분이 바로 이 구현체를 가져오기 위한 코드이다.

`SLF4J`의 기본 로깅 구현체가  `Logback`이며, 만일 `Log4j(v1)` 또는 `JUL`를 사용하는 경우에도 브릿지 라이브러리를 통해 코드 변경 없이 `SLF4J` + `Logback` 조합으로 사용할 수 있다.
```java
// SLF4J
org.slf4j.Logger logger = org.slf4j.LoggerFactory.getLogger(MyClass.class);

logger.info("This is an info message");
logger.error("This is an error message");
logger.debug("Here is a debug message"); //  you do not need 'logger.isDebugEnabled' checks anymore. SLF4J will handle that for you).
```

#### Logback
`Log4j`의 후속 프로젝트이며, 둘 다 Ceki Gülcü에 의해 개발되었기에 개념적으로 `Log4j`와 비슷하지만, 많은 사항이 개선되어 출시되었다.

[Reasons to prefer logback over log4j 1.x](https://logback.qos.ch/reasonsToSwitch.html#fasterImpl)
- 빨라진 성능, 더 적은 메모리를 사용하도록 개선
- 상세한 docs 제공
- 설정 파일 변경 시, 재시작 없이도 자동 반영
- 광범위한 로그 필터링 기능

등등의 개선점들이 있음
### Log4j2
`Log4j (v1)`의 후속 버전으로 출시되었으나, 완전히 다르게 다시 만들어졌다고 생각하자. (v1과 호환되지 않음)
- `log4j-api`와 `log4j-core`를 분리하여 API와 구현체를 따로 제공하는 형식이기 때문에 오히려 `SLF4J`와 공통점이 더 많다고 할 수 있다
- 시간 상 `SLF4J`이 나온 이후 개발되었다

그렇다면 기존에 사용하던 `SLF4J`를 대신할 만한 `Log4j2`만의 장점은 무엇이 있을까?
- 더 나은 성능
- 더 나은 API(문자열 뿐 아니라 객체도 기록 가능, 람다 지원)
```java
// Log4j (version 2)
org.apache.logging.log4j.Logger logger = org.apache.logging.log4j.LogManager.getLogger(MyApp.class);

logger.info("This is an info message");
logger.error("This is an error message");
logger.debug("Here is a debug message");
```
