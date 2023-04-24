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
***
## 🔶 강의 Background
- [수업메모](https://gist.github.com/carami/d3bb434cacf371d44e71538b93e6c212)
- [수업시간 GITHUB - todo project](https://github.com/carami/koscom_2023)
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
***
[맨 위로 가기](#top)
