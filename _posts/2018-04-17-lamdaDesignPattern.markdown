---
layout: post
title:  "람다로 객체지향 디자인 패턴 구현 하기"
date:   2018-04-17 12:11:59
author: louikwon
categories: java8
tags: java8
---
이번에는 디자인패턴 중 많이 사용되는 전략패턴, 템플릿 메소드 패턴, 팩토리 패턴을 람다식을 이용하여 어떻게 구현할 수 있는지 알아보도록 하겠습니다.



람다를 이용하면 이전에 디자인 패턴으로 해결하던 문제를 더 쉽고 간단하게 해결할 수 있고 간결하게 구현할 수 있습니다.

***
### 전략패턴

전략패턴은 자신의 기능 맥락에서, 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴입니다.

##### 전략패턴의 구성
- 알고리즘을 나타내는 인터페이스
- 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현
- 전략 객체를 사용하는 한 개 이상의 클라이언트


##### 기존의 방식으로 전략 패턴 구현
```java

//알고리즘을 나타내는 인터페이스  
public interface InputValidatation {
    boolean execute(String s);
}
//이메일 형식을 체크하는 구현체
public class IsEmail implements InputValidatation {
    public boolean execute(String s) {
        return s.matches("^[_a-z0-9-]+(.[_a-z0-9-]+)*@(?:\\w+\\.)+\\w+$");
    }
}
//핸드폰번호 형식을 체크하는 구현체
public class IsPhoneNumber implements InputValidatation {
    public boolean execute(String s) {
        return s.matches("^(01[016789]{1}|02|0[3-9]{1}[0-9]{1})-?[0-9]{3,4}-?[0-9]{4}$");
    }
}
//전략 객체를 사용하는 한 개 이상의 클라이언트 
public class Validator {
    private final InputValidatation validatation;

    public Validator(InputValidatation validatation) {
        this.validatation = validatation;
    }

    public boolean validate(String s) {
        return validatation.execute(s);
    }
}


@Test
public void 람다사용전_전략패턴_실행예제() {
    //기존 방식
    StrategyPattern.IsEmail emailValid = new StrategyPattern().new IsEmail();
    StrategyPattern.Validator emailValidator = new StrategyPattern().new Validator(emailValid);
    boolean isEmail = emailValidator.validate("test@daum.net");

    Assert.assertTrue(isEmail);

    StrategyPattern.IsPhoneNumber phoneNumberValid = new StrategyPattern().new IsPhoneNumber();
    StrategyPattern.Validator phoneNumberValidator = new StrategyPattern().new Validator(phoneNumberValid);
    boolean isPhone = phoneNumberValidator.validate("02-1111-1111");

    Assert.assertTrue(isPhone);
}

```

람다 표현식을 이용하면..
 - 자잘한 코드를 제거 가능
 - 코드 조각(또는 전략)을 캡슐화 하여 전략 디자인 패턴을 대신할 수 있습니다.


```java
@Test
public void 람다사용한_전략패턴() {
    //람다를 직접 전달하여 해당 구현체를 구현하는 클라이언트를 직접 생성한다.
    StrategyPattern.Validator emailValidator = new StrategyPattern().new Validator(
            (String s) -> s.matches("^[_a-z0-9-]+(.[_a-z0-9-]+)*@(?:\\w+\\.)+\\w+$")
    );
    boolean isEmail = emailValidator.validate("test@daum.net");

    Assert.assertTrue(isEmail);

    StrategyPattern.Validator phoneNumberValidator = new StrategyPattern().new Validator(
            (String s) -> s.matches("^(01[016789]{1}|02|0[3-9]{1}[0-9]{1})-?[0-9]{3,4}-?[0-9]{4}$")
    );
    boolean isPhone = phoneNumberValidator.validate("02-1111-1111");
    boolean isPhone = phoneNumberValidator.validate("02-1111-1111");

    Assert.assertTrue(isPhone);
}

```
***

### 템플릿 메서드 패턴
템플릿 메소드 패턴은 상속을 통해 기능을 확장해서 사용하는 패턴입니다. 
변하지 않는 부분은 슈퍼클래스에 두고 변하는 부분은 추상 메소드로 정의해둬서 서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 합니다.

```java
abstract class Company {
    public void processEmployee(int id) {
        Employee employee = Database.getEmployeeById(id);
        makeEmployeeHappy(employee);
    }

    abstract void makeEmployeeHappy(Employee employee);
} 
```
회사를 정의하고, 직원들을 행복하게 할 수 있게 각각의 회사는 makeEmployeeHappy 메서드가 원하는 동작을 수행하도록 구현할 수 있습니다.

```java
//Consumer<Employee> 형식을 갖는 두번째 인수를 추가.
public void processEmployee(int id, Consumer<Employee> makeEmployeeHappy) {
    Employee employee = Database.getEmployeeById(id);
    makeEmployeeHappy.aceept(employee);
}
```

Company 클래스를 상속 받지 않고, 직접 람다 표현식을 전달해서 동작을 추가할 수 있습니다. 

```java
    new CompanyLambda().processEmployee(2000, (Employee e) -> increseSalary());
```
