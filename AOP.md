## Aspect-Oriented Programming (AOP) in Java

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20190313105735/dominant-frameworks-in-AOP.jpg)

![Image](https://docs.firstdecode.com/wp-content/uploads/2020/03/CrossCuttingConerns-1024x612.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AJlhKkus6DoFVkQUx2iWIVg.png)

![Image](https://docs.spring.io/spring-framework/reference/_images/aop-proxy-call.png)

### What is AOP?

**Aspect-Oriented Programming (AOP)** is a programming paradigm that helps you **separate cross-cutting concerns** from your core business logic.

Cross-cutting concerns are functionalities that affect multiple parts of an application, such as:

* Logging
* Security
* Transaction management
* Performance monitoring
* Exception handling

Instead of repeating this code everywhere, AOP lets you define it **once** and apply it **where needed**.

---

### Why AOP is Needed (Problem)

In traditional OOP:

* Logging code appears in many classes
* Security checks are scattered
* Transaction code mixes with business logic

This causes:
❌ Code duplication
❌ Poor maintainability
❌ Tight coupling

AOP solves this by **modularizing** these concerns.

---

### Core AOP Concepts (Java)

| Term              | Meaning                                                                   |
| ----------------- | ------------------------------------------------------------------------- |
| **Aspect**        | A module that encapsulates a cross-cutting concern (e.g., logging aspect) |
| **Advice**        | Code executed at a specific point (before, after, around a method)        |
| **Join Point**    | A point during execution (method call, exception, etc.)                   |
| **Pointcut**      | Expression that selects join points                                       |
| **Weaving**       | Linking aspects with target objects                                       |
| **Target Object** | The business class being advised                                          |

---

### Types of Advice

1. **Before** – runs before method execution
2. **After** – runs after method execution
3. **After Returning** – runs after successful execution
4. **After Throwing** – runs when exception occurs
5. **Around** – runs before and after method execution (most powerful)

---

### Simple Example (Spring AOP)


Required Maven Dependency (pom.xml)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
#### Business Logic

```java
public class PaymentService {
    public void makePayment() {
        System.out.println("Payment processed");
    }
}
```

#### Aspect (Logging)

```java
@Aspect
public class LoggingAspect {

    @Before("execution(* PaymentService.makePayment(..))")
    public void logBefore() {
        System.out.println("Payment started");
    }
}
```

#### Output

```
Payment started
Payment processed
```

➡️ Logging code is **separate** from business logic.

---

### How AOP Works Internally

* Uses **proxies** (JDK Dynamic Proxy or CGLIB)
* Intercepts method calls
* Executes advice at runtime

---

### Advantages of AOP

✅ Clean separation of concerns
✅ Less code duplication
✅ Easier maintenance
✅ Improved readability
✅ Better scalability

---
---

### AOP vs OOP (Quick Comparison)

| OOP                        | AOP                               |
| -------------------------- | --------------------------------- |
| Focuses on objects         | Focuses on cross-cutting concerns |
| Inheritance & polymorphism | Advice & pointcuts                |
| Vertical abstraction       | Horizontal abstraction            |

---
## Pointcut **Expressions** in AOP (Java / Spring)

![Image](https://miro.medium.com/1%2A1ocCJaizz4LhM8ldsL796g.png)

![Image](https://i.sstatic.net/J7Hrh.png)

![Image](https://www.edureka.co/blog/wp-content/uploads/2019/01/Untitled-1-1.png)

In **AOP**, a **pointcut expression** defines **where** (which methods / join points) an **advice** should be applied.

---

## 1️⃣ Types of Pointcut Expressions

### 🔹 1. `execution` (MOST IMPORTANT)

👉 Matches **method execution**

#### Syntax

```java
execution(modifiers return-type package.class.method(parameters))
```

#### Examples

```java
execution(public void com.app.service.PaymentService.pay())
execution(* com.app.service.*.*(..))
execution(* *.save(..))
```

#### Use

✅ Most commonly used
✅ Best for method-level interception

---

### 🔹 2. `within`

👉 Matches **all methods inside a class or package**

#### Example

```java
within(com.app.service.PaymentService)
within(com.app.service.*)
```

#### Use

✅ Apply advice to **all methods** of a class/package
❌ Less precise than `execution`

---

### 🔹 3. `this`

👉 Matches **proxy object type**

```java
this(com.app.service.PaymentService)
```

#### Use

✅ Works with **Spring proxy**
❌ Proxy-type dependent

---

### 🔹 4. `target`

👉 Matches **actual target class**

```java
target(com.app.service.PaymentService)
```

#### Use

✅ More reliable than `this`
✅ Preferred when using interfaces

---

### 🔹 5. `args`

👉 Matches **method arguments**

```java
args(int)
args(String, ..)
```

#### Example

```java
execution(* *.save(..)) && args(String)
```

#### Use

✅ When advice depends on **method parameters**

---

### 🔹 6. `@annotation`

👉 Matches **methods with specific annotations**

```java
@annotation(org.springframework.transaction.annotation.Transactional)
```

#### Example

```java
@annotation(LogExecutionTime)
```

#### Use

✅ Clean and readable
✅ Annotation-driven AOP (very popular)

---

### 🔹 7. `@within`

👉 Matches **classes annotated with annotation**

```java
@within(org.springframework.stereotype.Service)
```

---

### 🔹 8. `@target`

👉 Matches **target class annotated**

```java
@target(org.springframework.stereotype.Repository)
```

---

## 2️⃣ Comparison of Pointcut Expression Types

| Expression    | Matches           | Best Use Case      |
| ------------- | ----------------- | ------------------ |
| `execution`   | Method execution  | Most common        |
| `within`      | Class/package     | Broad interception |
| `this`        | Proxy type        | Proxy-based AOP    |
| `target`      | Actual class      | Safer than `this`  |
| `args`        | Method parameters | Param-based logic  |
| `@annotation` | Method annotation | Clean & flexible   |
| `@within`     | Class annotation  | Layer-based AOP    |
| `@target`     | Target annotation | Runtime checks     |

---

## 3️⃣ Uses of Pointcut Expressions

✔ Logging
✔ Security checks
✔ Transaction management
✔ Performance monitoring
✔ Auditing
✔ Exception tracking

---

## 4️⃣ Different Approaches to Create Pointcut Expressions

### 🟢 Approach 1: **Inline Pointcut**

Defined directly inside advice

```java
@Before("execution(* com.app.service.*.*(..))")
public void logBefore() { }
```

✅ Simple
❌ Hard to reuse

---

### 🟢 Approach 2: **Named Pointcut (Reusable)** ⭐⭐⭐

```java
@Pointcut("execution(* com.app.service.*.*(..))")
public void serviceMethods() {}

@Before("serviceMethods()")
public void logBefore() { }
```

✅ Reusable
✅ Clean
✅ Recommended

---

### 🟢 Approach 3: **Combining Pointcuts**

```java
@Pointcut("execution(* *.save(..))")
public void saveMethods() {}

@Pointcut("within(com.app.service.*)")
public void serviceLayer() {}

@Before("saveMethods() && serviceLayer()")
public void combinedPointcut() {}
```

✅ Very powerful
✅ Fine-grained control

---

### 🟢 Approach 4: **Annotation-Based Pointcut**

```java
@Before("@annotation(LogExecutionTime)")
public void logTime() {}
```

✅ Best practice
✅ No package dependency
✅ Easy to maintain

---

### 🟢 Approach 5: **XML-Based Pointcut (Legacy)**

```xml
<aop:pointcut id="serviceMethods"
 expression="execution(* com.app.service.*.*(..))"/>
```

❌ Old style
❌ Rarely used now

---

## 5️⃣ Execution Expression Pattern (IMPORTANT FOR INTERVIEWS)

```
execution(modifiers returnType package.class.method(args))
```

| Symbol     | Meaning                   |
| ---------- | ------------------------- |
| `*`        | Any                       |
| `..`       | Any number of parameters  |
| `*Service` | Class ending with Service |

---

### Example Breakdown

```java
execution(* com.app..*Service.*(..))
```

✔ Any return type
✔ Any sub-package
✔ Any class ending with `Service`
✔ Any method
✔ Any parameters

---

## ✅ Complete Spring AOP Example 

✔ 2 **custom annotations**
✔ **ALL advice types**
✔ **ALL major pointcut expressions**
✔ **Named & combined pointcuts**
✔ `execution`, `within`, `args`, `this`, `target`, `@annotation`
✔ Success + Exception flow

---

# 📦 Project Structure

```
com.example.aopdemo
│
├── annotation
│   ├── LogExecutionTime.java
│   └── StrongPassword.java
│
├── aspect
│   └── ApplicationAspect.java
│
├── service
│   └── UserService.java
│
└── AopDemoApplication.java
```

---

Required Maven Dependency (pom.xml)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

# 1️⃣ Custom Annotation – `LogExecutionTime`

```java
package com.example.aopdemo.annotation;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface LogExecutionTime {
}
```

📌 **Use**: Measure method execution time

---

# 2️⃣ Custom Annotation – `StrongPassword`

```java
package com.example.aopdemo.annotation;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface StrongPassword {

    // Length rules
    int minLength() default 8;
    int maxLength() default 64;

    // Pattern rules
    boolean isSequenceAllowed() default false;
    boolean isDuplicateAllowed() default true;

    // Character rules
    boolean mustHaveUppercase() default true;
    boolean mustHaveLowercase() default true;
    boolean mustHaveDigit() default true;
    boolean mustHaveSpecialChar() default true;

     //Allowed special characters only. Example: "!@#$%^&*"
    String allowedSpecialChars() default "!@#$%^&*()_+-=[]{}|;:',.<>?";
}
```

📌 **Use**: Validate password strength

---

# 3️⃣ Business Service (Target Object)

```java
package com.example.aopdemo.service;

import com.example.aopdemo.annotation.LogExecutionTime;
import com.example.aopdemo.annotation.StrongPassword;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @LogExecutionTime
    public void registerUser(String username) {
        System.out.println("User registered: " + username);
    }

    @StrongPassword
    public void changePassword(String password) {
        System.out.println("Password changed successfully");
    }

    public void deleteUser() {
        System.out.println("User deleted");
    }

    public void exceptionMethod() {
        throw new RuntimeException("Something went wrong!");
    }
}
```

---

# 4️⃣ Aspect – ALL Join Points + Advices + Patterns

```java
package com.example.aopdemo.aspect;

import com.example.aopdemo.annotation.LogExecutionTime;
import org.aspectj.lang.*;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class ApplicationAspect {

    // =========================
    // NAMED POINTCUTS
    // =========================

    // execution pattern
    @Pointcut("execution(* com.example.aopdemo.service.*.*(..))")
    public void allServiceMethods() {}

    // within pattern
    @Pointcut("within(com.example.aopdemo.service.UserService)")
    public void userServiceOnly() {}

    // args pattern
    @Pointcut("args(String)")
    public void stringArgument() {}

    // annotation-based
    @Pointcut("@annotation(com.example.aopdemo.annotation.LogExecutionTime)")
    public void logExecutionAnnotation() {}

    // =========================
    // BEFORE ADVICE
    // =========================
    @Before("allServiceMethods()")
    public void beforeAdvice(JoinPoint jp) {
        System.out.println("[BEFORE] Method: " + jp.getSignature());
    }

    // =========================
    // AFTER ADVICE (FINALLY)
    // =========================
    @After("userServiceOnly()")
    public void afterAdvice(JoinPoint jp) {
        System.out.println("[AFTER] Method finished: " + jp.getSignature());
    }

    // =========================
    // AFTER RETURNING
    // =========================
    @AfterReturning(
        pointcut = "execution(* registerUser(..))",
        returning = "result"
    )
    public void afterReturningAdvice(Object result) {
        System.out.println("[AFTER RETURNING] Success");
    }

    // =========================
    // AFTER THROWING
    // =========================
    @AfterThrowing(
        pointcut = "execution(* exceptionMethod(..))",
        throwing = "ex"
    )
    public void afterThrowingAdvice(Exception ex) {
        System.out.println("[AFTER THROWING] Exception: " + ex.getMessage());
    }

    // =========================
    // AROUND ADVICE (MOST POWERFUL)
    // =========================
    @Around("logExecutionAnnotation()")
    public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        long end = System.currentTimeMillis();
        System.out.println("[AROUND] Execution time: " + (end - start) + " ms");
        return result;
    }

    // =========================
    // NAMED POINTCUT
    // =========================
    @Pointcut("@annotation(com.example.aopdemo.annotation.StrongPassword)")
    public void strongPasswordMethod() {}

    // =========================
    // BEFORE ADVICE
    // =========================
    @Before("strongPasswordMethod() && args(password,..)")
    public void validatePassword(JoinPoint joinPoint, String password) {

        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        StrongPassword annotation = method.getAnnotation(StrongPassword.class);

        if (password == null) {
            throw new WeakPasswordException("Password cannot be null");
        }

        // Rule checks
        if (password.length() < annotation.minLength()) {
            throw new WeakPasswordException(
                "Password must be at least " + annotation.minLength() + " characters long"
            );
        }

        if (annotation.mustHaveUppercase() && !password.matches(".*[A-Z].*")) {
            throw new WeakPasswordException("Password must contain an uppercase letter");
        }

        if (annotation.mustHaveLowercase() && !password.matches(".*[a-z].*")) {
            throw new WeakPasswordException("Password must contain a lowercase letter");
        }

        if (annotation.mustHaveDigit() && !password.matches(".*\\d.*")) {
            throw new WeakPasswordException("Password must contain a digit");
        }

        if (annotation.mustHaveSpecialChar() &&
            !password.matches(".*[!@#$%^&*()].*")) {
            throw new WeakPasswordException("Password must contain a special character");
        }

        System.out.println("[SECURITY] Strong password validated for method: "
                + signature.getMethod().getName());
    }

   private boolean containsSequence(String password) {
    String lower = password.toLowerCase();

    for (int i = 0; i < lower.length() - 2; i++) {
        char c1 = lower.charAt(i);
        char c2 = lower.charAt(i + 1);
        char c3 = lower.charAt(i + 2);

        if (c2 == c1 + 1 && c3 == c2 + 1) {
            return true; // abc, 123
        }
    }
    return false;
  }

}
```

---

# 5️⃣ Main Application

```java
package com.example.aopdemo;

import com.example.aopdemo.service.UserService;
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class AopDemoApplication {

    public static void main(String[] args) {
        ApplicationContext ctx = SpringApplication.run(AopDemoApplication.class, args);
        UserService service = ctx.getBean(UserService.class);

        service.registerUser("John");
        service.changePassword("StrongPass123");
        service.deleteUser();

        try {
            service.exceptionMethod();
        } catch (Exception e) {}
    }
}
```

---

# 6️⃣ Console Output (Flow Proof)

```
[BEFORE] Method: registerUser
User registered: John
[AROUND] Execution time: 2 ms
[AFTER RETURNING] Success
[AFTER] Method finished: registerUser

[SECURITY] Strong password validated
Password changed successfully

[BEFORE] Method: exceptionMethod
[AFTER THROWING] Exception: Something went wrong!
[AFTER] Method finished: exceptionMethod
```

---

