---
title: "Spring Retry"
series: "spring-boot"
date: 2024-08-29
tags: ["java", "spring-retry"]
---

# Spring Retry

在日常开发中，我们经常会遇到一些需要重试的场景，比如调用第三方服务时，由于网络等原因导致调用失败，这时我们可以通过重试的方式来提高成功率。Spring Retry 是 Spring 的一个子项目，它提供了一种简单的重试机制，可以帮助我们在调用失败时进行重试。

## 依赖

```xml
    <dependency>
      <groupId>org.springframework.retry</groupId>
      <artifactId>spring-retry</artifactId>
    </dependency>

    <!-- 如果使用注解方式需要 aop 相关依赖-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
```

## 使用

### 1. 注解方式

```java
import org.springframework.retry.annotation.Recover;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;

/**
 * RetryService
 */
@Service
public class RetryService {

    /**
     * 重试方法
     * retryFor: 出现异常时重试
     * maxAttempts: 最大重试次数
     * @param t
     */
    @Retryable(retryFor = RuntimeException.class, maxAttempts = 3)
    public void sayHi(boolean t) {
        System.out.println("say hi");
        if (t) {
            throw new RuntimeException("error");
        }
    }

    /**
     * 重试失败后执行的方法
     * @param e
     */
    @Recover
    public void recover(RuntimeException e) {
        System.out.println("recover");
    }
}
```

### 2. 编程方式

```java
RetryTemplate retryTemplate = RetryTemplate.builder()
        .maxAttempts(3) // 最大重试次数
        .fixedBackoff(100) // 重试间隔
        .retryOn(RuntimeException.class) // 出现异常时重试
        .build();

String result = retryTemplate.execute(context -> {
    System.out.println("retryTemplate");
    if (context.getRetryCount() < 3) {
        throw new RuntimeException("retryTemplate");
    }
    return "ok";
}, context -> {
    System.out.println("recover");
    return "recover";
});
```

## 总结

Spring Retry 提供了一种简单的重试机制，可以帮助我们在调用失败时进行重试。它支持注解方式和编程方式，可以根据实际需求选择合适的方式来使用。


个人推荐使用编程方式，更加灵活，偶合度更低，更容易控制。需要修改调用逻辑时，可以在 lambda 中进行修改。旧项目中调用第三方服务时，可能已经有了一些重试逻辑不够优雅，这时可以使用编程方式来进行替换，不需太多修改。

## 参考
[Spring Retry](https://github.com/spring-projects/spring-retry)
