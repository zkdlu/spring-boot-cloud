# Eureka
- 마이크로 서비스들의 정보를 레지스트리에 등록할 수 있도록 하고 마이크로 서비스의 동적인 탐색과 로드밸런싱을 제공

# Zuul
- 모든 마이크로 서비스에 대한 요청을 먼저 받아들이고 라우팅하는 프록시 API Gateway기능 수행

## 1. Eureka Server
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
