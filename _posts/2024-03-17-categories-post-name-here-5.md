---
title: "[JAVA] Primitive Type VS Wrapper Class "
excerpt: "Primitive Type, Wrapper Class의 차이점에 대해서"

categories:
  - JAVA
tags:
  - [Java, primitive, wrapper]

permalink: /java/post1/

toc: true
toc_sticky: true

date: 2024-03-17
last_modified_at: 2024-03-17
---
## 궁금증 발생 point  
스프링 공부 중 test 코드를 작성하다가 long이 아니라 Long(Wrapper Class)를 사용하는 것을 보고 구체적으로 어떤 차이가 있는지 궁금증이 발생하였다.
![output](/assets/images/posts_img/java-cate/long.png)
![output](/assets/images/posts_img/java-cate/long_wrapper.png)  
간단히는 null을 담을 수 있는지 없는지가 제일 큰 차이였는데 primitive 타입인 long 같은 경우에는 null을 담을 수가 없고, wrapper 타입인 Long은 null을 담을 수가 있어서 향후 DB를 사용할 때 null 값이 담길 것을 대비하여 Long을 사용해주었다.

## Primitive Type & Wrapper Class 란?
### Primitive Type  
* primitive Type(원시타입)은 자바의 기본 데이터 타입이며 byte, short, int, long, double 등 총 8종류가 있다.  
* 기본형은 사용하기 전에 선언된다.
* 비객체 타입으로 null 값을 갖지 못한다. (test 코드 작성 시 사용하지 않았던 이유)
* 값을 간단하게 저장하고 메모리 공간을 적게 차지한다.
* Stack 영역에 저장되며 값을 가져올 때 속도가 빨라서 좋다.  
### Wrapper Class  
* 인수로 데이터 타입을 전달 받아 해당 값을 객체로 만들어준다. (Java는 객체지향 언어, 모든 것을 객체로 다룬다.)  
* Byte, Boolean, Character, Double, Float, Integer, Long, Short로 사용해준다.
* Wrapper Class는 참조된 주소를 비교하여 같은 객체인지 확인하므로 equals()를 사용한다. (== 연산으로는 확인 불가, 단 String이 리터럴 표현식이면 == 연산 확인 가능)
* Heap 영역에 저장되어서 오버헤드로 인해 접근 속도가 느릴 수 있다. 

## etc.
### 생각 정리
### 참조
https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/lang/Long.html
https://en.wikipedia.org/wiki/Primitive_wrapper_class_in_Java#cite_note-1
https://ontheway.tistory.com/73  
+ChatGPT
