## Zuul로 라우팅 해보기
### Micro Service
- Service A : localhost:8081/
- Service B : localhost:8082/

### Step 1. Zuul로 Gateway 만들기
1. Zuul 의존성 추가
```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-zuul'
}
```
2. EnableZuulProxy 어노테이션 추가
```java
@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {
```

3. Zuul routing 설정 (application.yml)
```yml
zuul:
  prefix: /api
  routes:
    servicea:
      path: /a/**
      url: http://localhost:8081
    serviceb:
      path: /b/**
      url: http://localhost:8082
```
> localhost:8080/api/a 또는 localhost:8080/api/b 로 사용 

### Step 2. Filter 적용해보기
- Zuul에서 클라이언트가 보낸 요청을 라우팅 하기 전(pre), 라우팅할 때(routing), 라우팅한 후 응답을 돌려 받았을 때(post) 필요한 작업을 수행 + (error)

- Type
	: pre, route, post, error (ZuulFilter를 상속받아 구현)
- Order
	: filter 실행 순서를 결정 (filterOrder())
- Criteria
	: filter 실행 여부 결정 (shouldFilter())
- Action
	: filter criteria 만족 시 실행할 비즈니스 로직 (run())

```java
@Component
public class PreFilter extends ZuulFilter {
    private static Logger logger = LoggerFactory.getLogger(PreFilter.class);

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();

        logger.info((String.format("%s request to %s",
                request.getMethod(),
                request.getRequestURL().toString())));

        return null;
    }
}
```

## Config Server + Zuul + Eureka 
### Config Server
- 서비스의 환경 설정을 중앙에서 관리하고 실시간으로 설정을 변경
- 각각 서비스에 중복으로 존재하던 설정값을 관리할 수 있음

<img src="./images/config server.png" width="50%" height="50%"/>

> 아직 구현 안됨
