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

# Ribbon
- 클라이언트 측 로드 밸런서
- 여러 서버를 라운드로빈 방식의 부하 분산 기능을 제공(여러 알고리즘 사용 가능)
- Spring Cloud Config와 결합하여, 서버 목록을 제공받아 사용할 수 있음


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
> - register-with-eureka: eureka의 registry에 등록할지 여부
> - fetch-registry: registry에 있는 정보를 가져올지 여부
> - eureka-server는 사용하지 않으므로 false (기본값은 true)

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
server:
  port: 8081
  
spring:
  application:
    name: serviceA

eureka:
  client:
    service-url:
      defaultZone: ${EUREKA_URL:http://127.0.0.1:8787/eureka/} // default-zone이라고 하니까 안되는데 이거떄문인가 우연인가.
ribbon:
  eureka:
    enabled: true
```
3. 어노테이션 추가
```java
@EnableDiscoveryClient
```

> - @EnableDiscoveryClient : spring-cloud-common에 존재, 다른 클라이언트 구현체(컨설,주키퍼 등)을 지원
> - @EnableEurekaClient : spring-cloud-netflix에 존재, 유레카만 지원

## 3. Zuul
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
      path: /test/**
      service-id: SERVICEA
```
3. 어노테이션 추가
> @EnableZuulServer는 자체 라우팅 서비스를 만들고 내장된 주울 기능을 사용하지 않을 때 사용
```java
@EnableDiscoveryClient
@EnableZuulProxy
@SpringBootApplication
public class ZuulApplication {
```

## 4. 테스트
- client 서비스는 localhost:8081/ 인데 zuul 라우트를 통해 localhost:8080/으로 연결 됨

## 5. 기타
- Eureka Client를 종료 했을 때 Server에 인스턴스가 내려가게 하고 싶었다.
- 맞는 방법인진 모르겠다.
1. Eureka Server 자기 보호 모드 비활성화
```yml
eureka:
  server:
    enable-self-preservation: false # 자기 보호 모드(네트워크 장애가 발생하여도 서비스 해제를 방지하는 모드)
```
2. Eureka Client 하트비트 전송
```yml
eureka:
  instance:
    lease-renewal-interval-in-seconds: 1 # 디스커버리한테 1초마다 하트비트 전송
    lease-expiration-duration-in-seconds: 2 # 디스커버리는 서비스 등록 해제 하기 전에 마지막 하트비트에서부터 2초 기다림
```

이러니까 되긴 하더라.
