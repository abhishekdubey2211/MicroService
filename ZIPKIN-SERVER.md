
---

# Zipkin Distributed Tracing (Observability) – 2025.1.0

![Image](https://dz2cdn1.dzone.com/storage/temp/14019791-part12-overview.png)

![Image](https://media.licdn.com/dms/image/v2/D5612AQFkni3U3AOP_A/article-cover_image-shrink_720_1280/article-cover_image-shrink_720_1280/0/1723434431208?e=2147483647\&t=wvyGl6zUv-Chkc_yL0i4By5_YFl77koKhlrvCr4gXRE\&v=beta)

![Image](https://zipkin.io/public/img/web-screenshot.png)

![Image](https://blog.risingstack.com/wp-content/uploads/2021/07/Distributed_transaction_tracing_with_service_topology_map_trace_by_risingstack_1462456507669-1466682320244.png)

---

## 1. Overview

**Zipkin** is a **distributed tracing system** used to **track and visualize requests** as they flow across microservices.

With Zipkin, you can:

* Trace requests end-to-end
* Identify slow services
* Analyze latency bottlenecks
* Debug production issues quickly
* Understand service dependencies

Zipkin works seamlessly with **Spring Boot 3.x** using **Micrometer Tracing**.

---

## 2. Why Zipkin?

Without distributed tracing:

❌ You only see logs per service
❌ Hard to correlate requests
❌ No visibility into inter-service latency
❌ Debugging production issues is painful

✅ **Zipkin solves this by:**

* Assigning a **Trace ID** per request
* Propagating it across services
* Collecting spans centrally
* Visualizing the entire request lifecycle

---

## 3. Architecture Flow

![Image](https://zipkin.io/public/img/architecture-1.png)

![Image](https://dataflow.spring.io/static/a2351db00e7aa110b3742d85003dbed7/a331c/SCDF-stream-tracing-architecture.png)

1. Client sends request → API Gateway / Service A
2. Trace ID is generated
3. Trace ID propagates to downstream services
4. Each service creates spans
5. Spans are sent to Zipkin Server
6. Zipkin UI visualizes the trace

---

## 4. Prerequisites

| Requirement  | Version                 |
| ------------ | ----------------------- |
| Java         | **21 (LTS)**            |
| Spring Boot  | **3.3.x or later**      |
| Spring Cloud | **2024.x (2025.1.0)**   |
| Tracing      | Micrometer Tracing      |
| Transport    | HTTP / Kafka (optional) |

---

## 5. Zipkin Server Setup

### 5.1 Run Zipkin Server (Recommended – Docker)

```bash
docker run -d -p 9411:9411 --name zipkin openzipkin/zipkin
```

📌 Zipkin UI will be available at:

```
http://localhost:9411
```

---

### 5.2 Zipkin Server (Spring Boot – NOT RECOMMENDED)

⚠️ Zipkin Server **should NOT be embedded** in Spring Boot anymore.
Always run it as a **standalone service** (Docker / Kubernetes).

---

## 6. Integrate Zipkin with Microservices

### 6.1 Required Dependencies (Spring Boot 3.x)

```xml
<!-- Micrometer Tracing -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>

<!-- Zipkin Reporter -->
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>

<!-- Actuator (Mandatory) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

📌 **Important**

* Spring Cloud Sleuth is **REMOVED**
* Micrometer Tracing replaces Sleuth
* Brave is the default tracing bridge

---

## 7. Zipkin Configuration (Microservices)

### 7.1 application.yml

```yaml
spring:
  application:
    name: order-service

management:
  tracing:
    sampling:
      probability: 1.0   # 100% sampling (use lower in prod)
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
```

📌 **Sampling Notes**

| Environment | Probability |
| ----------- | ----------- |
| Dev         | 1.0         |
| QA          | 0.5         |
| Prod        | 0.1 or less |

---

## 8. Trace Propagation (Automatic)

Zipkin automatically propagates headers:

```
X-B3-TraceId
X-B3-SpanId
X-B3-ParentSpanId
X-B3-Sampled
```

✔ Works with:

* RestTemplate
* WebClient
* Feign
* Spring Cloud Gateway

No manual configuration required.

---

## 9. Viewing Traces in Zipkin UI

1. Open Zipkin UI

   ```
   http://localhost:9411
   ```

2. Select Service Name

3. Click **Find Traces**

4. Inspect:

   * Latency
   * Spans
   * Errors
   * Dependencies

![Image](https://lumigo.io/wp-content/uploads/2023/07/unnamed-5.png)

![Image](https://i.sstatic.net/G1Pnm.png)

![Image](https://kamon.io/assets/img/recipes/quickstart-zipkin-span-detail.png)

---

## 10. Logging with Trace ID (BEST PRACTICE)

### 10.1 Logback Configuration

```xml
<pattern>
    %d{yyyy-MM-dd HH:mm:ss} [%X{traceId:-}] [%X{spanId:-}] %-5level %logger{36} - %msg%n
</pattern>
```
Example  For using (SLF4J + Log4j2) exclude default logback config from all starter dependencies 
```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
      <scope>provided</scope>
    </dependency>

<!--  Log4j2 (includes log4j-core + slf4j bridge automatically)  -->
  <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-log4j2</artifactId>
 </dependency>

```
📍 Location --> src/main/resources/log4j2-spring.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">

    <!-- ================================================= -->
    <!--                 CONFIGURATION PROPERTIES          -->
    <!-- ================================================= -->
    <Properties>
        <Property name="LOG_FILE_NAME">${spring:log.file.name:-application}</Property>
        <Property name="APP_NAME">${spring:spring.application.name:-APP}-${spring:server.port}</Property>
        <Property name="ROTATION_INTERVAL">${spring:log.rotation.interval:-1}</Property>
        <Property name="LOG_RETENTION_DAYS">${spring:log.retention.days:-7}</Property>
        <Property name="LOG_FILE_PATH">${spring:log.file.path:-./logs}</Property>
        <Property name="ROOT_LOG_LEVEL">${spring:log.root.level:-INFO}</Property>
        <Property name="FILE_SIZE">${spring:log.file.size:-100MB}</Property>

        <!-- ✅ Centralized log pattern (single source of truth) -->
        <Property name="LOG_PATTERN">
            %d{yyyy-MM-dd HH:mm:ss.SSS} %-5p [%t] [${APP_NAME}]
            [%X{traceId:-}] [%X{spanId:-}]
            %notEmpty{[%X{correlationId}]}
            %c{2}%notEmpty{.%M}%notEmpty{ [%L]} -- %m%n
        </Property>
    </Properties>

    <!-- ================================================= -->
    <!--                    APPENDERS                     -->
    <!-- ================================================= -->
    <Appenders>

        <!-- ================================================= -->
        <!-- MAIN ROLLING FILE APPENDER -->
        <!-- ================================================= -->
        <RollingFile name="RollingFileAppender"
                     fileName="${LOG_FILE_PATH}/${LOG_FILE_NAME}-live.log"
                     filePattern="${LOG_FILE_PATH}/${LOG_FILE_NAME}-%d{yyyy-MM-dd-HH}.log.gz">

            <PatternLayout charset="UTF-8" pattern="${LOG_PATTERN}"/>

            <!-- Time + size based rotation -->
            <Policies>
                <TimeBasedTriggeringPolicy interval="${ROTATION_INTERVAL}" modulate="true"/>
                <SizeBasedTriggeringPolicy size="${FILE_SIZE}"/>
            </Policies>

            <!-- Retention policy -->
            <DefaultRolloverStrategy>
                <Delete basePath="${LOG_FILE_PATH}" maxDepth="1">
                    <IfFileName glob="${LOG_FILE_NAME}-*.log.gz"/>
                    <IfLastModified age="${LOG_RETENTION_DAYS}d"/>
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>

        <!-- ================================================= -->
        <!-- ERROR LOG FILE -->
        <!-- ================================================= -->
        <RollingFile name="ErrorFile"
                     fileName="${LOG_FILE_PATH}/error.log"
                     filePattern="${LOG_FILE_PATH}/error-%d{yyyy-MM-dd}.log.gz">

            <PatternLayout charset="UTF-8" pattern="${LOG_PATTERN}"/>

            <Policies>
                <TimeBasedTriggeringPolicy interval="24" modulate="true"/>
            </Policies>

            <Filters>
                <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingFile>

        <!-- ================================================= -->
        <!-- CONSOLE OUTPUT -->
        <!-- ================================================= -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout charset="UTF-8" pattern="${LOG_PATTERN}"/>
        </Console>

    </Appenders>

    <!-- ================================================= -->
    <!--                    LOGGERS                        -->
    <!-- ================================================= -->
    <Loggers>

        <!-- Spring Framework -->
        <Logger name="org.springframework" level="INFO"/>
        <Logger name="org.springframework.web" level="INFO"/>

        <!-- Hibernate / JPA -->
        <Logger name="org.hibernate" level="WARN"/>

        <!-- Feign -->
        <Logger name="feign" level="INFO"/>

        <!-- Application Root -->
        <!-- AsyncRoot used for non-blocking logging -->
        <AsyncRoot level="${ROOT_LOG_LEVEL}">
            <AppenderRef ref="RollingFileAppender"/>
            <AppenderRef ref="ErrorFile"/>
            <AppenderRef ref="Console"/>
        </AsyncRoot>

    </Loggers>

</Configuration>
</Configuration>

```
📌 This allows you to **correlate logs with traces**.

---

## 11. Zipkin with Eureka (Optional)

Zipkin does **NOT** require Eureka.

However:

* Services discovered via Eureka
* Zipkin groups traces by `spring.application.name`

✔ Fully compatible with your existing architecture

---

## 12. Common Mistakes (Avoid These)

❌ Using Spring Cloud Sleuth
❌ Forgetting sampling configuration
❌ Not running Zipkin server
❌ Blocking `/api/v2/spans` endpoint
❌ Expecting Zipkin to store data forever (it does not)

---

## 13. Production Recommendations

| Concern   | Recommendation            |
| --------- | ------------------------- |
| Storage   | Use Elasticsearch         |
| Sampling  | < 10%                     |
| Transport | Kafka (high traffic)      |
| Security  | Network-restricted Zipkin |
| Retention | External storage          |

---

## 14. Zipkin vs Other Tracing Tools

| Tool          | Use Case            |
| ------------- | ------------------- |
| Zipkin        | Simple, lightweight |
| Jaeger        | Kubernetes heavy    |
| OpenTelemetry | Vendor-neutral      |
| New Relic     | SaaS                |

---

## 15. Final Architecture (With Admin + Zipkin)

![Image](https://raw.githubusercontent.com/aelkz/microservices-observability/master/_images/intro.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A805/0%2Ah2nTiP_zOkpyi_bd.png)

✔ Spring Boot Admin → **Health & Metrics**
✔ Zipkin → **Distributed Tracing**
✔ Eureka → **Service Discovery**

---
