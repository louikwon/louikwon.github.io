---
layout: post
title:  "@Transactional, 정말 잘 알고 쓰는거니?"
date:   2018-05-04 12:11:59
author: louikwon
categories: spring
tags: spring
---
#### @Transactional ####

스프링을 사용할 때 특정 로직이  트랙잭션 기반으로 동작되어야 할 때, 가장 많이 사용하고 가장 쉽게 적용할 수 있는 어노테이션. 

하지만 나도 모르게 무심코 @Transactional 을 여기저기 덕지덕지 붙이면서 잘 되겠지머 하면서 넘어가는 경우가 빈번히 생기게 되었고, 실제로 내가 의도하지 않는대로 동작하는 경우가 발생하게 되었습니다.  문득, 내가 정말 @Transactional 에 대해서 잘 이해하고 적절하게 잘 쓰고 있나라는 의문이 들기 시작 했었구요. 

이번 기회에 @Transactional 에 대해 한번 정리하고 넘어가려고 합니다. 

***
#### AOP Proxy ?? ####

@Transactional은 Spring AOP Proxy 기반으로 동작됩니다. AOP Proxy ? 물론 많이 들어 봤지만 정확히 딱 와닫지는 않는것 같습니다. 우선 AOP Proxy 에 대해서 살펴 볼게요.

기본적으로 Spring AOP Proxy는 두가지 메커니즘을 이용합니다. 

- **JDK 동적 프록시**
    
    - java의 리플렉션을 이용해서 객체를 만듬.
    - 프록시 대상 객체가 최소 하나의 인터페이스를 구현해야함. (그래서 예전에는 AOP 가 동작하게 하기 위해서 불필요하지만 인터페이스를 구현했었던 기억이 있네요. )



- **Cglib 프록시** 

    - 바이트코드를 조작해 프록시 객체를 만듬.
    - 프록시 대상 객체가 인터페이스를 구현하지 않아도   AOP 가 동작됨. 
    - Spring 3.2 부터 Spring Core 에 포함됨. 
    - Spring boot 는 기본적으로 transaction 대상의 aop를 동작시킬 프록시를 cglib 로 사용. 
    - JDK 프록시 보다 성능이 우수함. 

현재는 특별한 경우가 아니면 Spring AOP Proxy는 Cglib 프록시 방식으로 동작 됩니다. 

그렇다고 해도 Spring 은 인터페이스를 통한 DI , 다양한 DI 활용방법을 사용하도록 권장합니다. 다만 Cglib 프록시를 사용하게 됨으로서 불필요한 인터페이스 구현은 줄일수 있게 되었습니다. 

***
이러한 AOP Proxy 방식으로 동작되면서, @Transactional 을 사용할 때 주의해야할 점을 몇가지 살펴 보겠습니다. 

- 스프링에서 Transactional 은 인스턴스에서 처음으로 부르는 메서드의 속성(클래스의 속성 포함)을 따라갑니다. 즉 **@Transactional 이 적용된 메서드에서 @Transactional 이 적용되지 않은 메서드를 호출할 때는 Transactional 이 적용 됩니다.**
 물론 전파 속성에 따라 진행 중인 트랜잭션이 있는 경우 이미 시작된 트랜잭션에 참여할 수 도 있고(PROPAGATION REQUIRED), 이미 진행된 트랜잭션 존재 여부에 관계 없이 항상 새로운 트랜잭션을 시작(PROPAGATION_REQUIRES_NEW) 할 수도 있습니다.




- **@Transactional(readOnly = true) 가 적용된 메서드에서 @Transactional 혹은 @Transactional(readOnly = false)가 적용된 메서드를 호출 할 경우 무조건 read-only Transaction이 적용됩니다.** (mysql 은 5.6.5 버전부터 readOnly를 지원하므로, 이전 버전에서는 CUD 를 할 경우 에러가 발생하지 않습니다.) 

  만약 @Transactional 혹은 @Transactional(readOnly = false)가 적용된 메서드에서 @Transactional(readOnly = true) 를 호출할 경우에는 처음 시작될 때 속성으로 진행 되므로, readOnly 속성이 적용되지 않습니다.





- @Transactional 의 경우 AOP 프록시를 사용하므로 클라이언트로부터 호출이 일어날 때만 적용됩니다. 여기서 클라이언트라 함은 인터페이스를 통해 타깃 오브젝트를 사용하는 다른 모든 오브젝트를 말합니다. 
 
  여기서 주의할 점은 클라이언트로 부터 호출이 일어날 때만 적용되므로, **타깃 오브젝트가 자기 자신의 메소드를 호출할 때에는 프록시를 통한 부가기능의 적용이 일어나지 않습니다. 즉 같은 타깃 오브젝트안에서 호출할 경우 트랜잭션이 무시됩니다.** 

  단, AspectJ와 같은 타깃의 바이트코드를 직접 조작하는 방식의 AOP 에서는 같은 타깃 오브젝트내에서의 호출시에도 트랜잭션이 적용됩니다.




- **Spring 에서 Transaction 사용시 롤백 여부는 Unchecked Exception (Runtime Exception) 이 발생했는지 여부에 따라 결정됩니다.** 런타임 예외가 아닌 체크 예외를 던지는 경우에는 이것을 예외 상황으로 인식하지 않고 일종의 비즈니스 로직에 따른 의미가 있는 리턴 방식의 한가지로 인식해서 트랜잭션을 커밋해 버립니다. 

  비즈니스적인 의미가 있는 예외 상황만 체크 예외를 사용하고, 그 외의 모든 복구 불가능한 순수한 예외의 경우는 런타임 예외로 포장되어 전달한다는 기본적인 예외 처리 원칙을 스프링은 따른다고 가정합니다.

  만약 런타임 예외가 아는 체크 예외를 던지는 경우에도 롤백이 필요하다고 한다면, 아래와 같이 전파방식을 설정해야 합니다.

```java

@Transactional(rollbackFor = Exception.class)

```
