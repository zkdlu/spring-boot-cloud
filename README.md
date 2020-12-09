# Eureka
- 마이크로 서비스들의 정보를 레지스트리에 등록할 수 있도록 하고 마이크로 서비스의 동적인 탐색과 로드밸런싱을 제공
- Eureka Server와 Eureka Client로 구성
## Eureka Server
- Eureka Client에 해당하는 마이크로 서비스들의 상태 정보가 등록되어 있는 레지스트리를 갖는다.
## Eureka Client
- 서비스가 시작될 때 Eureka Server에 자신의 정보를 등록한다. 등록된 후에는 30초마다 레지스트리에 ping을 전송하여 가용상태임을 알리는데 일정 횟수 이상 ping이 확인되지 않으면 Server의 레지스트리에서 제외 됨

> - 레지스트리의 정보는 모든 Eureka Client에 복제 됨. 가용 상태인 서비스 목록을 확인 가능하며 30초마다 갱신 됨.
> - 가용상태의 서비스 목록을 확인할 경우 서비스의 이름을 기준으로 탐색하여 내부적으로 Ribbon (클라이언트 측 로드밸런서)를 사용한다.

# Zuul
- 모든 마이크로 서비스에 대한 요청을 먼저 받아들이고 라우팅하는 프록시 API Gateway기능 수행

## 1. Eureka Server
1. 의존성 추가
```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
}
```
2. application.yml 작성
```yml
server:
  port: 8787

eureka:
  instance:
    hostname: 127.0.0.1
  client:
    service-url:
      default-zone: http://${eureka.instance.hostname}:${server.port}/eureka/
    register-with-eureka: false
    fetch-registry: false
```
3. @EnableEurekaServer 어노테이션 추가
```java
@EnableEurekaServer
@SpringBootApplication
public class ServerApplication {
...
```




- @EnableEurekaServer
```yml
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  server:
    response-cache-update-interval-ms: 1000
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

## 2. Eureka Client
- @EnableEurekaClient
```yml
server:
  port: 8081

spring:
  application:
    name: serviceA

eureka:
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 30
  client:
    register-with-eureka: true
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

## 3. Zuul Gateway
- @EnableZuulProxy
- `@EnableZuulServer는 자체 라우팅 서비스를 만들고 내장된 주울 기능을 사용하지 않을 때 사용`
```yml
spring:
  application:
    name: zuul-gateway

server:
  port: 8100

zuul:
  routes:
    serviceA:
      path: /service/**
      serviceId: serviceA

ribbon:
  eureka:
    enabled: false

serviceA:
  ribbon:
    listOfServers: http://localhost:8081, http://localhost:8082

eureka:
  instance:
    prefer-ip-address: true
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```
