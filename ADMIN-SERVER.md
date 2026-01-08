---

# Spring Boot Admin Server (Monitoring & Observability) – 2025.1.0

![Image](https://miro.medium.com/1%2A5N9tJm4fDuxvpIeCHAWjqQ.png)

![Image](https://elang2.github.io/images/sb-admin/spring-boot-admin.png)

![Image](https://zoltanaltfatter.com/images/2018-05-15/eureka.png)

---

## 1. Overview

A **Spring Boot Admin Server** provides a **centralized UI** to monitor and manage Spring Boot applications using **Actuator endpoints**.

It gives visibility into:

* Application health
* Metrics (JVM, memory, CPU)
* Environment & configuration
* Logs
* Thread dumps
* HTTP traces

**Spring Boot Admin** is maintained by the Spring ecosystem and integrates seamlessly with **Netflix Eureka**.

---

## 2. Why Spring Boot Admin?

Without an Admin Server:

* Each service exposes its own Actuator endpoints
* No centralized monitoring
* Hard to debug production issues
* Manual access to each service

✅ **Spring Boot Admin solves this by:**

* Discovering services automatically
* Aggregating Actuator data
* Providing a web-based dashboard
* Supporting security & alerts

---

## 3. Architecture Flow

1. Microservices expose Actuator endpoints
2. Admin Server registers with Eureka
3. Microservices register with Eureka
4. Admin Server discovers services via Eureka
5. Admin UI displays real-time health & metrics

---

## 4. Prerequisites

| Requirement       | Version                        |
| ----------------- | ------------------------------ |
| Java              | **21 (LTS)**                   |
| Spring Boot       | **3.3.x or later**             |
| Spring Cloud      | **2024.x (2025.1.0 baseline)** |
| Service Discovery | Eureka Server                  |
| Observability     | Spring Boot Actuator           |

---

## 5. Create the Admin Server

### 5.1 Create a New Spring Boot Project

**Project Name**

```
admin-server
```

---

## 6. Maven Configuration (pom.xml)

### ✅ Correct & Updated Dependencies

```xml
	<properties>
		<java.version>21</java.version>
		<spring-boot-admin.version>3.5.6</spring-boot-admin.version>
		<spring-cloud.version>2025.0.1</spring-cloud.version>
	</properties>
	<dependencies>
		<!-- Actuator -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

		<!-- Web -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<!-- Spring Boot Admin Server -->
		<dependency>
			<groupId>de.codecentric</groupId>
			<artifactId>spring-boot-admin-starter-server</artifactId>
		</dependency>

		<!-- Eureka Client -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<!-- Security (Recommended) -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
			<dependency>
				<groupId>de.codecentric</groupId>
				<artifactId>spring-boot-admin-dependencies</artifactId>
				<version>${spring-boot-admin.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement></dependencies>
```

📌 **Important**

* Spring Boot Admin has **its own versioning**
* Do **NOT** import Spring Cloud dependencies here unless needed

---

## 7. Enable Admin Server

### 7.1 Main Application Class (CORRECT)

```java
package com.jodo.service.admin;

import de.codecentric.boot.admin.server.config.EnableAdminServer;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableAdminServer
@EnableDiscoveryClient
public class AdminServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(AdminServiceApplication.class, args);
	}

}
```

---

## 8. Admin Server Configuration

### 8.1 application.yml (Recommended)

```yaml

spring:
  application:
    name: ADMIN-SERVICE

server:
  port: 8082

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8081/eureka/
    register-with-eureka: true
    fetch-registry: true

management:
  endpoints:
    web:
      exposure:
        include: health,info
        include: "*"
```

📌 Admin Server itself must expose Actuator endpoints.

---

## 9. Access the Admin UI

Start the Admin Server and open:

```
http://localhost:8082
```

You will see:

* Registered applications
* Health status
* Metrics dashboard

---

## 10. Register Microservices with Admin Server

### 10.1 Add Actuator Dependency (Microservices)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

### 10.2 Microservice application.yml

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,loggers,env,threaddump

management:
  endpoint:
    health:
      show-details: always
```

📌 **No direct Admin Server URL is needed** when using Eureka.

---

## 11. How Discovery Works (Important)

* Admin Server queries Eureka periodically
* Finds all registered services
* Checks which ones expose Actuator
* Automatically registers them in the UI

✔ Zero manual configuration
✔ Scales automatically
✔ Works across environments

---

## 12. Security (STRONGLY RECOMMENDED)

### 12.1 Basic Security Example

```yaml
spring:
  security:
    user:
      name: admin
      password: admin123
```

📌 In production:

* Use OAuth2 / Keycloak
* Restrict Actuator endpoints
* Use HTTPS

---

## 13. Common Mistakes (Avoid These)

❌ Forgetting Actuator on services
❌ Not exposing health details
❌ Blocking Actuator endpoints with security
❌ Mixing Admin Server & Config Server roles

---

## 14. When NOT to Use Spring Boot Admin

| Environment  | Alternative                |
| ------------ | -------------------------- |
| Kubernetes   | Prometheus + Grafana       |
| Cloud-native | Cloud provider monitoring  |
| Serverless   | Native observability tools |

---
