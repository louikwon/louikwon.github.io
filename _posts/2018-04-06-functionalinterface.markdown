---
layout: post
title:  "Java8에서 제공해주는 함수형 인터페이스의 활용"
date:   2018-04-06 12:11:59
author: louikwon
categories: java8
tags: java8
---
오늘은 함수형 인터페이스 중에서도 Java8에서 기본으로 제공해 주는 함수형 인터페이스를 알아보고 어떻게 할용할 수 있는지 알아보겠습니다.

***
### Predicate
java.util.function.Predicate<T>

test 라는 추상 메서드를 정의하고, test는 제네릭 형식 T의 객체를 인수로 받아 불린을 반환.

활용 : 특정 로직에서 불린 형식으로 체크할 때, 해당 체크 케이스가 다양한 경우.

```java
/**
  * predicate를 이용하여 특정 조건을 만족시키는 항목으로 구성된 list를 반환 하는 예제.
  * @param list 체크할 데이터 셋.
  * @param p  조건.
  * @param <T> 체크할 데이터 형식
  * @return 조건을 만족하는 데이터 셋.
*/
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
   List<T> resultList = new ArrayList<>();
   for (T s: list) {
       if (p.test(s)) {
           resultList.add(s);
       }
   }
   return resultList;
}
```

```java
@Test
public void predicate_실행예제() {
	List<String> originalList = Arrays.asList("liverpool", "manchester united", "manchester city");

	//체크할 조건으로 함수형 인터페이스 인스턴스 생성.
  //Boolean을 반환하는 동작을 선언한다.
	Predicate<String> manContainPredicate = (String s) -> s.contains("man");

	List<String> manContainList = FunctionalInterfaceExample.filter(originalList, manContainPredicate);

	Assert.assertEquals(manContainList.size(), 2);

}

```
***
### Consumer ###
java.util.function.Consumer<T>

제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상메서드를 정의

활용 : void 를 반환하므로, **반환값이 없는 동작을 실행할때 활용** 할 수 있다.

```java
/**
  * Consumer 를 이용하여 특정 동작을 실행. Consumer는 void 를 반환
  * * T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 사용
  * @param list 데이터셋
  * @param c consumer
  * @param <T> list  형색
*/
 public static <T> void forEach(List<T> list, Consumer<T> c) {
     for(T i : list) {
         c.accept(i);
     }
 }
```

```java
@Test
public void consumer_실행예제() {
	FunctionalInterfaceExample.forEach(
			Arrays.asList(1,2,3,4,5,6,7,8,9) ,
			(Integer i) -> {
				Integer sum = i;
				sum = sum + 1;
				System.out.print(" Sum is " + sum);
			}
	);
}
```

***
### Function ###
java.util.function.Function<T, R>

제네릭 형식 T를 받아서 제네릭 형식 R 객체를 반환하는 apply라는 추상 메서드를 정의

활용 : 반환값이 있는 특정 로직을 전달 받아 해당 정보를 추출

```java
/**
   * 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 apply 라는 추상 메서드를 정의.
   * 입력을 출력으로 매핑하는 람다를 정의 할 때 활용할 수 있다.
   * String 리스트를 인수로 받아 각 String의 길이를 포함하는 Integer 리스트로 변환하는 map  이라는 메서드를 정의하는 예제
   * @param list 데이터셋
   * @param f function
   * @param <T> 인수 형식
   * @param <R> 반환형식
   * @return
*/
public static <T,R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> resultList = new ArrayList<>();
    for (T s : list) {
        resultList.add(f.apply(s));
    }
    return resultList;
}
```

```java
@Test
public void function_실행예제() {
	List<Integer> resultList = FunctionalInterfaceExample.map(
		Arrays.asList("everton", "sunderland", "leedsunited"),
			(String s) -> s.length()
	);

	List<Integer> checkList = Arrays.asList(7, 10, 11);

	Assert.assertEquals(checkList, resultList);
}

```

***
