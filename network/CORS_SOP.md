# CORS와 SOP

## SOP (Same-Origin policy) [🔗](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
동일 출처 정책으로, 한 출처(origin)에서 로드되는 문서나 스크립트가 다른 출처의 리소스와 상호작용 하는 것을 제한하는 매커니즘을 말한다.
&rarr; 쉽게 말하면 같은 출처의 리소스만 가져올 수 있도록 하는 보안정책

### 그렇다면 동일 출처(Same-Origin)라는 것은 무엇일까? [🔗](https://docs.tosspayments.com/resources/glossary/cors)
![image](https://github.com/user-attachments/assets/21662f1e-6ec1-4df0-b60a-21605c8aa697)
프로토콜, 포트, 호스트가 동일할 경우 출처가 동일한 것으로 본다. 세 가지 중 하나라도 다르면 동일하지 않은 것이다.

`http://store.company.com/dir/page.html` 라는 URL을 가지고 동일 출처의 예시를 살펴보자

| URL | 동일 출처인가? | 이유            |
|-----|----------|---------------|
|  `http://store.company.com/dir/page.html`   | 동일 출처    | path 제외 모두 동일 |
|   `http://store.company.com/dir/inner/another.html`  | 동일 출처    | path 제외 모두 동일 |
|   `https://store.company.com/page.html`  | 동일 출처 아님 | 프로토콜 다름       |
|  `http://store.company.com:81/dir/page.html`   | 동일 출처 아님 | 포트가 다름        |
|  `http://news.company.com/dir/page.html`   | 동일 출처 아님 | 호스트가 다름       |

## CORS (Cross-Origin Resource Sharing) [🔗]((https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS))
교차 출처 리소스 공유라는 의미로, CORS를 설정한다는 것은 주고 받으려는 리소스의 출처가 다른 경우에도 해당 리소스의 공유를 허용한다는 것

### 웹의 발달과 CORS
예전에는 백엔드와 프론트엔드의 구분 없이 하나의 도메인 안에서 처리가 가능했으나, 요즘에는 클라이언트에서 백엔드의 API를 호출하는 방식을
사용하게 되었다. 이런 경우 프론트-백이 다른 도메인을 사용하는 경우가 많기 때문에, 교차 출처의 리소스 공유도 허용하는 CORS 라는 정책이 생겨났다.

### CORS 허용 설정하기
#### 단순 요청 (Simple requests)
프리플라이트 요청을 하지 않는 단순 요청의 경우가 있는데 다음과 같은 조건을 모두 충족해야만 한다.
- GET, HEAD, POST 세 개의 매서드중 하나여야 함
- 자동으로 추가되는 헤더 외에, 다음 헤더들만 추가가 가능
  - Accept
  - Accept-Language
  - Content-Language
  - Content-Type
  - Range (단순한 범위 값을 가지는 경우만)
> #### Content-Type 헤더가 있는 경우
> `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 세 가지 타입만 허용된다.
- ReadableStream 객체 사용 금지

#### 프리플라이트 요청 (Preflighted requests)
단순 요청과 다르게 브라우저가 본 요청 이전에 OPTIONS 메서드를 사용하여 본 요청을 전송해도 안전한지 확인하는 요청 절차를 가진다.

<img width="780" alt=" 7 49 37" src="https://github.com/user-attachments/assets/b2edb720-2152-420e-89f4-8fb3b680f19b">

프리플라이트 요청에는 다음과 같은 두 헤더가 포함되어야 한다.
- Access-Control-Request-Method: POST (실제 요청에서 어떤 메서드를 사용할 것인지 명시)
- Access-Control-Request-Headers: content-type,x-pingother (실제 요청에서 어떤 커스텀 헤더를 포함해서 보낼 것인지 명시)

이후 응답에 두 요청 헤더에 포함된 정보를 허용하는 부분이 있는 것을 볼 수 있다.
- Access-Control-Allow-Origin: https://foo.example (리소스 접근을 허용할 출처 명시)
- Access-Control-Allow-Methods: POST, GET, OPTIONS (허용할 메서드 명시)
- Access-Control-Allow-Headers: X-PINGOTHER, Content-Type (허용할 헤더 명시)
- Access-Control-Max-Age: 86400 (프리플라이트 요청을 보내지 않아도 되도록 하는 캐시의 만료 시간 설정)

#### 자격 증명이 있는 요청 (Requests with credentials)
클라이언트에서 서버로 요청을 보낼 때 자격 증명(credential)을 포함해서 요청을 보내는 방식이다. HTTP 쿠키나 헤더에 있는 토큰 값 등등이 자격 증명으로 사용된다.

해당 요청에 대해 주의해야 할 점들이 있다.
- 서버는 `Access-Control-Allow-Credentials` 헤더 값을 true로 설정해야만 클라이언트가 정상적으로 응답을 받을 수 있다.
- 자격 증명이 있는 요청에 대해 서버는 `Access-Control-Allow-Origin`, `Control-Allow-Headers`, `Control-Allow-Methods`, `Control-Expose-Headers` 헤더의 값을 와일드카드로 지정해서는 안 된다.
- 서드파티 쿠키 정책의 영향을 받기 때문에, 브라우저가 서드파티 쿠키를 거부하는 상태일 경우 자격 증명 요청을 할 수 없게 될 수 있다. SameSite 속성 정책이 적용된다.