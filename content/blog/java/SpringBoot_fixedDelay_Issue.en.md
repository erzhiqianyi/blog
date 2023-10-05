---
title: "Head first fixedDelay in Spring Boot"
description: .  
draft: false
language: en
date: 2023-10-05
updated: 2023-10-05
taxonomies:
  categories:
    - Java
  tags:
    - Java
    - SpringBoot
---

In Spring Boot, the fixedDelay is a configuration parameter used with the @Scheduled annotation to specify when a method should be executed periodically. It controls the interval between the completion of one execution of the method and the start of the next execution. Here's an easy-to-understand explanation of fixedDelay:

<!-- more -->
## exmaple
Imagine you have a method in your Spring Boot application that you want to run at regular intervals, say every 5 seconds. You can use the @Scheduled annotation along with the fixedDelay parameter to achieve this.

Here's how you would use fixedDelay:

- Import the necessary classes:
```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
```

- Create a Spring bean (a class managed by the Spring framework) and annotate it with @Component so that Spring knows to manage it.

```java
@Component
public class MyScheduledTask {
}

```

- Define a method that you want to run periodically and annotate it with @Scheduled. 
 
  Set the fixedDelay parameter to specify the delay between method executions. The value of fixedDelay is in milliseconds, so for 5 seconds, you would use 5000 milliseconds.

```java
    @Scheduled(fixedDelay = 5000)
    public void runTask() {
        // Code to be executed periodically
        System.out.println("Task executed at regular intervals");
    }
```

In this example, the runTask() method will be executed every 5 seconds (5000 milliseconds). The fixedDelay ensures that there will be a delay of 5 seconds after the completion of one execution before starting the next execution.

So, in summary, fixedDelay in Spring Boot is a simple way to schedule and execute a method at regular intervals, allowing you to control the time gap between the completion of one execution and the start of the next one.