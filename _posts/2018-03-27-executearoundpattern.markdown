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

### 실행어라운드 패턴이란?
java에서 파일을 읽어오거나, jdbc 를 활용하여 db에 crud 작업을 할때 자바 코드는 보통 아래와 같은 형태로 구성되게 됩니다.

*[초기화/준비 코드]*

파일을 열거나, db connection 등의 작업을 진행하는 부분

*[작업 영역]*

파일을 읽어오거나, db select / insert 등의 작업을 하는 부분.


*[정리/마무리 코드]*

파일 Stream 을 닫거나, db connection close 등 실제 작업에 대한 마무리 하는 부분.
