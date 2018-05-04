---
layout: post
title:  "람다를 활용한 실행 어라운드 패턴 실행"
date:   2018-03-27 12:11:59
author: louikwon
categories: java8
tags: java8
---
java8 in action 책을 보면서 쉽게 지나칠 수도 있지만, 알고 있으면 활용도가 높을 만한 주제로 하나씩 글을 작성하고자 합니다.
오늘은 람다를 활용한 실행 어라운드 패턴 실행에 대해서 알아보겠습니다.

***
### 실행어라운드 패턴이란?
java에서 파일을 읽어오거나, jdbc 를 활용하여 db에 crud 작업을 할때 자바 코드는 보통 아래와 같은 형태로 구성되게 됩니다.

**[초기화/준비 코드]**

파일을 열거나, db connection 등의 작업을 진행하는 부분

**[작업 영역]**

파일을 읽어오거나, db select / insert 등의 작업을 하는 부분.


**[정리/마무리 코드]**

파일 Stream 을 닫거나, db connection close 등 실제 작업에 대한 마무리 하는 부분.


> **실행어라운드패턴** 은 위와 같은 형식으로 설정과 정리 두 과정이 둘러싸는 형태의 패턴을 말합니다.

***

실제로 코드를 구현해 보면, 초기화/준비 코드와 정리/마무리 코드는 대부분 중복되는 경우가 많고, 우리가 구현이 필요한 부분은 **결국 작업 영역 부분** 입니다.

예를 들면 파일을 한줄만 읽거나, 또는 두줄만 읽거나 , 아니면 db 에서 A라는 테이블에서 데이터를 조회해 오거나, 또는 B라는 테이블에서 조회해 와야 하거나 등등..
이런 경우마다 초기화/준비 코드와 정리/마무리 코드를 모두 구현해서 사용하여 사용하면 중복되는 코드가 많이 발생하겠죠.

***

아래와 같이 함수형 인터페이스를 생성 후 동작을 파라미터화 하여 활용해 보겠습니다.

로컬에서 파일을 읽어 해당 파일의 내용을 라인 단위로 출력하는 예제를 살펴보겠습니다.
일반적으로, 아래와 같은 형태로 구성됩니다.

```java
/**
 * 파일에서 한 행을 읽는 코드.
 * @return 파일 1라인.
 * @throws IOException
 */
public static String printFile() throws IOException {
    //초기화 코드
    try (BufferedReader br = new BufferedReader(
          new FileReader("/Users/loui.kwon/documents/example/louikwon-data.txt"))) {

        //실제 작업을 실행하는 부분
        return br.readLine();

    } //정리 - 마무리 코드
}
```

파일을 하나의 행만 읽어야 하는 경우, 두개의 행을 한꺼번에 읽어야 하는 경우 등 여러가지 케이스가 필요한 경우, 각 케이스 마다 각각 메소드를 구현한다면
위와 같이 초기화 코드와 정리/마무리 코드가 중복될 수 밖에 없는 상황이 발생합니다.

아래와 같이 함수형 인터페이스를 생성한 후 , 해당 동작만 파라미터로 전달하면 중복된 코드 없이 활용할 수 있습니다.

```java
/**
 * 아래 예제들을 통해서 실행어라운드 패턴을 적용해 보자.
 * 동작을 파라미터화 하기 위해 함수형 인터페이스를 생성
*/
@FunctionalInterface
public interface BufferedReaderProcessor {
    String executeProc(BufferedReader b) throws IOException;
}
```

```java
/**
 * 함수형 인터페이스의 추상 메서드 구현을 직접 전달 가능
 * @param p 함수형 인터페이스의 추상 메소드 구현체
 * @return
 * @throws IOException
*/
public static String printFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("/Users/loui.kwon/documents/example/louikwon-data.txt"))) {
       return p.executeProc(br);
    }
}

```

```java
@Test
public void executeAroundPatternTest() throws IOException {
  //한줄만 읽어야 할 때
  String result = ExecuteAroundPattern.printFile((BufferedReader br) -> br.readLine());

  Assert.assertEquals("test1" , result );

  //두줄을 읽어야 할 때
  String resultMultiLine = ExecuteAroundPattern.printFile(
      (BufferedReader br) -> br.readLine() + br.readLine() );

  Assert.assertEquals("test1test2" , resultMultiLine );
}
```
