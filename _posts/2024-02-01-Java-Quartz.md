---
title: "[Java] 쿼츠(Quartz)"
date: 2024-02-01 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java, quartz]
render_with_liquid: false
---

## 📌 들어가며

현재 RPA 개발자로 일하며 로봇들에게 업무를 스케줄링할 때 필수적으로 쓰는 라이브러리가 **Quartz**다. 그동안 RPA 솔루션의 풀스택 개발만 하면서 스케줄링 커스텀을 깊게 파지 못했는데, 이제 딥다이브해보고 싶어 정리한다.

> **Quartz란?**
> 오픈 소스의 **자바 기반 스케줄링 프레임워크**. 반복적이고 예약된 작업을 효과적으로 관리하며, 다양한 설정 옵션으로 폭넓은 스케줄링 요구사항을 해결한다.

---

## 1. 기본 개념 — Job · Trigger · Scheduler

Quartz는 세 가지 핵심 요소로 동작한다. **무엇을(Job)**, **언제(Trigger)**, **누가 관리하나(Scheduler)**로 나뉜다.

| 요소 | 역할 |
|------|------|
| **Job** | 실행되어야 하는 **실제 작업**을 구현한 클래스 (무엇을) |
| **Trigger** | Job이 **언제** 실행될지 정의 (특정 시간·주기·이벤트) |
| **Scheduler** | Job과 Trigger를 관리하고 **실행 시점을 제어**하는 핵심 |

```
Scheduler (관리·실행)
   ├─ Job      "무엇을 실행할까?"  (MyJob.execute)
   └─ Trigger  "언제 실행할까?"    (5초마다, 매일 09시 등)
```

---

## 2. 사용법

### ① 의존성 추가

```xml
<!-- Maven -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.2</version>
</dependency>
```

### ② Job 클래스 작성

실행할 작업을 `Job` 인터페이스의 `execute()`에 구현한다.

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

### ③ Trigger·Scheduler 설정

Trigger로 실행 조건을 정하고, Scheduler를 시작해 Job을 등록한다.

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

위 예시는 `MyJob`을 **5초마다** 실행하도록 설정한 것이다.

---

## 📌 주의사항 및 팁

| 항목 | 내용 |
|------|------|
| **세밀한 설정** | 설정 옵션이 매우 다양 → 공식 문서 참고 권장 |
| **예외 처리** | Job에서 예외 발생 시 해당 Job은 기본적으로 **중단**됨 → 예외 처리 주의 |
| **클러스터 모드** | 여러 서버에 작업을 **분산**시킬 수 있음 |

---

## 📝 정리

```
Quartz 스케줄링
├─ Job        무엇을 실행할지 (execute 구현)
├─ Trigger    언제 실행할지 (주기·시간·이벤트)
├─ Scheduler  Job+Trigger 관리 및 실행 시점 제어
└─ 특징       세밀한 설정 · 예외 시 중단 · 클러스터 분산
```

| 개념 | 한 줄 정의 |
|------|------|
| **Quartz** | 자바 기반 스케줄링 프레임워크 |
| **Job / Trigger / Scheduler** | 무엇을 / 언제 / 누가 관리하나 |

Quartz는 "무엇을(Job) 언제(Trigger) 실행할지"를 스케줄러가 관리하는 구조다. 이 삼각 구조만 이해하면, 복잡한 반복 작업도 체계적으로 예약·관리할 수 있다.
