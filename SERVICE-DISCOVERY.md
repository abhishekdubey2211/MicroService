---

# Service Registry with Spring Boot (2025.1.0 – Professional Edition)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2A2_UwLm-IShplbqM-jFJtZA.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AOg7drtttjN5gNzBG36zxdw.png)

![Image](https://miro.medium.com/0%2Aj821tfro1lxZ9z0i.png)

## 1. Overview

This guide demonstrates how to implement a **Service Registry** using **Spring Boot** and **Netflix Eureka** with **Spring Cloud 2024.x**.

A **Service Registry** provides:

* Dynamic service registration
* Runtime service discovery
* Load-balanced communication between microservices
* Decoupling of service locations from service consumers

### The Core Problem in Microservices

In a distributed system:

* Services are independently deployed
* Instances scale dynamically (horizontal scaling)
* IP addresses and ports change frequently
* Hard-coded URLs break resiliency and elasticity

❌ **Anti-pattern**

```
http://10.0.1.15:8080
```

✅ **Solution**

```
http://USER-SERVICE
```

Service discovery resolves the actual instance at runtime.

### Popular Service Registries (Java Ecosystem – 2025)

| Technology       | Typical Usage                 |
| ---------------- | ----------------------------- |
| Netflix Eureka   | Spring Cloud (non-Kubernetes) |
| HashiCorp Consul | Hybrid / multi-cloud          |
| Apache Zookeeper | Legacy / coordination         |
| Kubernetes DNS   | Cloud-native clusters         |

> ⚠️ **Note:** If you are running fully on Kubernetes, native service discovery is recommended over Eureka.

---

## 2. Prerequisites

| Requirement  | Version                          |
| ------------ | -------------------------------- |
| Java         | **21 (LTS)**                     |
| Spring Boot  | **3.3.x or later**               |
| Spring Cloud | **2024.0.x (2025.1.0 baseline)** |
| Build Tool   | Maven 3.9+                       |

---

## 3. Eureka Server (Service Registry)

### 3.1 Maven Configuration (pom.xml)

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
    <!-- Eureka Server -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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

### 3.2 Configuration (application.yml – Recommended)

```yaml
spring:
  application:
    name: service-registry

server:
  port: 9092

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: true
    eviction-interval-timer-in-ms: 60000

management:
  endpoints:
    web:
      exposure:
        include: health,info
```
OR
### 3.2 Create src/main/resources/application.properties:
```properties
# Application name
spring.application.name=service-registry

# Eureka Server port
server.port=9092

# Eureka instance hostname
eureka.instance.hostname=localhost

# Do NOT register Eureka Server with itself
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

# Eureka server behavior
eureka.server.enable-self-preservation=true
eureka.server.eviction-interval-timer-in-ms=60000

```
#### Why YAML?

* Better readability
* Cloud-native standard
* Strongly preferred in enterprise projects

---

### 3.3 Eureka Server Bootstrap Class

```java
package com.example.registry;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class ServiceRegistryApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceRegistryApplication.class, args);
    }
}
```

---

### 3.4 Running the Eureka Server

```bash
mvn clean package
java -jar target/service-registry-0.0.1-SNAPSHOT.jar
```

Dashboard URL:

```
http://localhost:9092
```

> 🔐 **Production Recommendation**
> Run **at least 3 Eureka servers** behind a load balancer with peer awareness enabled.

---

## 4. Registering Microservices with Eureka

### 4.1 Microservice Dependencies

```xml
<dependencies>
    <!-- Eureka Client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <!-- REST APIs -->
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

### 4.2 Microservice Configuration (application.yml)

```yaml
spring:
  application:
    name: user-service

server:
  port: 8081

eureka:
  client:
    service-url:
      defaultZone: http://localhost:9092/eureka/
    register-with-eureka: true
    fetch-registry: true
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 30

management:
  endpoints:
    web:
      exposure:
        include: health,info
```

---

### 4.3 Running the Microservice

```bash
mvn spring-boot:run
```

Verify registration:

```
http://localhost:9092
```

You should see:

```
USER-SERVICE (1 instance)
```

---

## 5. Professional Best Practices (2025)

### ✅ Mandatory

* Use **Actuator health checks**
* Externalize config (Spring Cloud Config / Vault)
* Enable structured logging (JSON)
* Use **Resilience4j** instead of Hystrix (deprecated)

### ✅ Recommended

* Replace `RestTemplate` with **WebClient**
* Use **Spring Cloud LoadBalancer**
* Enable distributed tracing (OpenTelemetry)

### ❌ Avoid

* Hard-coded service URLs
* Single Eureka node in production
* Eureka inside Kubernetes clusters

---
