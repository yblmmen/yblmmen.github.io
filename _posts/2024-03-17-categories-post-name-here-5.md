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
![output](/assets/images/posts_img/java-cate/pri_wrap.png)
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
  
## Boxing & UnBoxing
### boxing, unboxing 이란?
먼저 Boxing은 primitive type이나 reference type을 wrapper 타입으로 변환하는 과정을 의미한다.  
자바에서는 AutoBoxing 기능을 제공해준다. UnBoxing은 그 반대의 의미라고 생각하면 된다.  
그런데 이 과정에서 비용이 추가적으로 발생하는데 primitive type을 가져오려고 할 때에도 wrapper class로 인해 메모리를 추가적으로 탐색하게 된다.  
```
primitive type > Stack에서 빠르게 조회  
boxing 후 > Stack + Heap 영역 조회
```  
  
## 결론  
불필요한 객체 생성과 참조 비용이 발생하여 성능 저하를 일으킬 수 있으므로 필요할 때 잘 골라서 사용하는 것이 좋을 것 같다.  
Not null이 보장된 상황, 간단한 값 저장과 연산에서는 primitive을 사용하고 객체지향적인 기능이 필요하거나 null 값이 발생할 것 같을 때에는 Wrapper Class를 쓰자.  


## etc.
### 생각 정리  
JAVA를 배우긴 했으나 실제 응용 과정에서 왜 그렇게 사용해야 하는지 이해가 안될 때가 있는 것 같다.  
스프링 공부를 해가면서 추가적으로 JAVA 라는 언어에 대한 깊은 이해와 공부도 필요하다고 느낀다.  

### 참조
https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/lang/Long.html
https://en.wikipedia.org/wiki/Primitive_wrapper_class_in_Java#cite_note-1
https://ontheway.tistory.com/73  
+ChatGPT
