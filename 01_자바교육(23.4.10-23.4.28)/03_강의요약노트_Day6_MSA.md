# 💎 MSA (교육 6회차_230424) {#top}
> created : 2023-04-24
> 
> updated : 2023-04-24
> 
> author : 백민주
##
***
## 🔶 목차
1. [강의 Background](#-강의-background)
2. [MSA 이벤트 프로그래밍](#-msa-이벤트-프로그래밍)
    - [Step 1. Event 객체 만들기](#-step1-event-객체-만들기-data-layer)
    - [Step 2. Event Handler 만들기](#-step2-event-handler-만들기-business-layer)
    - [Step 3. Service 만들기](#-step3-service-만들기-business-layer)
    - [Step 4. Controller 만들기](#-step4-controller-만들기-presentaion-layer)
3. [MSA Thread](#-msa-thread)
    - [thread class 구현 및 실행](#-thread-class-구현-및-실행)
4. [Async 프로그래밍](#-async-프로그래밍)
    - [Async 프로그래밍 설정](#-async-프로그래밍-설정)
    - [Async Or Sync Service 구현](#-async-or-sync-service-구현)
    - [RestController로 실행](#-restcontroller로-실행)
***
## 🔶 강의 Background
- [수업메모](https://gist.github.com/carami/94ec6cbe0517cc799185a71a4b6d1269)
- [강의자료](https://drive.google.com/drive/folders/1W50_qdHFIklLIPQ94GdDvxtvaFKLIDyb)
- [수업시간 GITHUB](https://github.com/carami/koscom_2023)
  - 소스 작업내역 초기화 : 1차 `git reset --hard` 2차 `git clean -fd`
  - git url로 갖고오기 : `Ctrl + Shift + P` `Clone from Github`
  - git 업데이트 구문(terminal에서) : `git pull origin main`
***

- 트랜잭션 : 나눌수 없는 업무 단위이기 때문에 비즈니스 Layer에서 하는게 나음
- Monolithic Architecture : 전통적인 애플리케이션 아키텍처로, 하나의 큰 애플리케이션으로 구성되는 구조
- MSA : 여러 개의 작은 서비스 각각 독립적으로

***
## 🔶 MSA 이벤트 프로그래밍
#### 🔹 Step1. Event 객체 만들기 (Data Layer)
```java
package com.example.eventexam.event;

public class RegisteredEvent {
    public String name;

    public RegisteredEvent(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
#### 🔹 Step2. Event Handler 만들기 (Business Layer)
```java
package com.example.eventexam.event;

import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class SmsEventHandler {
    @EventListener //이벤트 객체 등록
    public void sendSms(RegisteredEvent event) {
        System.out.println(event.getName() + "님에게 가입 축하 메시지 전송");
    }

    @EventListener
    public void makeCoupon(RegisteredEvent event) {
        System.out.println(event.getName() + "님에게 쿠폰을 전송");
    }
}
```
#### 🔹 Step3. Service 만들기 (Business Layer)
- **하나의 ApplicationEvent에 여러개의 EventListner가 붙어있을 수 있음**
```java
package com.example.eventexam.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

import com.example.eventexam.event.RegisteredEvent;

@Service
public class RegisterEventService {
    // 이벤트를 처리하려면 메시지 브로커가 필요(Application Event = 메시지 브로커)
    @Autowired
    ApplicationEventPublisher applicationEventPublisher;

    public void register(String name) {
        // 회원가입 로직 실행!
        System.out.println("회원 추가 완료!");
        applicationEventPublisher.publishEvent(new RegisteredEvent(name)); //서비스 객체 호출
    }
}
```
#### 🔹 Step4. Controller 만들기 (Presentaion Layer)
```java
package com.example.eventexam.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import com.example.eventexam.service.RegisterEventService;

import lombok.RequiredArgsConstructor;

@Controller
@RequiredArgsConstructor // 생성자 자동 생성
public class TestController {
    // 파이널하게 선언했으면 Lombok이 알아서 생성자를 만들어줌
    // private final RegisterService registerService; // 1) 기존의 서비스 이용 방식
    private final RegisterEventService registerService; // 2) MSA의 이벤트 이용 방식

    @GetMapping("/register/{name}")
    public void register(@PathVariable String name) { // String으로 return해야 화면에 String이 나옴
        registerService.register(name);
        System.out.println("회원가입을 완료했습니다.");
    }
}
```
***
## 🔶 MSA Thread
- 쓰레드 : 하나의 싱글쓰레드가 아니라 여러 수행기능을 함께 하는 것(서버스레드 기본 갯수 : 200개)
  - 스레드가 너무 많을 때에는? 1) 쓰레드수를 늘리던지(=**스케일업**) 2) 서버수를 늘리던지(=**스케일아웃**)
#### 🔹 thread class 구현 및 실행
##### 구현하기 ver 1 : extends Thread
```java
package sample.thread;

public class MyThread extends Thread { // Thread만 extend하면 Thread로 구현 가능함

    @Override // Thread에서 Override 시킨 기능
    public void run() {
        //쓰레드 구현 부분
        try {
            sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
##### 구현하기 ver 2 : implements Runnable
```java
package sample.thread;

public class MyThread2 implements Runnable {
    @Override
    public void run() {
        //쓰레드 구현 부분
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
##### 구현하기 ver 3 : synchronzied (thread 순차대로 실행)
```java
package sample.thread;

public class Teacher {
    public synchronized void answer1() {
    }
    public synchronized void answer2() {
    }
}
```

##### 실행하기 ver 1 : extends Thread
```java
public class ThreadTest1 {
    public static void main(String[] args) {
        MyThread my1 = new MyThread();
        my1.start(); // thread 시작 함수
        // run이라는 method는 내가 수행하면 안됨
   }
}
```
##### 실행하기 ver 2 : implements Runnable
```java
public class ThreadTest2 {
    public static void main(String[] args) {
        Thread t1 = new Thread(new MyThread2("kang"));
        t1.start();
    }
}
```
##### 실행하기 ver 3 : DaemonThread
```java
public class DaemonThreadTest {
    public static void main(String[] args) {
        DaemonThread daemonThread = new DaemonThread();
        daemonThread.setDaemon(true); // 데몬으로 셋팅함(데몬이면 main 끝나면 같이 끝남)
        daemonThread.start();
    }
}
```
***
## 🔶 Async 프로그래밍
#### 🔹 Async 프로그래밍 설정
- Configuration을 만들어줘야 Async 프로그래밍을 할 수 있음(강의자료의 35pg 참고)
```java
@Configuration
@EnableAsync // Async를 Enable 할거에요
public class AsyncConfig implements AsyncConfigurer {
    // AsyncConfigurer : Spring Framework에서 제공하는 interface

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() // Executor는 java.util.concurrent에서 제공
    {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(50);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("AsyncThread-");
        executor.initialize();
        return executor;
    }

    @Override
    @Nullable
    public Executor getAsyncExecutor() {
        return taskExecutor();
    }

    @Override
    @Nullable
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new SimpleAsyncUncaughtExceptionHandler();
    }

}
```
#### 🔹 Async or Sync Service 구현
##### Step1. 세부 객체 작성
```java
@Service
public class GetOneService {
    public String getOne() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println("getOne : " + Thread.currentThread().getName());
        return "one";
    }

    @Async
    public Future<String> getAsyncOne() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("getOne : " + Thread.currentThread().getName());
        return new AsyncResult<>("one");
    }
}
```
##### Step2. 구현 객체 작성
```java
@Service
@RequiredArgsConstructor
public class AsyncTestService {
    private final GetOneService getOneService; // 이 서비스에서 다른 서비스를 injection하는 방법
    private final GetTwoService getTwoService;

    @Autowired
    @Qualifier("taskExecutor")
    private Executor taskExecutor;

    public String executeWithAsync() {
        long start = System.nanoTime();

        // 자체가 Async한 애는 Future객체를 얻어올 수 있음
        Future<String> value1Future = getOneService.getAsyncOne();
        Future<String> value2Future = getTwoService.getAsyncTwo();

        while (!value1Future.isDone() || !value2Future.isDone()) {
            try {
                Thread.sleep(100);
                System.out.print(".");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        try {
            String value1 = value1Future.get();
            String value2 = value2Future.get();

            System.out.println("value1 : " + value1);
            System.out.println("value2 : " + value2);
        } catch (Exception e) {
            e.printStackTrace();
        }

        long end = System.nanoTime();
        String str = "<p><b>executeWithAsync 걸린 시간 : </b>" + formatNanoseconds(end - start) + "</p>";
        System.out.println(str);

        return str;
    }

    public String executeWithCompletableFuture() {
        long start = System.nanoTime();
        CompletableFuture<String> value1Future = CompletableFuture.supplyAsync(() -> getOneService.getOne());
        CompletableFuture<String> value2Future = CompletableFuture.supplyAsync(() -> getTwoService.getTwo());

        CompletableFuture.allOf(value1Future, value2Future).join();

        String value1 = value1Future.join();
        String value2 = value2Future.join();

        System.out.println("value1 : " + value1);
        System.out.println("value2 : " + value2);

        long end = System.nanoTime();
        String str = "<p><b>executeWithCompletableFuture 걸린 시간 : </b>" + formatNanoseconds(end - start) + "</p>";
        System.out.println(str);

        return str;
    }

    public String executeWithoutAsync() {
        long start = System.nanoTime();
        String value1 = getOneService.getOne();
        String value2 = getTwoService.getTwo();
        long end = System.nanoTime();

        System.out.println("value1 " + value1);
        System.out.println("value2 " + value2);

        String str = "<p><b>executeWithoutAsync 걸린 시간 : </b>" + formatNanoseconds(end - start) + "</p>";
        System.out.println(str);

        return str;
    }

    private String formatNanoseconds(long nanoSeconds) {
        long hours = TimeUnit.NANOSECONDS.toHours(nanoSeconds);
        long minutes = TimeUnit.NANOSECONDS.toMinutes(nanoSeconds) % 60;
        long seconds = TimeUnit.NANOSECONDS.toSeconds(nanoSeconds) % 60;
        long millis = TimeUnit.NANOSECONDS.toMillis(nanoSeconds) % 1000;
        long micros = TimeUnit.NANOSECONDS.toMicros(nanoSeconds) % 1000;

        return String.format("%02d:%02d:%02d.%03d,%03d", hours, minutes, seconds, millis, micros);
    }
}
```
#### 🔹 RestController로 실행
```java
@RestController
@RequiredArgsConstructor
public class TestController {

    private final AsyncTestService asyncTestService;

    @GetMapping("/test1")
    public String test1() {
        String str = "<h1># test1</h1>";

        return str + asyncTestService.executeWithoutAsync();
    }

    @GetMapping("/test2")
    public String test2() {
        String str = "<h1># test2</h1>";
        return str + asyncTestService.executeWithoutAsync();
    }

    @GetMapping("/test3")
    public String test3() {
        String str = "<h1># test3</h1>";
        str = str + asyncTestService.executeWithAsync();
        System.out.println("##########test3 끝");
        return str;
    }
}
```
***
[맨 위로 가기](#top)
