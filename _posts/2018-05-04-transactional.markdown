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

- JDK 동적 프록시
    
    - java의 리플렉션을 이용해서 객체를 만듬.
    - 프록시 대상 객체가 최소 하나의 인터페이스를 구현해야함. (그래서 예전에는 AOP 가 동작하게 하기 위해서 불필요하지만 인터페이스를 구현했었던 기억이 있네요. )



- Cglib 프록시 

    - 바이트코드를 조작해 프록시 객체를 만듬.
    - 프록시 대상 객체가 인터페이스를 구현하지 않아도   AOP 가 동작됨. 
    - Spring 3.2 부터 Spring Core 에 포함됨. 
    - Spring boot 는 기본적으로 transaction 대상의 aop를 동작시킬 프록시를 cglib 로 사용. 
    - JDK 프록시 보다 성능이 우수함. 

현재는 특별한 경우가 아니면 Spring AOP Proxy는 Cglib 프록시 방식으로 동작 됩니다. 

그렇다고 해도 Spring 은 인터페이스를 통한 DI , 다양한 DI 활용방법을 사용하도록 권장합니다. 다만 Cglib 프록시를 사용하게 됨으로서 불필요한 인터페이스 구현은 줄일수 있게 되었습니다. 

***
이러한 AOP Proxy 방식으로 동작되면서, @Transaction 을 사용할 때 주의해야할 점을 몇가지 살펴 보겠습니다. 