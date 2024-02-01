---
title: "[Java] 쿼츠(Quartz)"
date: 2024-02-01 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java, quartz]
render_with_liquid: false
---

# **Java에서 스케줄링을 높여주는 Quartz**

현재 RPA 개발자로 일하면서 로봇들에게 업무 스케쥴링하는데 필수적인 라이브러리입니다. 직접적인 스케쥴링 커스텀은 깊게 들어가지 않아서 그동안 rpa솔루션의 풀스택 개발만 했는데 이제 배운지도 좀 됐고 딥다이브 해보고 싶어서 quartz공부를 하려고 합니다.

## ✅**Quartz란?**

---

Quartz는 오픈 소스의 자바 기반 스케줄링 프레임워크로, 반복적이고 예약된 작업을 효과적으로 관리할 수 있게 해줍니다. Quartz는 강력한 기능과 다양한 설정 옵션을 제공하여 다양한 스케줄링 요구 사항을 해결할 수 있습니다.

## ✅**기본 개념**

---

### **1. Job(작업)**

Quartz에서 스케줄링되는 단위를 나타냅니다. **`Job`**은 실행되어야 하는 실제 작업을 구현한 클래스입니다.

### **2. Trigger(트리거)**

**`Trigger`**는 **`Job`**이 언제 실행되어야 하는지를 정의합니다. 특정 시간, 주기적으로, 혹은 특정 이벤트가 발생했을 때 **`Job`**을 실행할 수 있도록 설정합니다.

### **3. Scheduler(스케줄러)**

**`Scheduler`**는 **`Job`**과 **`Trigger`**를 관리하고 실행 시점을 제어하는 핵심 역할을 합니다. 여러 개의 **`Job`**을 스케줄링하고 동시에 실행할 수 있도록 지원합니다.

## ✅**Quartz 사용법**

---

### **1. 의존성 추가**

Quartz를 사용하기 위해서는 먼저 Maven 또는 Gradle과 같은 빌드 도구를 통해 의존성을 추가해야 합니다.

```xml
<!-- Maven -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.2</version>
</dependency>

```

### **2. Job 클래스 작성**

실행할 작업을 구현한 **`Job`** 클래스를 작성합니다.

```java
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class MyJob implements Job {
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        // 실제로 수행할 작업 내용
        System.out.println("Quartz Job 실행: " + System.currentTimeMillis());
    }
}

```

### **3. Trigger 및 Scheduler 설정**

**`Trigger`**와 **`Scheduler`**를 설정하여 **`Job`**을 실행할 조건을 정의하고 스케줄러를 시작합니다.

```java
import org.quartz.*;
import org.quartz.impl.StdSchedulerFactory;

public class QuartzExample {
    public static void main(String[] args) throws SchedulerException {
        // JobDetail 생성
        JobDetail jobDetail = JobBuilder.newJob(MyJob.class)
                .withIdentity("myJob", "group1")
                .build();

        // Trigger 생성 - 5초마다 실행
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("myTrigger", "group1")
                .startNow()
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInSeconds(5)
                        .repeatForever())
                .build();

        // Scheduler 생성 및 시작
        Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
        scheduler.start();
        scheduler.scheduleJob(jobDetail, trigger);
    }
}

```

위 예시에서는 **`MyJob`** 클래스를 5초마다 실행하도록 설정했습니다.

## 📌**주의사항 및 팁**

---

- Quartz의 설정은 매우 다양하므로 필요에 따라 세밀하게 조정할 수 있습니다. 공식 문서를 참고하면 다양한 설정 옵션을 확인할 수 있습니다.
- **`Job`** 클래스에서 예외가 발생하면 해당 **`Job`**은 기본적으로 중단됩니다. 따라서 예외 처리에 주의해야 합니다
- Quartz는 클러스터 모드로 동작할 수 있어 여러 대의 서버에서 작업을 분산시킬 수 있습니다.