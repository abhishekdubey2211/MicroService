---

# API Gateway using Spring Cloud Gateway (2025.1.0)

![Image](https://i.postimg.cc/kDP2Dj23/Spring-Cloud-Architecture.png)

![Image](https://assets.bytebytego.com/diagrams/0252-lb-api-gateway.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AopwoTH3vt54NZ2RUuU5Www.png)

---

## 1. Overview

An **API Gateway** acts as a **single entry point** for all client requests in a microservices architecture.
It sits between clients and backend services and applies **cross-cutting concerns** consistently.

**Spring Cloud Gateway** is the official Spring-based API Gateway built on **WebFlux** and **Project Reactor**.

---

## 2. Responsibilities of an API Gateway

### Core Responsibilities

* Routing requests to the correct backend service
* Authentication & Authorization (JWT, OAuth2, API keys)
* Rate limiting & throttling
* Request / response transformation
* Caching
* API versioning
* Centralized logging & monitoring

---

## 3. API Gateway vs Load Balancer (Important Clarification)

### What Is a Load Balancer?

A **Load Balancer** distributes traffic **only across instances of the same service** to improve:

* Scalability
* Availability
* Fault tolerance

### Typical Load Balancer Responsibilities

* Traffic distribution (round-robin, weighted, etc.)
* Health checks
* Failover
* SSL termination (sometimes)

---

### Key Difference (Corrected)

| API Gateway                        | Load Balancer                       |
| ---------------------------------- | ----------------------------------- |
| Entry point for all APIs           | Instance-level traffic distribution |
| Handles auth, rate limits, routing | Only balances traffic               |
| Aware of API paths                 | Not API-aware                       |
| Client-facing                      | Infrastructure-facing               |

📌 **Spring Cloud Gateway internally uses Spring Cloud LoadBalancer** when `lb://SERVICE-NAME` is used.

---

## 4. Common Java-Friendly API Gateways

* **Spring Cloud Gateway** (recommended for Spring Boot)
* Netflix Zuul (legacy / deprecated)
* Kong (language-agnostic)
* AWS API Gateway (managed)
* NGINX (gateway + reverse proxy)

---

## 5. Prerequisites (Corrected)

| Requirement       | Version                        |
| ----------------- | ------------------------------ |
| Java              | **17+ (21 recommended)**       |
| Spring Boot       | **3.3.x or later**             |
| Spring Cloud      | **2024.x (2025.1.0 baseline)** |
| Service Discovery | Eureka or Kubernetes           |
| Stack             | Reactive (WebFlux)             |

---

## 6. Required Dependencies (API Gateway)
### ✅ pom.xml

```xml
<properties>
    <java.version>21</java.version>
    <spring-cloud.version>2024.0.0</spring-cloud.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>

    <!-- API Gateway (includes WebFlux) -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <!-- Service Discovery -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <!-- Load Balancer -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>

    <!-- Centralized Config (Optional) -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <!-- Monitoring -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

</dependencies>
```

---

## 7. API Gateway Main Class

### ✅  Main Class

```java
package com.example.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

📌 No additional annotations are required for Spring Cloud Gateway.

---

## 8. Route Configuration (YAML – Recommended)

### ✅ Correct application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

  cloud:
    gateway:

      discovery:
        locator:
          enabled: true
          lower-case-service-id: true

      httpclient:
        connect-timeout: 5000
        response-timeout: 10s

      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/user/**
          filters:
            - StripPrefix=1

        - id: payment-service
          uri: lb://PAYMENT-SERVICE
          predicates:
            - Path=/payment/**
          filters:
            - StripPrefix=1

        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/order/**
          filters:
            - StripPrefix=1

        - id: product-service
          uri: lb://PRODUCT-SERVICE
          predicates:
            - Path=/product/**
          filters:
            - StripPrefix=1

        - id: cart-service
          uri: lb://CART-SERVICE
          predicates:
            - Path=/cart/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                redis-rate-limiter.requestedTokens: 1
```

---

## 9. Why YAML Is Preferred

* No recompilation required
* Centralized route control
* Easy production updates
* Compatible with Spring Cloud Config

---

## 10. Header Propagation Using GlobalFilter (Best Practice)

```java
@Component
public class HeaderForwardingFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        ServerHttpRequest request = exchange.getRequest();
        HttpHeaders headers = request.getHeaders();

        String authHeader = headers.getFirst(HttpHeaders.AUTHORIZATION);
        String correlationId = headers.getFirst("X-Correlation-Id");

        if (correlationId == null) {
            correlationId = UUID.randomUUID().toString();
        }

        ServerHttpRequest mutatedRequest = request.mutate()
                .header("X-Request-Id", UUID.randomUUID().toString())
                .header("X-Correlation-Id", correlationId)
                .headers(h -> {
                    if (authHeader != null) {
                        h.set(HttpHeaders.AUTHORIZATION, authHeader);
                    }
                })
                .build();

        return chain.filter(exchange.mutate().request(mutatedRequest).build());
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

### Headers Forwarded

* `Authorization` → security context
* `X-Request-Id` → per-request tracing
* `X-Correlation-Id` → distributed tracing

---

## 11. How Load Balancing Works (Correct Explanation)

When using:

```yaml
uri: lb://USER-SERVICE
```

Flow:

1. Gateway queries Eureka
2. All healthy instances are retrieved
3. Requests are distributed automatically (round-robin)
4. Failed instances are skipped

✔ No custom code required
✔ Fully reactive & non-blocking

---

## 12. Backend Service Header Consumption

```java
@GetMapping("/status")
public String status(
        @RequestHeader("X-Request-Id") String requestId,
        @RequestHeader("X-Correlation-Id") String correlationId,
        @RequestHeader(value = "Authorization", required = false) String auth) {

    log.info("requestId={}, correlationId={}", requestId, correlationId);
    return "OK";
}
```

---

## 13. Optional: Gateway Actuator Endpoints

```yaml
management:
  endpoints:
    web:
      exposure:
        include: gateway, health
```
---

## 9. Alternative: Java-Based Route Configuration (Optional)

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20230515200450/Android-amia-client-API-gateway-microservice-01.webp)

![Image](https://i.postimg.cc/kDP2Dj23/Spring-Cloud-Architecture.png)

While **YAML-based routing is recommended for most production use cases**, Spring Cloud Gateway also supports **Java-based route definitions** using `RouteLocator`.

**Spring Cloud Gateway** allows routes to be declared programmatically when more control is required.

---

## 9.1 When to Use Java-Based Routes

Java-based routing is useful when you need:

* Dynamic route creation at runtime
* Conditional routing logic
* Integration with databases or feature flags
* Complex filter logic that is hard to express in YAML

⚠️ **Trade-off**
Java routes require **code changes and redeployment**, whereas YAML routes can be updated dynamically (especially with Spring Cloud Config).

---

## 9.2 Correct Java RouteLocator Configuration

### ✅ Corrected & Production-Safe Version

```java
package com.example.gateway.config;

import java.util.UUID;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.cloud.gateway.route.builder.GatewayFilterSpec;
import org.springframework.http.HttpHeaders;
import org.springframework.http.server.reactive.ServerHttpRequest;

@Configuration
public class GatewayRoutesConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()

            .route("user-service", r -> r
                .path("/user/**")
                .filters(this::commonFilters)
                .uri("lb://USER-SERVICE"))

            .route("payment-service", r -> r
                .path("/payment/**")
                .filters(this::commonFilters)
                .uri("lb://PAYMENT-SERVICE"))

            .route("order-service", r -> r
                .path("/order/**")
                .filters(this::commonFilters)
                .uri("lb://ORDER-SERVICE"))

            .route("product-service", r -> r
                .path("/product/**")
                .filters(this::commonFilters)
                .uri("lb://PRODUCT-SERVICE"))

            .route("cart-service", r -> r
                .path("/cart/**")
                .filters(this::commonFilters)
                .uri("lb://CART-SERVICE"))

            .build();
    }

    private GatewayFilterSpec commonFilters(GatewayFilterSpec f) {
        return f.filter((exchange, chain) -> {

            ServerHttpRequest request = exchange.getRequest();
            HttpHeaders headers = request.getHeaders();

            String authHeader = headers.getFirst(HttpHeaders.AUTHORIZATION);
            String correlationId = headers.getFirst("X-Correlation-Id");

            if (correlationId == null) {
                correlationId = UUID.randomUUID().toString();
            }

            ServerHttpRequest mutatedRequest = request.mutate()
                .header("X-Request-Id", UUID.randomUUID().toString())
                .header("X-Correlation-Id", correlationId)
                .headers(h -> {
                    if (authHeader != null) {
                        h.set(HttpHeaders.AUTHORIZATION, authHeader);
                    }
                })
                .build();

            return chain.filter(
                exchange.mutate()
                    .request(mutatedRequest)
                    .build()
            );
        });
    }
}
```

---

## 9.3 What Was Corrected (Important)

### ❌ Issues Fixed

* Avoided overwriting `Authorization` with `null`
* Ensured headers are conditionally propagated
* Used method reference `this::commonFilters` (cleaner & safer)
* Ensured reactive, non-blocking behavior

### ✅ Result

* Fully reactive
* Safe header propagation
* Load-balanced via Eureka
* Production-ready

---

## 10. Headers Passed to Backend Services

Every request forwarded by the Gateway will contain:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
X-Request-Id: 9c7cdb5e-9e7d-4c91-a92a-8e123456abcd
X-Correlation-Id: 3f2e1c99-bc45-4ef8-90d4-abc987654321
```

---

## 11. Why These Headers Matter

| Header             | Purpose                         |
| ------------------ | ------------------------------- |
| `Authorization`    | Security context (JWT / OAuth2) |
| `X-Request-Id`     | Per-request tracing             |
| `X-Correlation-Id` | End-to-end distributed tracing  |

These headers are critical for:

* Observability
* Debugging production issues
* Log correlation across services
* OpenTelemetry / Zipkin / Jaeger tracing

---

## 12. How Backend Java Services Consume These Headers

### Example: REST Controller

```java
@GetMapping("/profile")
public ResponseEntity<String> profile(
        @RequestHeader("X-Request-Id") String requestId,
        @RequestHeader("X-Correlation-Id") String correlationId,
        @RequestHeader("Authorization") String auth) {

    log.info("requestId={}, correlationId={}", requestId, correlationId);
    return ResponseEntity.ok("User Profile");
}
```

✔ Headers are automatically forwarded
✔ No gateway-specific code required in backend
✔ Works with MVC or WebFlux services

---
