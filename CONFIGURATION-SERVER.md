
---

# Centralized Configuration with Spring Cloud Config Server (2025.1.0)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2ASckDaXxM3o9nds3FZMZIzQ.png)

![Image](https://javatechonline.com/wp-content/uploads/2022/04/SpringCloudConfigServer-1.jpg)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AJ7SMr1pE98CmMi8Yb2limg.jpeg)

## 1. Overview

A **Config Server** provides **centralized and externalized configuration** for all microservices in a distributed system.

Instead of embedding `application.properties` inside every service, configuration is stored in an external repository (typically Git) and fetched dynamically at runtime.

**Spring Cloud Config** is the official Spring solution for centralized configuration management and integrates seamlessly with **Netflix Eureka**.

---

## 2. Why Use a Config Server?

In a microservices architecture:

* Configuration changes frequently
* Multiple services share common properties
* Environment-specific configurations are required (dev / qa / prod)
* Restarting services for config changes increases downtime

✅ **Spring Cloud Config Server solves these problems by:**

* Centralizing configuration
* Supporting environment-based profiles
* Enabling runtime refresh
* Versioning configuration via Git

---

## 3. Configuration Flow (Runtime)

1. Config Server loads configuration from Git
2. Config Server registers itself with Eureka
3. Microservices start and request configuration
4. Services discover the Config Server dynamically via Eureka
5. Configuration is applied before the application context is fully initialized

---

## 4. Prerequisites

| Requirement      | Version                        |
| ---------------- | ------------------------------ |
| Java             | **21 (LTS)**                   |
| Spring Boot      | **3.3.x or later**             |
| Spring Cloud     | **2024.x (2025.1.0 baseline)** |
| Build Tool       | Maven 3.9+                     |
| Service Registry | Running Eureka Server          |
| Config Backend   | Git (local or remote)          |

---

## 5. Creating the Config Server

### 5.1 Create a New Spring Boot Project

**Project Name**

```
config-server
```

---

### 5.2 Maven Dependencies (pom.xml)

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
    <!-- Spring Cloud Config Server -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <!-- Eureka Client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <!-- Web Layer -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Observability -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

---

## 6. Enable the Config Server

### 6.1 Main Application Class

```java
package com.example.configserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableConfigServer
@EnableDiscoveryClient
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

📌 `@EnableDiscoveryClient` allows the Config Server to register with Eureka.

---

## 7. Config Server Configuration

### 7.1 application.yml (Recommended)

```yaml
spring:
  application:
    name: config-server

  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo
          default-label: main
          clone-on-start: true

server:
  port: 8888

eureka:
  client:
    service-url:
      defaultZone: http://localhost:9092/eureka/
    register-with-eureka: true
    fetch-registry: true

management:
  endpoints:
    web:
      exposure:
        include: health,info
```
OR
### 7.1 application.properties 
```properties
# Application name (important for discovery)
spring.application.name=config-server

# Server port
server.port=8888

# Register with Eureka
eureka.client.service-url.defaultZone=http://localhost:9092/eureka/
eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true

# Git-backed configuration
spring.cloud.config.server.git.uri=https://github.com/your-org/config-repo
spring.cloud.config.server.git.default-label=main
spring.cloud.config.server.git.clone-on-start=true

# Actuator
management.endpoints.web.exposure.include=health,info

```


### Supported Configuration Backends

📌 Spring Cloud Config supports:

* Public Git repositories
* Private Git (SSH / token-based auth)
* Local filesystem repositories
* HashiCorp Vault
* AWS S3 / GCS (advanced setups)

---

## 8. Configuration Repository Structure (Git)

### 8.1 Recommended Repository Layout

```text
config-repo/
 ├── application.yml
 ├── user-service.yml
 ├── order-service.yml
 ├── user-service-dev.yml
 └── user-service-prod.yml
```

---

### 8.2 Example: user-service.yml

```yaml
server:
  port: 8081

message: Hello from Config Server
```

### Configuration Resolution Order

1. `{application}-{profile}.yml`
2. `{application}.yml`
3. `application-{profile}.yml`
4. `application.yml`

---

## 9. Verifying the Config Server

Start the Config Server and open:

```
http://localhost:8888/user-service/default
```

### Expected Result

A JSON response containing resolved configuration properties.

✔ Confirms Git connectivity
✔ Confirms service naming
✔ Confirms profile resolution

---

## 10. Connecting Microservices to the Config Server

### 10.1 Add Config Client Dependency

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

---

### 10.2 Microservice Configuration (application.yml)

```yaml
spring:
  application:
    name: user-service

  config:
    import:optional:configserver:http://localhost:9095
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server

eureka:
  client:
    service-url:
      defaultZone: http://localhost:9092/eureka/

management:
  endpoints:
    web:
      exposure:
        include: health,info
```
OR
Update application.properties (Microservice)

```properties
spring.application.name=user-service

spring.config.import=optional:configserver:http://localhost:9095
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.service-id=config-server

eureka.client.service-url.defaultZone=http://localhost:9092/eureka/
```
📌 Microservices now discover Config Server via Eureka.


📌 **Key Advantage**
Microservices **discover the Config Server dynamically via Eureka**, allowing:

* Zero hard-coded URLs
* High availability Config Server clusters
* Seamless failover

---

## 11. Refreshing Configuration (Optional)

### 11.1 Enable Refresh Endpoint

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,refresh
```

---

### 11.2 Trigger Configuration Refresh

```http
POST http://localhost:8081/actuator/refresh
```

✔ Reloads updated configuration
✔ No service restart required

> 🔔 **Enterprise Recommendation**
> Use **Spring Cloud Bus (Kafka / RabbitMQ)** for broadcasting refresh events across all services.

---

## 12. Best Practices (2025 – Enterprise Grade)

### ✅ Mandatory

* Externalize all environment-specific configuration
* Separate dev / qa / prod profiles
* Protect Config Server endpoints
* Version-control configuration changes

### ✅ Recommended

* Use **Vault** for secrets
* Enable **Config Server HA**
* Combine with **Spring Cloud Bus**
* Enable **OpenTelemetry tracing**

### ❌ Avoid

* Storing secrets in Git
* Hard-coded Config Server URLs
* Restart-based configuration updates

---

## 13. When NOT to Use Spring Cloud Config

| Environment     | Preferred Alternative   |
| --------------- | ----------------------- |
| Kubernetes      | ConfigMaps / Secrets    |
| Serverless      | Provider-native config  |
| Small monoliths | Local `application.yml` |

---
