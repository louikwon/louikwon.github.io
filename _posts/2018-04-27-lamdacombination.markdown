---
layout: post
title:  "람다식을 조합하여 복합한 표현식 만들기"
date:   2018-04-27 12:11:59
author: louikwon
categories: java8
tags: java8
---
 오늘은 람다 표현식을 조합해서 유용하게 활용 할 수 있는 방법을 알아보도록 하겠습니다.

***
### Comparator

```java

 public Optional<List<Car>> comparatorChain(List<Car> carList) {
    //비교에 사용할 키를 추출하는 Funcation 기반의 Comparator
    Comparator<Car> c = comparing(Car::getPrice);

    //가격을 내림차순으로 정렬 후 가격이 같으면 년도를 오름차순으로 정렬
    carList.sort(comparing(Car::getPrice)
            .reversed()
            .thenComparing(Car::getYear));

    return Optional.ofNullable(carList);
}

@Test
public void comparing_조합예제() {
    List<Car> carList = new ArrayList();
    carList.add(
            Car.builder().name("BMWX6").price(1000).year(2015).build()
    );
    carList.add(
            Car.builder().name("K7").price(600).year(2017).build()
    );
    carList.add(
            Car.builder().name("MINI").price(600).year(2014).build()
    );
    carList.add(
            Car.builder().name("CRUZE").price(300).year(2013).build()
    );


    new LamdaChain().comparatorChain(carList)
        .orElseGet(Collections::emptyList)
        .forEach(car -> {
            log.info(">> name {} / year {} / price {}", car.getName(), car.getYear(), car.getPrice());
        });
}

```
***


***
### Predicate
predicate 에서는 or , and 등을 조합하여 여러 조건을 조합 할 수 있습니다. 

```java
public boolean predicateChain (Car car) {
    Predicate<Car> blueCar = (c -> "blue".equals(c.getColor()));

    //a.or(b).and(c) == (a || b) && c
    Predicate<Car> notBlueCarAndNameIsK7 = blueCar
            .or(c -> "K7".equals(c.getName()))
            .and(c -> c.getPrice() > 3000);

    return notBlueCarAndNameIsK7.test(car);
}

@Test
public void predicate_조합예제() {
    Car car1 = Car.builder()
            .name("K7")
            .color("red")
            .price(5000)
            .build();

    Car car2 = Car.builder()
            .name("K5")
            .color("red")
            .price(4000)
            .build();

    LamdaChain lamdaChain = new LamdaChain();

    Assert.assertEquals(true, lamdaChain.predicateChain(car1));

    Assert.assertEquals(false, lamdaChain.predicateChain(car2));

}   
```
***

***
###Function
계산식 들을 조합헤서 결과를 추출하는 등의 작업을 할 수 있습니다.

```java
public int functionChain(Integer input) {
    Function<Integer, Integer> f = x -> x + 10;
    Function<Integer, Integer> g = x -> x * 20;

    //f를 실행한 반환값으로 g를 실행.
    Function<Integer, Integer> h = f.andThen(g);

    return h.apply(input);
}

@Test
public void Function_조합예제() {
    LamdaChain lamdaChain = new LamdaChain();

    Assert.assertEquals(400, lamdaChain.functionChain(10));
}   

```

***