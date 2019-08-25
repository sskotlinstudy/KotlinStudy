---
title: "Kotlin Study - 02 코틀린 기초 -1 "
date: 2019-07-24 20:22:00 
categories: kotlin study
---

# 코틀린 기초

## Hello World!
~~~
fun main(args: Array<String>) {
  println("Hello World!")
}
~~~
* 함수 선언 키워드는 fun
* 변수/파라미터 선언 뒤에 타입을 작성
* 클래스 안에 함수가 존재할 필요 X
* 배열처리를 위한 문법 X, 다른 클래스와 동일하게 처리
* System.out.println -> println 의 Wrapper
* ; 이 줄 마지막에 없음

## 함수
~~~
fun max(a: Int, b: Int): Int {
  return if (a>b) a else b
}
~~~
* if가 값을 만들어 내지 못하는 문장이 아니라 결과를 만드는 식

### 좀더 간결한 함수(블록 -> 식으로 변경)
~~~
fun max(a: Int, b: Int): Int = if (a>b) a else b
~~~

### 더더욱 간결한 함수(반환타입 생략)
~~~
fun max(a: Int, b: Int) = if (a>b) a else b
~~~
* 반환타입을 따로 명시 하지 않아도, 컴파일러가 식의 본문을 분석해서 식의 결과 타입을 반환타입으로 정해줌
* 단, 블록이 본문인 함수는 반드시 return 과 반환타입을 지정해야함

## 변수
~~~
val question = "질문이 있습니다"
val answer = 44
val otherAnswer: Int = 53
val realNumber = 7.5e6
val initValue: Int
initValue = 42
~~~
* 코틀린에서는 컴파일러가 컴파일시에 타입을 지정 해주므로 변수 타입 생략이 흔함
* 원한다면 타입을 명시 할 수 있음
* 부동소수점(7.5e6) 과 같은 상수를 사용 한다면, 변수 타입은 Double로 됨
* 초기화 식을 사용하지 않을 경우, 반드시 타입을 지정 해줘야함

### 변경 가능한 변수와 변경 불가능한 변수
* var : 변경 가능한 변수
* val : 변경 불가능한 변수 (java의 final)
  - 기본적으로 모든 변수를 val로 선언하고 나중에 꼭 필요할 경우에만 var로 변경 -> 그래야 함수형 코드에 가까워진다
  - val 변수는 반드시한번만 초기화 해야 된다
  - val 참조 자체는 불변일지라도, 그 참조가 가르키는 객체의 내부 값은 변경이 가능하다
  ~~~
  val programmingLanguages = arryListOf("Java")
  programmingLanguages.add("Kotlin")
  ~~~
  - var를 사용하더라도 변수의 값은 변경되지만, 변수의 타입은 변경되지 않는다
  
## 문자열 템플릿
~~~
fun main(args: Array<String>) {
  val name = if (args.size > 0) argl[0] else "kotlin"
  println("Hello $name")
}
~~~
* 다른 스크립트 언어들과 유사하게 문자열 리터럴 내부에서 변수를 사용 하려면 $ 를 사용
* ${args[0]} 와 같이 중괄호로 묶어서 사용 가능
* 중괄호로 묶은식 안에서도 "" 사용 가능

## 클래스와 프로퍼티
~~~
Java
class person {
  private final String name;
  
  public person(String name) {
    this.name = name;
  }
  
  public String getName() {
    return name;
  }
}

Kotlin
class person(val name: String)
~~~
* 코틀린의 기본 가시성은 public
* 코틀린은 기본으로 프로퍼티를 제공
* val로 선언한 프로퍼티는 읽기 전용(getter만 제공)
* var로 선언한 프로퍼티는 변경 가능(getter/setter 모두 제공)
* 하지만 val,var 둘다 필드는 비공개

~~~
class person {
  val name: String,
  var isMarried: Boolean
}

Java
person p = new person("yoo", false);
System.out.println(p.getName());
System.out.println(p.isMarried());

Kotlin
val p = person("yoo,false)
println(p.name)
println(p.isMarried)
~~~
* 둘의 로직은 동일, 둘다 프로퍼티를 사용 하는것
* java가 p.setIsMarried(false) 이렇게 설정한다면, 코틀린은 p.isMarried = flase 로 함

### 커스텀 접근자
~~~
class rectangle(val h: Int, val w: Int) {
  val isSquare: Boolean
    get() {
      return h == w
    }
}
~~~
* isSquare 프로퍼티에는 자체 값을 저장하는 필드가 필요없음. 자체 구현을 제공하는 getter만 존재
  - 클라이언트가 프로퍼티에 접근 할때마다 매번 다시 계산
  - get() = h == w 로 바꿔도 똑같이 사용 가능
  
  ~~~
  val rect = rectangle(33,44)
  println(rect.isSquare) 

  false
  ~~~
  
  - 파라미터가 없는 함수와 커스텀 게터는 비슷 하지만, 일반적으로 클래스의 특성을 정의 할때에는 프로퍼티를 사용 

## 디렉터리와 패키지
* java와 동일하게 package 가 존재
* package를 파일 맨 앞에 선언 할수 있고, 같은 packag라면 다른 파일에서 정의한 선언도 가져다 쓸 수있음
* 다른 package를 사용 할때에는, java와 동일하게 import 사용
* 코틀린에서는 클래스 import와 함수 import가 동일하여, 모든 선언을 import로 가져 올 수있음

~~~
package geometry.shapes
import java.util.Random

class rect(val h: Int, val w: Int) {
  val isSquare: Boolean
    get() {
      return h == w
    }
}

fun createRandomRect() : rect {
  val random = Random()
  return rect(random.nextInt(), random.nextInt())
}
~~~

~~~
package geometry.example
import package geometry.shapes.createRandomRect

fun main(args: Array<String>) {
  println(createRandomRect().isSquare)
}
~~~

* java의 경우 패키지 구조와 일치하는 디렉터리 구조를 만들어서 위치 시키지만, 코틀린은 마음대로 할 수있다.
* 하지만 java와의 마이그레이션을 생각해서 java와 동일하게 만드는것이 좋다
