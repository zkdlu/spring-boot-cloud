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

## 2. Eureka Client
1. 의존성 추가
```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}
```
2. application.yml 작성
```yml
spring:
  application:
    name: serviceA

eureka:
  client:
    service-url:
      defaultZone: ${EUREKA_URL:http://127.0.0.1:8787/eureka/} // default-zone이라고 하니까 안되는데 이거떄문인가 우연인가.
```
3. 어노테이션 추가
```java
@EnableDiscoveryClient
```

> - @EnableDiscoveryClient : spring-cloud-common에 존재, 다른 클라이언트 구현체(컨설,주키퍼 등)을 지원
> - @EnableEurekaClient : spring-cloud-netflix에 존재, 유레카만 지원

## 2. Zuul
1. 의존성 추가
```gradle
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-zuul'
}
```
> API Gateway도 서비스의 하나인 듯
2. application.yml 작성
```yml
spring:
  application:
    name: api-gateway-service

eureka:
  client:
    service-url:
      defaultZone: ${EUREKA_URL:http://localhost:8787/eureka}

zuul:
  routes:
    serviceA:
      path: /**
      service-id: serviceA
```
3. 어노테이션 추가
> @EnableZuulServer는 자체 라우팅 서비스를 만들고 내장된 주울 기능을 사용하지 않을 때 사용
```java
@EnableDiscoveryClient
@EnableZuulProxy
@SpringBootApplication
public class ZuulApplication {
```